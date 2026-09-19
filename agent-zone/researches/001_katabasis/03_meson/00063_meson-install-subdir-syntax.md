# Meson `install_subdir()` 語法研究

`install_subdir()` 是 Meson 建置系統中用來安裝整個子目錄樹的函式，定義在官方參考手冊與安裝指南中[^ref-meson][^ref-install]。

## 基本語法

```meson
install_subdir('mydir', install_dir : 'include')
```

- 第一個（且唯一必要）位置引數：來源子目錄名稱，相對於當前 `meson.build` 所在的原始碼目錄。
- `install_dir` 關鍵字引數（必要）：指定安裝目標路徑，可為相對於 `prefix` 的相對路徑，或絕對路徑（如 `/etc`）。

## 安裝行為

執行 `install_subdir('mydir', install_dir : 'include')` 後，原始碼樹中的 `mydir/` 完整內容會被複製到安裝前綴下的 `include/mydir/`[^ref-install]。

```
原始碼樹：
  mydir/
    ├── file1.txt
    └── sub/
        └── file2.txt

安裝後：
  <prefix>/include/mydir/
    ├── file1.txt
    └── sub/
        └── file2.txt
```

## 注意事項

- `install_dir` 若為相對路徑，則相對於安裝前綴（`prefix`）；若為絕對路徑（如 `/etc`），則直接安裝到該路徑[^ref-install]。
- 與 `install_data()` 不同，`install_subdir()` 不接受 `rename` 關鍵字引數——它是整棵子樹複製，無法逐一重新命名檔案。
- 支援 `install_mode` 關鍵字引數（自 Meson 0.47.0 起），可指定安裝後的檔案權限、擁有者與群組[^ref-meson]。
- 支援 `install_tag` 關鍵字引數（自 Meson 0.60.0 起），可搭配 `meson install --tags` 選擇性安裝[^ref-meson]。
- `exclude_directories` 與 `exclude_files` 關鍵字引數（自 Meson 1.1.0 起）可用來排除特定目錄或檔案，不進行安裝。
- `strip_directory` 關鍵字引數（自 Meson 1.3.0 起）：若設為 `true`，則只複製子目錄的**內容**，而不建立子目錄本身。例如 `install_subdir('mydir', install_dir : 'include', strip_directory : true)` 會將 `mydir/` 底下的內容直接安裝到 `include/`，而非 `include/mydir/`。

## 範例

```meson
# 基本用法：安裝 data/ 到 share/myapp/
install_subdir('data', install_dir : get_option('datadir') / 'myapp')

# 安裝到絕對路徑（如 /etc）
install_subdir('config', install_dir : '/etc/myapp')

# 排除特定檔案
install_subdir('web', install_dir : 'share/myapp',
               exclude_files : ['secret.json'])

# 排除特定目錄
install_subdir('web', install_dir : 'share/myapp',
               exclude_directories : ['tests'])

# 剝離目錄層級（只安裝內容）
install_subdir('assets', install_dir : 'share/myapp',
               strip_directory : true)

# 設定檔案權限
install_subdir('scripts', install_dir : get_option('bindir'),
               install_mode : ['rwxr-xr-x'])
```

## 原始碼參考

Meson 原始碼中 `install_subdir` 的實作位於 `mesonbuild/minstall.py`，使用 `pathlib.Path` 遞迴複製目錄結構，並透過 `shutil.copy2` 保留檔案元資料[^ref-source]。

---

[^ref-meson]: The Meson Build System. (n.d.). *Reference manual — Functions*. Retrieved 2026-09-18, from https://mesonbuild.com/Reference-manual_functions.html

[^ref-install]: The Meson Build System. (n.d.). *Installing*. Retrieved 2026-09-18, from https://mesonbuild.com/Installing.html

[^ref-source]: The Meson Build System. (n.d.). *Source code — minstall.py*. Retrieved 2026-09-18, from https://github.com/mesonbuild/meson/blob/master/mesonbuild/minstall.py