# Ansible 能否用來抓取（拉取）遠端 Log 檔案到本地？

## 摘要

可以。Ansible 內建的 `ansible.builtin.fetch` 模組正是為此而設計，其行為相當於 `copy` 模組的反向操作：把遠端主機上的檔案「拉」回控制節點（Controller，亦即執行 `ansible-playbook` 的本地機器），並依照主機名稱自動建立目錄樹[^fetchdoc]。對於單一或少數 Log 檔案，`fetch` 已足夠；若需要拉取整個 Log 目錄、大量檔案或體積較大的 Log，則建議改用 `ansible.posix.synchronize`（rsync 的包裝）以 `mode: pull` 方式進行[^syncdoc]。

本報告僅聚焦「從遠端抓取 Log 到本地」這個方向（pull），不涉及反向的推送（push）。

## 一、核心模組：`ansible.builtin.fetch`

`fetch` 官方文件明確描述其用途為：「This module works like ansible.builtin.copy, but in reverse. It is used for fetching files from remote machines and storing them locally in a file tree, organized by hostname.」[^fetchdoc] 也就是說，它會將遠端檔案抓回本地，並以主機名稱作為目錄結構的一層。

它同時也支援 Windows 目標節點[^fetchdoc]。

### 主要參數

| 參數 | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `src` | string（必填） | — | 遠端要抓取的檔案。**必須是檔案，不能是目錄**；不支援遞迴抓取[^fetchdoc]。 |
| `dest` | string（必填） | — | 本地儲存目錄。除非設定 `flat`，否則會存成 `dest/hostname/完整遠端路徑`[^fetchdoc]。 |
| `flat` | boolean | `false` | 覆寫預設的「主機名＋路徑」目錄結構。若 `dest` 以 `/` 結尾，則使用來源檔案的 basename（類似 `copy`）。**警告：多台主機若有相同檔名，會互相覆寫**[^fetchdoc]。 |
| `fail_on_missing` | boolean | `true`（2.5 起） | 設為 `true` 時，遠端檔案無法讀取即任務失敗。若某些主機沒有該 Log，可設為 `false` 避免中斷[^fetchdoc]。 |
| `validate_checksum` | boolean | `true` | 抓取後驗證來源與目的檔案 checksum 是否一致[^fetchdoc]。 |

### 目錄結構示例

以 `src=/etc/profile`、`dest=/backup`、主機 `host.example.com` 為例，檔案會存成：

```
/backup/host.example.com/etc/profile
```

其中使用的主機名是 **inventory 名稱**（`inventory_hostname`），未必等同 FQDN 或 `ansible_host`[^fetchdoc]。

### 基本 Playbook 範例

```yaml
- name: Gather log files from remote hosts
  hosts: all
  tasks:
    - name: Fetch the application log from each host
      ansible.builtin.fetch:
        src: /var/log/myapp/app.log
        dest: /backup/logs/
        fail_on_missing: no
        validate_checksum: yes
```

結果：檔案會落在 `/backup/logs/<hostname>/var/log/myapp/app.log`[^fetchdoc]。

### 使用 `flat` 攤平目錄

```yaml
    - name: Fetch logs, flattening directory structure
      ansible.builtin.fetch:
        src: /var/log/syslog
        dest: /backup/logs/{{ inventory_hostname }}-syslog
        flat: yes
```

在 `dest` 中代入 `{{ inventory_hostname }}` 可避免多主機同名檔案互相覆寫[^fetchdoc]。

## 二、抓取多個 Log 檔案的常見做法

`fetch.src` 一次僅接受**單一檔案**，且**無法遞迴目錄**。而 `fileglob` lookup 是在控制節點端（controller-side）比對本地檔案，並非遠端檔案系統[^fileglob]。因此若要抓取多個遠端 Log，需採用下列模式。

### 模式 A：已知檔案清單，用 `loop` 逐檔抓取

```yaml
- name: Fetch several known logs from each host
  ansible.builtin.fetch:
    src: "{{ item }}"
    dest: "./logs/{{ inventory_hostname }}/"
    flat: true
    fail_on_missing: false
  loop:
    - /var/log/messages
    - /var/log/secure
    - /var/log/nginx/access.log
```

配合 `dest` 含 `{{ inventory_hostname }}` 與尾端斜線＋`flat: true`，可得到依主機分開的檔案：

```
./logs/web01.example.com/messages
./logs/web01.example.com/secure
./logs/web01.example.com/access.log
```

### 模式 B：先在遠端用 `find` 找出檔案，再逐檔 `fetch`

因為 `fileglob` 是本地端比對，要迭代遠端檔案必須使用 `ansible.builtin.find` 模組[^finddoc][^fileglob]：

```yaml
- name: Find all *.log files on the remote host
  ansible.builtin.find:
    paths: /var/log/myapp
    patterns: "*.log,*.log.*"
    recurse: true
    age: 1d
  register: log_files

- name: Fetch each discovered log into a host-specific directory
  ansible.builtin.fetch:
    src: "{{ item.path }}"
    dest: "./logs/{{ inventory_hostname }}/"
    flat: true
    fail_on_missing: false
  loop: "{{ log_files.files }}"
  loop_control:
    label: "{{ item.path }}"
```

`find` 的重點[^finddoc]：
- `patterns` 只比對 **base name**，支援逗號分隔或 YAML 清單；預設為 glob。
- 若設 `use_regex: true`，正則必須匹配 **整個檔名**。
- 可用 `excludes`、`age`、`size`、`file_type`、`recurse`、`hidden`、`limit` 等過濾。
- 回傳 `files`（含 `path`、`checksum`、`mode` 的 dict 清單）、`matched`、`examined`。
- 使用 `loop_control.label` 可讓輸出僅顯示 `item.path`，提升可讀性。

### 模式 C：`synchronize` 以 `mode: pull` 拉取整個目錄（大量／大型 Log 首選）

由於 `fetch` 無法遞迴，若要拉取整個 Log 目錄，應改用 `ansible.posix.synchronize`：

```yaml
- name: Pull the whole log directory from each host
  ansible.posix.synchronize:
    mode: pull
    src: /var/log/myapp/
    dest: "./logs/{{ inventory_hostname }}/myapp/"
    rsync_opts:
      - "--exclude=*.gz"
      - "--no-motd"
```

`src` 結尾加 `/` 代表複製目錄「內容」，不加則複製目錄本身（與 rsync／copy 慣例相同）[^syncdoc]。

## 三、`fetch` 的限制與注意事項

1. **僅限單一檔案、不支援目錄遞迴**：`src` 必須是檔案，是抓取 Log 目錄時最大的限制[^fetchdoc]。
2. **`become` 併用時的記憶體問題**：官方文件警告，`fetch` 搭配 `become` 時，會另外使用 `ansible.builtin.slurp` 取得遠端 checksum，這**使傳輸量加倍**，視檔案大小可能耗盡遠端或本地記憶體而導致 `MemoryError`；因此建議盡可能**不要搭配 `become`** 執行[^fetchdoc][^slurpdoc]。
3. **不適合大型檔案／大量傳輸**：`fetch` 一次傳一個檔案，且走控制節點的傳輸路徑，對大量或大檔效率不佳。`slurp` 更是把整個檔案塞進記憶體，需要「至少兩倍於原檔大小的 RAM」[^slurpdoc]。
4. **預設目錄結構可能不符預期**：預設為 `dest/hostname/完整路徑`，需用 `flat` 覆寫，但要注意跨主機覆寫問題[^fetchdoc]。
5. **`fail_on_missing` 預設值於 2.5 改為 `true`**，可能讓舊 Playbook 行為改變，需要時應顯式設為 `false`[^fetchdoc]。
6. **`async` 不支援**，`safe_file_operations` 與 vault 亦不支援；check mode 與 diff mode 則完整支援[^fetchdoc]。

## 四、模組選擇建議

| 需求情境 | 建議模組 |
|---|---|
| 每台主機抓取**單一／少數**檔案到控制節點 | `ansible.builtin.fetch`[^fetchdoc] |
| 拉取**整個目錄**、多個檔案、遞迴 | `ansible.posix.synchronize`（`mode: pull`）[^syncdoc] |
| 需要檔案**內容作為變數**（非存檔） | `ansible.builtin.slurp`[^slurpdoc] |
| 反向往遠端**推送**檔案 | `ansible.builtin.copy`[^copydoc] |

> 註：`ansible.posix.synchronize` 為 collection 模組，官方文件提醒它「is a wrapper around rsync」，且由本地端發起；需本地與遠端都安裝 `rsync`，且僅支援 ssh、paramiko、local、docker 等連線類型，連線不得需要密碼，權限提升僅支援免密碼 sudo[^syncdoc]。

## 五、最佳實務（Best Practices）

1. **依情境選對工具**：少數個別檔案用 `fetch`；目錄、大量或大型 Log 用 `synchronize`（rsync, `mode: pull`）[^fetchdoc][^syncdoc]。
2. **盡量避免 `fetch` 搭配 `become`**：它會觸發 `slurp` 使傳輸量加倍，大檔有 `MemoryError` 風險；可改讓連線帳號有讀取 Log 權限，或改用 `synchronize`[^fetchdoc][^slurpdoc]。
3. **大量／大型 Log 優先使用 `synchronize` 拉取**，而非對上百個檔案迴圈 `fetch`。
4. **用 `find`（而非 `fileglob`）探索遠端檔案**：`fileglob` 是本地端且非遞迴；`find` 在目標端執行，可依 `patterns`、`age`、`size`、`excludes` 過濾[^finddoc][^fileglob]。
5. **優雅處理不存在檔案**：設 `fail_on_missing: false`（或 `ignore_errors`／`failed_when`），避免缺少 Log 的主機使整個 run 失敗[^fetchdoc]。
6. **目的路徑務必含 `{{ inventory_hostname }}`**（或依賴 `fetch` 預設目錄樹），尤其使用 `flat: true` 時，以防跨主機覆寫[^fetchdoc]。
7. **調控並行度**：Ansible 預設 forks 為 5 並平行執行任務；可用 `ansible.cfg` 的 `forks`、`serial` 或 `throttle` 控制對控制節點與網路的負載[^strategydoc]。
8. **善用 `loop_control.label`**：迴圈 `find` 結果時僅顯示 `item.path`，輸出更精簡[^finddoc]。
9. **完整性驗證**：`fetch` 保留預設 `validate_checksum: true`；`synchronize` 可用 `checksum: true` 以內容而非時間／大小判斷同步[^fetchdoc][^syncdoc]。
10. **排除雜訊檔**：`synchronize` 可用 `rsync_opts`（如 `--exclude=*.gz`、`--exclude=*.old`）或來源目錄的 `.rsync-filter` 檔過濾[^syncdoc]。

## 六、結論

Ansible **可以**用來抓取遠端 Log 檔案到本地。做法是：

- 少量、特定檔案 → 使用 `ansible.builtin.fetch`，它預設依主機名建立目錄樹，也可用 `flat` 攤平。
- 整個 Log 目錄、大量或大型檔案 → 使用 `ansible.posix.synchronize` 搭配 `mode: pull`，因為 `fetch` 無法遞迴且不適合大量傳輸。
- 探索遠端有哪些 Log 檔 → 使用 `ansible.builtin.find`，不要用本地端的 `fileglob`。

實務上最常見的陷阱是 `fetch` 搭配 `become` 造成記憶體爆量，以及多主機同名檔案因 `flat` 而互相覆寫，兩者都應事先留意。

## 參考文獻

[^fetchdoc]: Ansible Community Documentation. (n.d.). ansible.builtin.fetch module – Fetch files from remote nodes. Retrieved 2026-10-11, from https://docs.ansible.com/ansible/latest/collections/ansible/builtin/fetch_module.html

[^syncdoc]: Ansible Community Documentation. (n.d.). ansible.posix.synchronize module – Ansible managed sync (rsync wrapper). Retrieved 2026-10-11, from https://docs.ansible.com/ansible/latest/collections/ansible/posix/synchronize_module.html

[^slurpdoc]: Ansible Community Documentation. (n.d.). ansible.builtin.slurp module – Slurps a file from remote nodes. Retrieved 2026-10-11, from https://docs.ansible.com/ansible/latest/collections/ansible/builtin/slurp_module.html

[^copydoc]: Ansible Community Documentation. (n.d.). ansible.builtin.copy module – Copy files to remote locations. Retrieved 2026-10-11, from https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html

[^finddoc]: Ansible Community Documentation. (n.d.). ansible.builtin.find module – Return a list of files based on specific criteria. Retrieved 2026-10-11, from https://docs.ansible.com/ansible/latest/collections/ansible/builtin/find_module.html

[^fileglob]: Ansible Community Documentation. (n.d.). ansible.builtin.fileglob lookup – list files matching a pattern. Retrieved 2026-10-11, from https://docs.ansible.com/ansible/latest/collections/ansible/builtin/fileglob_lookup.html

[^strategydoc]: Ansible Community Documentation. (n.d.). Controlling playbook execution: strategies and more. Retrieved 2026-10-11, from https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_strategies.html