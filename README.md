# Skills

[青枫数科](https://github.com/MapleAI-Tech) 旗下的开源 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) Skills 集合。

## Skills

| Skill | Description |
|-------|-------------|
| [chuxin](./skills/chuxin/) | 产品大脑 — 项目初始化、每日简报、外部研究，驱动产品持续演进 |
| [doc-align](./skills/doc-align/) | 文档对齐 — 扫描、索引、整理项目中的所有 Markdown 文件，统一归档到 /docs 目录 |

## Install

```bash
git clone https://github.com/MapleAI-Tech/skills.git
cp -r skills/skills/chuxin ~/.claude/skills/
cp -r skills/skills/doc-align ~/.claude/skills/
```

## Usage

- `/chuxin init` — 初始化产品路线图
- `/chuxin` / `/chuxin briefing` — 每日简报，追踪进度
- `/chuxin research` — 外部竞品与技术趋势研究

## doc-align

- `/doc-align init` — 扫描项目 md 文件，生成文档索引
- `/doc-align` / `/doc-align sync` — 将散落的 md 文件归档到 /docs 目录
- `/doc-align consolidate` — 梳理文档、按主题重命名、合并去重，保留最终版本
- `/doc-align index` — 仅刷新索引
- `/doc-align check` — 检查归档状态

## License

[MIT](./LICENSE)
