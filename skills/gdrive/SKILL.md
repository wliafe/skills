---
name: gdrive
description: 用本机已登录的 rclone 远程 gdrive 操作谷歌网盘：列目录、看配额、读、下载、上传、建文件夹、移动、复制、同步、生成分享链接、挂载、查重、进回收站。用户提到网盘、云盘、Google Drive、gdrive，或要在网盘里找、传、下、整理文件时就用。
compatibility: 需要本机 /opt/homebrew/bin/rclone，以及已配置的远程 gdrive。
---

# 谷歌网盘

这台电脑的网盘已经用 rclone 登录。所有网盘操作都走这条远程。

- 命令：`/opt/homebrew/bin/rclone`
- 远程名：`gdrive`
- 配置在 `/Users/wliafe/.config/rclone/rclone.conf` 的 `[gdrive]`，权限是完整网盘（`scope = drive`）
- 不要读这个配置文件。里面有 client secret 和 token，写进聊天或记忆都会泄露。
- 盘里有什么以当场列出的为准。技能不记录文件夹、文件名或容量。

路径写成 `gdrive:文件夹/文件`。有空格或中文就加引号。

## 先看再改

改之前先列出目标路径，确认文件在。网盘很大时不要一次拉全盘。

```sh
rclone lsf "gdrive:某文件夹" --max-depth 2
rclone lsd "gdrive:"
rclone lsjson "gdrive:某文件夹" --max-depth 1
rclone size "gdrive:某文件夹"
rclone about gdrive:
rclone tree "gdrive:某文件夹" --level 2
```

`lsf` 列名字，`lsd` 只列文件夹，`lsjson` 带大小和时间，`about` 是配额。

## 读和下载

`cat` 只适合文本。PDF、Office、图片先下到本地再读。

```sh
rclone cat "gdrive:某文件夹/笔记.md"
rclone copyto "gdrive:某文件夹/a.pdf" /tmp/a.pdf
rclone copy "gdrive:某文件夹" ./local-dir
```

`copyto` 是单个文件，`copy` 是整个目录，已相同的文件会跳过。

## 写入

上传、移动、删除、清空之前，先说出要改的路径。

```sh
rclone copyto ./local.pdf "gdrive:某文件夹/a.pdf"
rclone copy "./一批文件" "gdrive:某文件夹/一批文件"
rclone copyurl "https://例子/a.pdf" "gdrive:某文件夹/a.pdf"
rclone mkdir "gdrive:某文件夹/新文件夹"
rclone moveto "gdrive:某文件夹/a.pdf" "gdrive:另一文件夹/a.pdf"
rclone move "gdrive:某文件夹/一批" "gdrive:另一文件夹/一批"
rclone touch "gdrive:某文件夹/空文件.txt"
rclone deletefile "gdrive:某文件夹/a.pdf"
rclone delete "gdrive:某文件夹"
rclone rmdir "gdrive:某空文件夹"
rclone purge "gdrive:某文件夹"
```

`deletefile` 删一个文件。`delete` 删目录里的文件，留下文件夹。`rmdir` 只删空文件夹。`purge` 连文件夹带内容一起删。`sync` 会把目标改成和来源一样，目标里多出来的文件会被删掉，所以只在用户明确说「同步」时用：

```sh
rclone sync ./local-dir "gdrive:某文件夹"
```

这份远程默认把删除放进网盘回收站（`--drive-use-trash`）。要看回收站里的东西：

```sh
rclone lsf "gdrive:" --drive-trashed-only
```

恢复要去 drive.google.com 的回收站。rclone 没有恢复命令。

## 链接、共享和核对

```sh
rclone link "gdrive:某文件夹/a.pdf"
rclone backend shortcut gdrive: 源路径 快捷方式路径
rclone check ./local-dir "gdrive:某文件夹"
rclone sha1sum "gdrive:某文件夹"
rclone dedupe "gdrive:某文件夹"
rclone lsf "gdrive:" --drive-shared-with-me
rclone lsf "gdrive:" --drive-starred-only
```

`link` 生成的是能打开的分享链接。`dedupe` 会改文件，先加 `--dry-run` 给用户看。别人共享过来的文件不在普通目录里，要加 `--drive-shared-with-me`。

Google 文档、表格、幻灯片不是普通文件。下载时 rclone 会按扩展名导出；要指定格式就加 `--drive-export-formats docx,xlsx,pptx,pdf`。

## 挂载

用户要把网盘当成一个本地文件夹时再挂载，不要为了读一个文件去挂。

```sh
mkdir -p /tmp/gdrive
rclone mount gdrive: /tmp/gdrive --daemon
```

用完卸掉：`umount /tmp/gdrive`。

## 不要做的事

- 不要读 `rclone.conf`，不要重新 `rclone config`，不要换 client id。登录已经完成，命令失败时查路径和权限，不重做授权。
- 不要 `rclone selfupdate`。
- 这个远程能改用户网盘里的文件。破坏性命令先说明路径。
