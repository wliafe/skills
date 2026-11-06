# skills

给智能体用的技能，一个目录一个。目录名就是技能的 `name`，每个目录里有一份 `SKILL.md`。

## 安装

用 [skills.sh](https://skills.sh) 的命令行安装，不用克隆仓库：

```bash
# 列出这个仓库里的技能
npx skills add wliafe/skills --list

# 安装其中一个，安到当前项目
npx skills add wliafe/skills --skill gdrive

# 安装全部，放到用户目录，对所有已安装的智能体生效
npx skills add wliafe/skills --all -g
```

更新已安装的技能用 `npx skills update`，卸载用 `npx skills remove <技能名>`。命令说明见 [skills CLI](https://github.com/vercel-labs/skills)。

| 目录 | 做什么 |
| --- | --- |
| [skills/drawio-skill](skills/drawio-skill) | 用 Next AI Draw.io MCP 检查连接、引导安装、改图，并导出 drawio、PNG、SVG 和可编辑的 drawio.svg。 |
| [skills/notion-maintenance-skill](skills/notion-maintenance-skill) | 用 Notion MCP 找到「使用说明」再维护 Notion。只点名本技能时，默认更新 AI 工作台。 |
| [skills/gdrive](skills/gdrive) | 用本机已登录的 rclone 远程 `gdrive` 操作谷歌网盘：列目录、传下载、整理、分享、进回收站。 |

三个技能都是 MIT 许可证，见根目录的 [LICENSE](LICENSE)。
