# skills

给智能体用的技能。一个目录一个技能，目录名就是技能的 `name`，每个目录里有一份 `SKILL.md`。技能只写操作流程，不把某台电脑上的文件、密钥或账号写进去。

本仓库采用 MIT 许可证，见根目录的 [LICENSE](LICENSE)。

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
| [notion-maintenance-skill](skills/notion-maintenance-skill) | 沿目录树按需读取 Notion 内容、排查 MCP 连接，或在写前定位 Notion 实时维护指南 | Notion MCP |
| [team-handoff](skills/team-handoff) | 用户要求用团队，或需要把任务交给队友并行或接力执行 | 支持子智能体和共享任务板的 DeepSeek Harness |
| [read-project](skills/read-project) | 了解陌生项目，或为改代码、配置环境、创建 worktree 补齐项目背景 | 可读取目标项目文件 |
| [personal-coding-style](skills/personal-coding-style) | 编写、重构或审查代码时，按个人习惯判断职责划分、函数拆分、跨项目复用和状态归属 | 无 |

### gdrive

通过 Google Drive 桌面端的本地同步目录（macOS File Provider）操作谷歌网盘，用 `ls`、`cp`、`mv` 这类普通命令，不用 rclone。盘里有什么以当场列出的为准，不记录文件夹、文件名或容量。

覆盖的操作：列目录、查文件、读文本、复制到本地、上传、建文件夹、移动重命名、删除。文件按需流式下载（`dataless`），读内容才会拉取，所以不对大目录做无过滤的全文搜索。本地目录做不到的事（云端配额、分享链接和权限、`.gdoc` 等在线文档导出）技能会如实说明，并指向桌面端或 [drive.google.com](https://drive.google.com)。

不改写 `.gdoc`、`.gsheet`、`.gslides` 指针文件，也不动 `.Encrypted`、`.shortcut-targets-by-id`、`.Trash`。覆盖和删除前先向用户确认路径。

### drawio-skill

用 Next AI Draw.io MCP 执行已经确定的绘图任务：检查连接、引导授权和安装、开会话、改图中的对象、控制插入图片的大小，再导出 drawio、PNG、SVG 和可编辑的 drawio.svg。图按主题分版本管理。

技能只管工具怎么用、怎么交付，不规定构图、配色和字体。连接、图片限制和绘图约定在 `references/` 里。

### notion-maintenance-skill

用 Notion MCP 沿目录树逐层定位并按需读取 Notion 内容，同时提供只读 MCP 连接检查与排查。规则正文、收集范围与结构演进规范只在 Notion 实时维护指南里，不内置在本技能中。

仅点名本技能、没有其他说明时，只做只读连接检查或确认读取入口，不默认收集对话或写入。只有明确要求写入或整理时，才先沿目录树找 Notion 实时维护指南并按其执行。

### read-project

先读完整 README，再核对环境和启动方式、理解关键目录职责，最后沿一条代表性路径读代码。多入口仓库先给根级概览，按本次目标选择一条主线。重要结论附出处，未运行的命令注明核验状态。

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
│   └── references/        MCP 检查与连接、目录树读取方法
├── personal-coding-style/
│   └── SKILL.md           个人编码习惯与设计取舍
├── read-project/
│   └── SKILL.md           从项目文档、启动入口和目录职责走到核心代码
└── team-handoff/
    └── SKILL.md           调查与执行模板、会话内复用和任务交接
```
