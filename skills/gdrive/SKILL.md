---
name: gdrive
description: 通过本地同步目录 /Users/wliafe/Google Drive（macOS Google Drive 桌面端挂载）访问与整理谷歌网盘：列目录、查文件名、读文件、复制到本地、上传写入、建文件夹、移动重命名、删除，以及识别本地目录做不到的云端操作边界。用户提到网盘、云盘、Google Drive、gdrive，或要在网盘里找、传、下、整理文件时使用。
compatibility: 需要 macOS 上已登录并运行 Google Drive 桌面端（/Applications/Google Drive.app），且 /Users/wliafe/Google Drive 可访问。
---

# 谷歌网盘（本地同步目录）

这台电脑通过 **Google Drive 桌面端**将谷歌网盘以 macOS File Provider（CloudStorage）形式同步到本地目录，不再使用 `rclone`。

- 根目录：`/Users/wliafe/Google Drive`（指向 `/Users/wliafe/Library/CloudStorage/GoogleDrive-<账号邮箱>` 的软链接）
- 我的云端硬盘：`/Users/wliafe/Google Drive/我的云端硬盘`（绝大多数用户文件都在这一层下）
- 根目录其他特殊目录：`.Trash`、`.shortcut-targets-by-id`、`.Encrypted`（系统/同步内部目录，不要手动写入或改动）
- 盘里有什么以当场列出的为准；技能本身不记录具体业务文件夹、文件名或容量。

## 访问前的确认与流式传输规则

1. **确认桌面端在线**：若目录为空或报权限/挂载错误，先检查 Google Drive 桌面端是否在运行：
   ```sh
   pgrep -x "Google Drive"
   ```
2. **路径加引号**：路径包含空格和中文，shell 命令里一律加双引号。
3. **软链接与 `find`**：`/Users/wliafe/Google Drive` 本身是软链接。用 `find` 遍历根目录时必须在末尾加斜杠（`find "/Users/wliafe/Google Drive/" ...`），或直接从 `"/Users/wliafe/Google Drive/我的云端硬盘"` 开始，否则 `find` 只会输出软链接自身。
4. **按需流式下载（`dataless`）**：桌面端处于流式传输模式，`ls -laO` 可见 `compressed,dataless` 标记（`stat` 能看到真实逻辑字节数 `st_size`，但本地占用块为 0）。
   - `ls`、`find`、`glob`、`stat` 只读元数据，**不会触发下载**。
   - 一旦读取文件内容（`read` 工具、`cat`、`cp`、`shasum`、内容 `grep`），系统会实时从云端拉取该文件（需要网络与代理正常）。
   - **禁止对网盘根目录或大量 PDF/数据集目录跑无过滤的全文递归搜索（如 `grep -r`）**，否则会把海量云端文件全部拉到本地。

## 先看再改（浏览与查找）

改动前先列出目标路径确认文件存在：

```sh
ls -la "/Users/wliafe/Google Drive/我的云端硬盘"
ls -laO "/Users/wliafe/Google Drive/我的云端硬盘/某文件夹"
find "/Users/wliafe/Google Drive/我的云端硬盘" -maxdepth 2 -type d
find "/Users/wliafe/Google Drive/我的云端硬盘" -iname "*关键词*"
stat "/Users/wliafe/Google Drive/我的云端硬盘/某文件夹/a.pdf"
```

- `ls -laO` 可分辨文件是已落地还是 `dataless`（未下载）。
- 不要用 `du` 统计云端文件夹真实体积（`dataless` 文件占用块为 `0`，统计结果不准）；单文件大小看 `ls -lh` 或 `stat`。

## 读和取到本地

- **纯文本 / Markdown / 代码**：先看大小，确认不大后直接用 `read` 工具或 `cat` 读取：
  ```sh
  cat "/Users/wliafe/Google Drive/我的云端硬盘/某文件夹/笔记.md"
  ```
- **PDF、Office、图片或大文件**：先复制到当前工作区或 `/tmp`，再用对应阅读或处理工具操作：
  ```sh
  cp "/Users/wliafe/Google Drive/我的云端硬盘/某文件夹/a.pdf" /tmp/a.pdf
  cp -R "/Users/wliafe/Google Drive/我的云端硬盘/某文件夹" ./local-dir
  ```
- **Google 在线文档（`.gdoc`、`.gsheet`、`.gslides`）**：
  - 它们在本地只是百余字节的 JSON 指针（内含 `doc_id` 与禁止编辑警告），**不是** `.docx`/`.xlsx`/`.pptx` 文件。
  - **绝对不要直接编辑或覆盖 `.gdoc`/`.gsheet`/`.gslides` 文件**，也不能用本地 Office 工具直接解析。需要内容时可读取其中的 `doc_id` 提示用户在浏览器打开或由用户在网页端导出。

## 写入、移动、整理与删除

上传、写入、覆盖、移动、重命名、删除前，**先向用户说明目标路径并确认**（特别是会覆盖同名文件或删除的操作）。

```sh
mkdir -p "/Users/wliafe/Google Drive/我的云端硬盘/某文件夹/新文件夹"
cp ./local.pdf "/Users/wliafe/Google Drive/我的云端硬盘/某文件夹/a.pdf"
cp -R "./一批文件" "/Users/wliafe/Google Drive/我的云端硬盘/某文件夹/"
mv "/Users/wliafe/Google Drive/我的云端硬盘/某文件夹/a.pdf" "/Users/wliafe/Google Drive/我的云端硬盘/另一文件夹/a.pdf"
```

- **避免静默覆盖**：`cp` 前先用 `ls` 确认目标不存在同名文件，或加 `-i`/`-n`（确认需要覆盖时再覆盖）。
- **同步目录**：仅在用户明确要求「同步」时用 `rsync -av --delete`（涉及 `--delete` 时必须先跑 `--dry-run` 给用户确认再执行）。
- **删除**：删除本地同步目录中的文件会同步删除云端对应文件。默认避免直接 `rm -rf`；确认要删时优先使用 macOS `/usr/bin/trash` 移入废纸篓，或在用户明确确认后执行：
  ```sh
  /usr/bin/trash "/Users/wliafe/Google Drive/我的云端硬盘/某文件夹/a.pdf"
  ```
  如需恢复，在 `drive.google.com` 回收站或本机废纸篓核查恢复。

## 本地目录的能力边界（这些事做不到）

当前已不再配置 `rclone`，以下云端元数据或权限操作**无法通过本地目录完成**：

- **查看云端剩余空间 / 总配额**：`df -h "/Users/wliafe/Google Drive"` 显示的是本机系统盘（Macintosh HD）的容量，不是 Google Drive 云端配额。需让用户在 Google Drive 桌面端菜单栏图标或 `https://drive.google.com` 查看。
- **生成/修改分享链接与共享权限**、**完整查看「与我共享」列表**、**直接导出 `.gdoc`/`.gsheet`/`.gslides` 为 Office/PDF 格式**：需在 Finder 中对文件右键通过 Google Drive 菜单操作，或在 `https://drive.google.com` 网页端处理。

遇到上述需求时，直接如实告知本地目录的边界与网页端/桌面端入口，不要假装 `df`/`du` 是云端配额，也不要调用 `rclone`。

## 不要做的事

- 不要调用 `rclone`（本机已退出登录并卸载）。
- 不要把 `df -h` 或 `du` 的输出当作 Google Drive 云端配额或云端真实体积。
- 不要直接修改、写回 `.gdoc`、`.gsheet`、`.gslides` 指针文件，也不要改动 `.Encrypted`、`.shortcut-targets-by-id`、`.Trash`。
- 不要对网盘大目录执行无过滤的全文递归扫描（如 `grep -r`），避免触发批量 `dataless` 下载。
- 不要在未向用户说明路径的情况下覆盖或删除网盘文件。
