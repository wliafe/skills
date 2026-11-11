# skills

给智能体用的技能。一个目录一个技能，目录名就是技能的 `name`，每个目录里有一份 `SKILL.md`。技能只写操作流程，不把某台电脑上的文件、密钥或账号写进去。

四个技能都是 MIT 许可证，见根目录的 [LICENSE](LICENSE)。

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
| [gdrive](skills/gdrive) | 用户提到网盘、云盘、Google Drive、gdrive，或要在网盘里找、传、下、整理文件 | 本机 rclone，已配好远程 `gdrive` |
| [drawio-skill](skills/drawio-skill) | 要用 Next AI Draw.io 画图、改图、插图片、导出 | Next AI Draw.io MCP（`@next-ai-drawio/mcp-server`） |
| [notion-maintenance-skill](skills/notion-maintenance-skill) | 要维护 Notion。只点名本技能、没有其他说明时，默认更新 AI 工作台 | Notion MCP |
| [xrdp-desktop](skills/xrdp-desktop) | 要开、关远程桌面，看桌面状态，或连上就断、闪退；要把桌面改成按需开启 | 目标机已装 xrdp + XFCE，本机能 SSH 上去 |

### gdrive

用本机已登录的 rclone 远程 `gdrive` 操作谷歌网盘。远程名和命令路径写在技能里，盘里有什么以当场列出的为准，不记录文件夹、文件名或容量。

覆盖的操作：列目录、看配额、读文本、下载、上传、建文件夹、移动、复制、同步、生成分享链接、挂载、查重、进回收站。删除进回收站，恢复只能去 [drive.google.com](https://drive.google.com)。

不读 rclone 的配置文件，里面有 client secret 和 token。不跑 `rclone config`，也不跑 `rclone selfupdate`。

### drawio-skill

用 Next AI Draw.io MCP 执行已经确定的绘图任务：检查连接、引导授权和安装、开会话、改图中的对象、控制插入图片的大小，再导出 drawio、PNG、SVG 和可编辑的 drawio.svg。图按主题分版本管理。

技能只管工具怎么用、怎么交付，不规定构图、配色和字体。连接、图片限制和绘图约定在 `references/` 里。

### notion-maintenance-skill

用 Notion MCP 维护 Notion。改之前先找到并读「使用说明」，再按那里的要求做。规则正文只在 Notion 里，不在这个技能里。

只点名本技能、没有其他说明时，默认收集当前与近期对话，更新 AI 工作台。带了其他说明，就按说明维护工作台、知识库、工作方式或项目页。

### xrdp-desktop

把已装好 xrdp 和 XFCE 的 Linux 机器改成「需要时才开」：不开机自启，用前手动开，开之前先清掉上次残留的会话，空闲超过设定分钟数自动关。

技能不写某台机器的地址、端口、用户名和密码。连哪台、端口多少、空闲多久，按当场情况和用户说的来。连不上时先读 `/var/log/xrdp.log` 和 `/var/log/xrdp-sesman.log` 再下结论。

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
└── xrdp-desktop/
    └── SKILL.md
```
