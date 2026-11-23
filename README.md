# skills

给智能体用的技能。一个目录一个技能，目录名就是技能的 `name`，每个目录里有一份 `SKILL.md`。技能只写操作流程，不把某台电脑上的文件、密钥或账号写进去。

三个技能都是 MIT 许可证，见根目录的 [LICENSE](LICENSE)。

## 安装

用 [skills.sh](https://skills.sh) 的命令行安装，不用克隆仓库。命令说明见 [skills CLI](https://github.com/vercel-labs/skills)。

```bash
# 列出这个仓库里的技能
npx skills add wliafe/skills --list

# 安装其中一个，安到当前项目
npx skills add wliafe/skills --skill gdrive

# 安装全部，放到用户目录，对所有已安装的智能体生效
npx skills add wliafe/skills --all -g
```

已安装的技能用 `npx skills update` 更新，用 `npx skills remove <技能名>` 卸载。

## 技能

| 技能 | 用在什么时候 | 依赖 |
| --- | --- | --- |
| [gdrive](skills/gdrive) | 用户提到网盘、云盘、Google Drive、gdrive，或要在网盘里找、传、下、整理文件 | macOS 上已登录的 Google Drive 桌面端，本地同步目录可访问 |
| [drawio-skill](skills/drawio-skill) | 要用 Next AI Draw.io 画图、改图、插图片、导出 | Next AI Draw.io MCP（`@next-ai-drawio/mcp-server`） |
| [notion-maintenance-skill](skills/notion-maintenance-skill) | 要维护 Notion。只点名本技能、没有其他说明时，默认更新 AI 工作台 | Notion MCP |

### gdrive

通过 Google Drive 桌面端的本地同步目录（macOS File Provider）操作谷歌网盘，用 `ls`、`cp`、`mv` 这类普通命令，不用 rclone。盘里有什么以当场列出的为准，不记录文件夹、文件名或容量。

覆盖的操作：列目录、查文件、读文本、复制到本地、上传、建文件夹、移动重命名、删除。文件按需流式下载（`dataless`），读内容才会拉取，所以不对大目录做无过滤的全文搜索。本地目录做不到的事（云端配额、分享链接和权限、`.gdoc` 等在线文档导出）技能会如实说明，并指向桌面端或 [drive.google.com](https://drive.google.com)。

不改写 `.gdoc`、`.gsheet`、`.gslides` 指针文件，也不动 `.Encrypted`、`.shortcut-targets-by-id`、`.Trash`。覆盖和删除前先向用户确认路径。

### drawio-skill

用 Next AI Draw.io MCP 执行已经确定的绘图任务：检查连接、引导授权和安装、开会话、改图中的对象、控制插入图片的大小，再导出 drawio、PNG、SVG 和可编辑的 drawio.svg。图按主题分版本管理。

技能只管工具怎么用、怎么交付，不规定构图、配色和字体。连接、图片限制和绘图约定在 `references/` 里。

### notion-maintenance-skill

用 Notion MCP 维护 Notion。改之前先找到并读「使用说明」，再按那里的要求做。规则正文只在 Notion 里，不在这个技能里。

只点名本技能、没有其他说明时，默认收集当前与近期对话，更新 AI 工作台。带了其他说明，就按说明维护工作台、知识库、工作方式或项目页。

## 目录

```
skills/
├── drawio-skill/
│   ├── SKILL.md
│   └── references/        连接、图片与容量、绘图约定
├── gdrive/
│   ├── SKILL.md
│   └── evals/            只读测试题
├── notion-maintenance-skill/
│   ├── SKILL.md
│   └── references/        怎么找「使用说明」、默认同步工作台
```
