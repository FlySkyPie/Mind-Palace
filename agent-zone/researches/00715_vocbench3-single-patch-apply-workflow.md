# VocBench 3：一個 patch、`git apply`、`docker compose up` 就得服務

## 背景

`vocbench3-docker` 儲存庫的 `docker-compose.yml` 定義了兩個服務，兩個都只有 `build:`、沒有 `image:`[^vb3-compose]，所以 `docker compose up` 會**把兩個容器一起建置並啟動**[^compose-up]：

- `vocbench3` 服務建置 `alpine/Dockerfile`，需要 VB3 發行版 ZIP[^vb3-dockerfile]
- `graphdb` 服務建置 `helpers/graphdb-with-st-sails/Dockerfile`，需要同一個 ZIP **加上** Lucene FTS 外掛 ZIP[^vb3-graphdb-dockerfile]

官方這兩個 Dockerfile 都用 `COPY` 從建置脈絡取檔[^docker-copy]，等於要求你先手動下載 163 MiB 的 ZIP 並複製進目錄。兩個服務都要建，所以兩個檔案都得改——**這是同一份 patch 的必要內容，不是二選一**。

Lucene 外掛沒有現成可下載的發行檔（該專案 releases 為空[^fts-gh]），因此改成在容器內用 Maven 從原始碼編譯。

## 關鍵陷阱：兩個檔案的行尾不同

這兩個 Dockerfile 的行尾不一致：

| 檔案 | 行尾 |
|---|---|
| `alpine/Dockerfile` | LF |
| `helpers/graphdb-with-st-sails/Dockerfile` | **CRLF** |

`git apply` 會逐字元比對被移除的行。若 patch 的 `-` 行是 LF 而原檔是 CRLF，就會得到：

```
error: patch failed: helpers/graphdb-with-st-sails/Dockerfile:10
error: helpers/graphdb-with-st-sails/Dockerfile: patch does not apply
```

所以**套用前先把行尾統一**。這一步不能省。

## 執行

```bash
git clone https://bitbucket.org/art-uniroma2/vocbench3-docker.git
cd vocbench3-docker
git checkout e52abee8a366703bd188128eb0cab9a746f146a6

sed -i 's/\r$//' helpers/graphdb-with-st-sails/Dockerfile

git apply --unidiff-zero --recount - <<'PATCH'
diff --git a/alpine/Dockerfile b/alpine/Dockerfile
--- a/alpine/Dockerfile
+++ b/alpine/Dockerfile
@@ -9,1 +9,1 @@
-COPY vocbench3-${vb_version}-full.zip /tmp
+ADD https://bitbucket.org/art-uniroma2/vocbench3/downloads/vocbench3-${vb_version}-full.zip /tmp/
diff --git a/helpers/graphdb-with-st-sails/Dockerfile b/helpers/graphdb-with-st-sails/Dockerfile
--- a/helpers/graphdb-with-st-sails/Dockerfile
+++ b/helpers/graphdb-with-st-sails/Dockerfile
@@ -10,8 +10,14 @@
-COPY vocbench3-${vb_version}-full.zip .
-COPY lucene-fts-plugin-graphdb-plugin.zip .
-
-RUN unzip -j "/tmp/vocbench3-${vb_version}-full.zip" \
-        "semanticturkey-${st_version}/lib/st-changetracking-sail-${st_version}.jar" \
-        "semanticturkey-${st_version}/lib/st-trivial-inference-sail-${st_version}.jar" \
-        -d /tmp/vb_lib && \
-    unzip -q "/tmp/lucene-fts-plugin-graphdb-plugin.zip" -d /tmp/fts_plugin
+ADD https://bitbucket.org/art-uniroma2/vocbench3/downloads/vocbench3-${vb_version}-full.zip /tmp/
+
+RUN unzip -j "/tmp/vocbench3-${vb_version}-full.zip" \
+        "semanticturkey-${st_version}/lib/st-changetracking-sail-${st_version}.jar" \
+        "semanticturkey-${st_version}/lib/st-trivial-inference-sail-${st_version}.jar" \
+        -d /tmp/vb_lib
+
+FROM maven:3-eclipse-temurin-21 AS plugin
+
+WORKDIR /src
+
+ADD https://github.com/Ontotext-AD/graphdb-lucene-fts-plugin.git#master .
+
+RUN mvn -q clean package
@@ -25,1 +31,3 @@
-COPY --from=unzipper /tmp/fts_plugin /opt/graphdb/dist/lib/plugins/
+COPY --from=plugin /src/target/lucene-fts-plugin-graphdb-plugin.zip /tmp/plugin.zip
+
+RUN unzip -q /tmp/plugin.zip -d /opt/graphdb/dist/lib/plugins/ && rm /tmp/plugin.zip
PATCH

docker compose up -d --build
```

完成後開 `http://localhost:1979/vocbench3`[^vb3-readme]。全程沒有下載任何 ZIP、沒有把任何檔案複製進目錄。

要改成「先驗證再套用」，把 `git apply` 換成 `git apply --check`（整段其餘不動）。

## 為什麼 patch 長這樣

- **`sed` 先統一 LF**：見上一節。這是讓 patch 能套上的前提。
- **`--unidiff-zero`**：允許沒有脈絡行的 patch[^gapply]。這裡刻意不放脈絡行——脈絡行若剛好是空白行，複製貼上時那個半形空格很容易被吃掉而讓 patch 損毀。
- **`--recount`**：讓 `git apply` 不採信行數標頭，改由內容自行推斷[^gapply]。
- **`<<'PATCH'` 的單引號**：必要，否則 `${vb_version}` 與行尾反斜線會被 shell 先吃掉。
- **`-p1`（預設）**：patch 的 `a/`、`b/` 前綴會被剝掉一層[^gapply]。

## 每一行在做什麼

| 改動 | 效果 |
|---|---|
| `COPY ... /tmp` → `ADD <url> /tmp/` | Docker 自己下載，不再需要手動放入。目的地結尾斜線讓檔名由 URL 推斷[^docker-add-url] |
| 移除 `COPY lucene-fts-plugin-graphdb-plugin.zip` | 不再需要那個手動 ZIP |
| 新增 `FROM maven:3-eclipse-temurin-21 AS plugin` | 另開一個階段編譯外掛；多階段建置只把產物帶進最終映像[^docker-multistage] |
| `ADD ...graphdb-lucene-fts-plugin.git#master .` | 以 Git URL 為來源，`#<ref>` 指定分支標籤或 commit[^docker-add-git] |
| `RUN mvn -q clean package` | 外掛的建置指令，產物為 `target/lucene-fts-plugin-graphdb-plugin.zip`[^fts-readme] |
| `COPY --from=plugin ... /tmp/plugin.zip` ＋ `RUN unzip ... plugins/` | 把編出來的 ZIP 解進 GraphDB 的 `lib/plugins/`；`COPY --from` 可從其他階段取檔[^docker-copy-from] |

Java 版本：Maven 階段用 21，因為官方要求外掛以 Semantic Turkey 所用 Java 版本（目前 Java 21）建置[^vb3-readme]；VB3 容器本身跑的是 Java 17[^vb3-dockerfile]，兩者用途不同。GraphDB 基底 `ontotext/graphdb:11.2.1` 與 `.env` 的 `GDB_VERSION` 相符[^vb3-env]。

`#master` 追蹤的是分支，不是不可變的版本；要可重現請換成完整 commit SHA[^docker-add-git]。

## 出錯時

| 症狀 | 指令 |
|---|---|
| `patch does not apply` | 就是行尾問題，回去跑那行 `sed -i 's/\r$//'`；或直接把檔案換掉（見下） |
| 想只套可套的部分 | `git apply --reject`，失敗的 hunk 留在 `*.rej`[^gapply] |
| 建置快取拿到舊結果 | `docker compose build --no-cache` |
| 建置時連外被擋 | `docker compose build --build-arg HTTP_PROXY=... --build-arg HTTPS_PROXY=...`；代理參數不進快取與 `docker history`[^docker-variables] |
| 想釘死下載內容 | 改成 `ADD --checksum=sha256:<hash> <url> /tmp/`；HTTP 來源只支援 SHA-256[^docker-add-checksum] |
| 換版本 | 改 `.env` 的 `VB_VERSION`、`ST_VERSION`、`GDB_VERSION`；`ARG` 由 `--build-arg` 覆寫且不寫入最終映像[^docker-arg] |

### 不想跟 patch 搏鬥時

`graphdb` 那個檔案本來就要大幅改寫，直接整檔覆蓋最省事，完全不受行尾影響：

```bash
cat > helpers/graphdb-with-st-sails/Dockerfile <<'EOF'
ARG gdb_version

FROM alpine:3.24.2 AS unzipper

ARG st_version
ARG vb_version

WORKDIR /tmp

ADD https://bitbucket.org/art-uniroma2/vocbench3/downloads/vocbench3-${vb_version}-full.zip /tmp/

RUN unzip -j "/tmp/vocbench3-${vb_version}-full.zip" \
        "semanticturkey-${st_version}/lib/st-changetracking-sail-${st_version}.jar" \
        "semanticturkey-${st_version}/lib/st-trivial-inference-sail-${st_version}.jar" \
        -d /tmp/vb_lib

FROM maven:3-eclipse-temurin-21 AS plugin

WORKDIR /src

ADD https://github.com/Ontotext-AD/graphdb-lucene-fts-plugin.git#master .

RUN mvn -q clean package

FROM ontotext/graphdb:${gdb_version}

WORKDIR /opt/graphdb/dist/lib

COPY --from=unzipper /tmp/vb_lib/*.jar /opt/graphdb/dist/lib/
COPY --from=plugin /src/target/lucene-fts-plugin-graphdb-plugin.zip /tmp/plugin.zip

RUN unzip -q /tmp/plugin.zip -d /opt/graphdb/dist/lib/plugins/ && rm /tmp/plugin.zip
EOF

sed -i 's|^COPY vocbench3-${vb_version}-full.zip /tmp$|ADD https://bitbucket.org/art-uniroma2/vocbench3/downloads/vocbench3-${vb_version}-full.zip /tmp/|' alpine/Dockerfile

docker compose up -d --build
```

（`alpine/Dockerfile` 是 LF，`sed` 抓得到結尾；`graphdb` 那個用 heredoc 整檔寫掉，行尾由 heredoc 決定。）

## 參考文獻

[^vb3-compose]: ART Group. (n.d.). *docker-compose.yml*. Retrieved 2026-11-07, from https://bitbucket.org/art-uniroma2/vocbench3-docker/raw/HEAD/docker-compose.yml

[^vb3-env]: ART Group. (n.d.). *.env*. Retrieved 2026-11-07, from https://bitbucket.org/art-uniroma2/vocbench3-docker/raw/HEAD/.env

[^vb3-dockerfile]: ART Group. (n.d.). *alpine/Dockerfile*. Retrieved 2026-11-07, from https://bitbucket.org/art-uniroma2/vocbench3-docker/raw/HEAD/alpine/Dockerfile

[^vb3-graphdb-dockerfile]: ART Group. (n.d.). *helpers/graphdb-with-st-sails/Dockerfile*. Retrieved 2026-11-07, from https://bitbucket.org/art-uniroma2/vocbench3-docker/raw/HEAD/helpers/graphdb-with-st-sails/Dockerfile

[^vb3-readme]: ART Group. (n.d.). *VocBench 3 Docker — README*. Retrieved 2026-11-07, from https://bitbucket.org/art-uniroma2/vocbench3-docker/raw/HEAD/README.md

[^fts-readme]: Ontotext-AD. (n.d.). *GraphDB Lucene FTS Plugin — README*. Retrieved 2026-11-07, from https://raw.githubusercontent.com/Ontotext-AD/graphdb-lucene-fts-plugin/master/README.md

[^fts-gh]: Ontotext-AD. (2026). *graphdb-lucene-fts-plugin repository metadata*. Retrieved 2026-11-07, from https://api.github.com/repos/Ontotext-AD/graphdb-lucene-fts-plugin

[^gapply]: Git Project. (n.d.). *git-apply — Apply a patch to files and/or to the index*. Retrieved 2026-11-07, from https://git-scm.com/docs/git-apply

[^docker-add-url]: Docker Inc. (n.d.). *Dockerfile reference — Adding files from a URL*. Retrieved 2026-11-07, from https://docs.docker.com/reference/dockerfile/#adding-files-from-a-url

[^docker-add-git]: Docker Inc. (n.d.). *Dockerfile reference — Adding files from a Git repository*. Retrieved 2026-11-07, from https://docs.docker.com/reference/dockerfile/#adding-files-from-a-git-repository

[^docker-add-checksum]: Docker Inc. (n.d.). *Dockerfile reference — ADD --checksum*. Retrieved 2026-11-07, from https://docs.docker.com/reference/dockerfile/#add---checksum

[^docker-copy]: Docker Inc. (n.d.). *Dockerfile reference — COPY*. Retrieved 2026-11-07, from https://docs.docker.com/reference/dockerfile/#copy

[^docker-copy-from]: Docker Inc. (n.d.). *Dockerfile reference — COPY --from*. Retrieved 2026-11-07, from https://docs.docker.com/reference/dockerfile/#copy---from

[^docker-arg]: Docker Inc. (n.d.). *Dockerfile reference — ARG*. Retrieved 2026-11-07, from https://docs.docker.com/reference/dockerfile/#arg

[^docker-multistage]: Docker Inc. (n.d.). *Multi-stage builds*. Retrieved 2026-11-07, from https://docs.docker.com/build/building/multi-stage/

[^docker-variables]: Docker Inc. (n.d.). *Build variables*. Retrieved 2026-11-07, from https://docs.docker.com/build/building/variables/

[^compose-up]: Docker Inc. (n.d.). *docker compose up* [Source file]. Retrieved 2026-11-07, from https://github.com/docker/compose/blob/main/docs/reference/compose_up.md
