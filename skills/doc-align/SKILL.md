---
name: doc-align
description: 文档对齐 — 扫描、索引、整理项目中的所有 Markdown 文件，统一归档到 /docs 目录
---

<Purpose>
doc-align 是项目的"文档管家"。各种 AI 工具（Claude、Cursor、Copilot、Windsurf 等）会在项目中生成各自的 md 文件（CLAUDE.md、.cursorrules、AGENTS.md、SPEC.md 等），散落各处，难以维护。doc-align 通过扫描索引 + 归档整理，让所有 md 文件有迹可循、有处可归。
</Purpose>

<Use_When>
- 用户运行 `/doc-align init` — 首次初始化，扫描并索引所有 md 文件
- 用户运行 `/doc-align` 或 `/doc-align sync` — 将散落的 md 文件整理归档到 /docs 目录
- 用户运行 `/doc-align index` — 仅刷新 DOC_INDEX.md 索引，不做文件移动
- 用户运行 `/doc-align check` — 检查哪些 md 文件尚未归档
- 用户说"整理一下文档"、"对齐文档"、"md 文件太多了" — 自动匹配 sync 模式
</Use_When>

<Do_Not_Use_When>
- 用户想编辑某个 md 文件的内容 — 直接编辑，不需要 doc-align
- 用户想创建新的文档 — 直接创建到 /docs 即可，之后用 doc-align sync 更新索引
- 用户在做代码相关工作 — doc-align 只处理 .md 文件
</Do_Not_Use_When>

<Project_Structure>
doc-align 在当前工作目录下操作：

```
当前工作目录/
├── docs/                        # 统一文档目录（doc-align 管理）
│   ├── DOC_INDEX.md             # 文档总索引（按时间排序）
│   └── {归档的 md 文件}
├── CLAUDE.md                    # AI 工具配置文件（保留原位，索引中记录）
├── .cursorrules                 # AI 工具配置文件（保留原位，索引中记录）
└── ...其他散落的 .md 文件
```

**关键规则**：
- `/docs` 目录是文档归档的目标位置
- `DOC_INDEX.md` 是索引文件，记录项目中所有 md 文件的位置、用途、时间
- AI 工具的配置文件（CLAUDE.md、.cursorrules、.github/copilot-instructions.md 等）**保留原位**，仅在索引中记录，不移动
- 只处理 `.md` 扩展名的文件，不处理 `.mdx`、`.txt` 等
- 排序严格使用文件的 **修改时间**（mtime）作为主要排序依据，**创建时间**（birthtime）作为次要排序依据
</Project_Structure>

<AI_Config_Files>
以下文件属于 AI 工具配置文件，初始化时**保留原位**，仅在 DOC_INDEX.md 中索引记录：

| 文件 | 所属工具 | 处理方式 |
|------|---------|---------|
| CLAUDE.md | Claude Code | 索引记录，保留原位 |
| .claude/CLAUDE.md | Claude Code (子目录) | 索引记录，保留原位 |
| .cursorrules | Cursor | 索引记录，保留原位 |
| .cursor/rules | Cursor (规则目录) | 索引记录，保留原位 |
| .github/copilot-instructions.md | GitHub Copilot | 索引记录，保留原位 |
| .windsurfrules | Windsurf | 索引记录，保留原位 |
| AGENTS.md | oh-my-claudecode | 索引记录，保留原位 |
| ROADMAP.md | chuxin skill | 索引记录，保留原位 |
| README.md | 项目根目录 | 索引记录，保留原位 |
| LICENSE | - | 忽略，不索引 |

判断规则：如果文件名在上表中，或在 `.cursor/`、`.claude/`、`.github/` 目录内，标记为"AI 配置文件"，索引但保留原位。
</AI_Config_Files>

<Commands>

### `/doc-align init` — 初始化文档索引

**核心原则：只扫描索引，不移动任何文件。让用户先看清全貌。**

#### 步骤

1. **创建 /docs 目录** — 如果不存在则创建：
   ```bash
   mkdir -p docs
   ```

2. **全面扫描** — 使用 Bash 获取项目中所有 .md 文件，同时获取每个文件的时间信息：
   ```bash
   find . -name "*.md" -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/vendor/*" -not -path "*/dist/*" -not -path "*/build/*" -printf "%T@\t%w@\t%p\n" | sort -n
   ```
   排除目录：node_modules、.git、vendor、dist、build、__pycache__、.next、.nuxt

3. **分类标记** — 将扫描到的文件分为三类：
   - **AI 配置文件**：参考 AI_Config_Files 表格，保留原位
   - **项目文档**：README.md、CHANGELOG.md 等，保留原位
   - **可归档文档**：散落的笔记、规范、设计文档等，可归档到 /docs

4. **获取时间信息** — 对每个文件获取：
   - 修改时间（mtime）：文件的最后修改时间
   - 创建时间（birthtime，如果文件系统支持）：文件的创建时间
   - 使用 `stat` 命令或 `find -printf` 获取

5. **生成 DOC_INDEX.md** — 严格按修改时间排序（最新在后），格式如下：

   ```markdown
   # DOC_INDEX — 文档总索引

   > 自动生成于 {日期时间}，由 doc-align 维护
   > 排序规则：按文件修改时间升序（最早的在前）

   ## 统计

   - 总文件数: {N}
   - AI 配置文件: {N}（保留原位）
   - 项目文档: {N}（保留原位）
   - 可归档文档: {N}（建议归档到 /docs）

   ## 文档列表

   | # | 文件路径 | 类型 | 修改时间 | 创建时间 | 归档位置 |
   |---|---------|------|---------|---------|---------|
   | 1 | {路径} | {类型} | {时间} | {时间} | {归档路径或"保留原位"} |
   | 2 | {路径} | {类型} | {时间} | {时间} | {归档路径或"保留原位"} |

   ## AI 配置文件（保留原位）

   | 文件路径 | 所属工具 | 修改时间 |
   |---------|---------|---------|
   | {路径} | {工具} | {时间} |

   ## 可归档文档

   以下文档可归档到 /docs 目录：

   | 文件路径 | 修改时间 | 建议归档路径 |
   |---------|---------|------------|
   | {路径} | {时间} | docs/{文件名} |
   ```

6. **输出初始化报告** — 告知用户：
   - 扫描到多少个 md 文件
   - 其中多少是 AI 配置文件（保留原位）
   - 多少是可以归档的文档
   - DOC_INDEX.md 已生成
   - 下一步可以运行 `/doc-align sync` 进行归档

### `/doc-align` 或 `/doc-align sync` — 文档对齐归档

**前提：docs/DOC_INDEX.md 存在。如果不存在，提示用户先运行 `/doc-align init`。**

**核心原则：归档不是移动，是复制+更新索引。原始文件可以选择保留或删除。**

#### 步骤

1. **读取索引** — 读取 `docs/DOC_INDEX.md`，了解当前状态

2. **重新扫描** — 检查是否有新增的 md 文件尚未索引

3. **归档操作** — 对"可归档文档"列表中的文件：

   a. 使用 `AskUserQuestion` 询问用户归档策略：
   - "复制到 /docs 并保留原文件"
   - "移动到 /docs（删除原文件）"
   - "跳过，仅索引记录"

   b. 对每个可归档文件执行操作：
   - 如果选择复制：`cp {原路径} docs/{文件名}`（如果 /docs 下已有同名文件，加后缀 `_2`、`_3` 等）
   - 如果选择移动：`mv {原路径} docs/{文件名}`
   - 更新 DOC_INDEX.md 中的归档位置

4. **更新索引** — 刷新 DOC_INDEX.md：
   - 更新文件时间信息
   - 更新归档位置
   - 添加新发现的文件

5. **输出对齐报告**：
   ```markdown
   ## 文档对齐完成

   - 归档了 {N} 个文件到 /docs
   - 更新了 {N} 个索引条目
   - 跳过了 {N} 个 AI 配置文件
   - 发现 {N} 个新文件已添加到索引
   ```

### `/doc-align index` — 刷新索引

仅刷新 DOC_INDEX.md，不执行任何文件操作。步骤同 init 的扫描和索引生成，但：
- 如果 DOC_INDEX.md 已存在，保留用户手动添加的备注
- 仅更新时间信息和文件列表

### `/doc-align check` — 检查归档状态

对比 DOC_INDEX.md 和实际文件系统：
- 列出已索引但文件已删除的条目
- 列出未索引的新文件
- 列出归档路径与实际不符的文件

</Commands>

<File_Conflict_Handling>
当归档文件与 /docs 下已有文件同名时：
1. 比较文件内容（使用 `diff`）
2. 如果内容相同：跳过，标记为"已归档"
3. 如果内容不同：保留两个版本，新文件加数字后缀（如 `design_2.md`），在 DOC_INDEX.md 中标注差异
</File_Conflict_Handling>

<Tool_Usage>
- 使用 `Bash` 运行 `find`、`stat` 命令获取文件列表和时间信息
- 使用 `Bash` 运行 `cp`、`mv` 命令执行归档操作
- 使用 `Read` 读取 DOC_INDEX.md 和 md 文件内容
- 使用 `Write` 创建/覆盖 DOC_INDEX.md
- 使用 `Edit` 更新 DOC_INDEX.md 的部分内容
- 使用 `Glob` 辅助查找 md 文件（`**/*.md`）
- 使用 `AskUserQuestion` 让用户选择归档策略
</Tool_Usage>

<Examples>
<Good>
Init 时的扫描报告：
```
扫描完成，共发现 23 个 .md 文件：
- 4 个 AI 配置文件（CLAUDE.md, .cursorrules, AGENTS.md, .github/copilot-instructions.md）→ 保留原位
- 2 个项目文档（README.md, CHANGELOG.md）→ 保留原位
- 17 个可归档文档（设计文档、会议记录、API 说明等）→ 建议归档到 /docs

DOC_INDEX.md 已生成到 docs/ 目录。
运行 /doc-align sync 开始归档。
```
Why good: 清晰分类，给用户下一步操作指引。
</Good>

<Good>
Sync 时处理冲突：
```
docs/api.md 已存在，与 src/API_NOTES.md 内容不同：
- docs/api.md: 修改于 2024-03-10，328 行
- src/API_NOTES.md: 修改于 2024-03-15，412 行

建议：将较新版本归档为 docs/api_v2.md，保留旧版本。
```
Why good: 主动发现冲突，给出建议方案，避免数据丢失。
</Good>

<Bad>
直接移动文件不通知：
```
（静默将所有 md 文件移动到 /docs）
```
Why bad: 用户可能不知道文件被移动了，破坏了其他工具的引用路径。
</Bad>

<Bad>
忽略时间排序：
```
（按文件名字母序排列所有文件）
```
Why bad: 用户明确要求按修改时间排序，时间线对于理解文档演进至关重要。
</Bad>
</Examples>

<Escalation_And_Stop_Conditions>
- 如果当前目录没有任何 .md 文件，告知用户"没有发现任何 Markdown 文件，无需初始化"
- 如果 /docs 目录不存在且 init 未执行，提示先运行 `/doc-align init`
- 如果扫描到的文件超过 100 个，提醒用户"文件较多，建议先 check 再 sync"
- AI 配置文件**永远不会被移动或删除**，只做索引记录
- 用户可以随时手动编辑 DOC_INDEX.md 添加备注
- 如果用户选择"跳过"某个文件，尊重用户选择，标记为"用户选择跳过"
</Escalation_And_Stop_Conditions>

<Final_Checklist>
- [ ] DOC_INDEX.md 严格按文件修改时间排序
- [ ] AI 配置文件（CLAUDE.md 等）仅索引，不移动
- [ ] 归档操作前询问用户策略（复制/移动/跳过）
- [ ] 同名文件冲突时有处理机制
- [ ] 扫描时排除 node_modules、.git、vendor、dist、build 等目录
- [ ] /docs 目录在当前工作目录下创建
- [ ] 每次操作后刷新 DOC_INDEX.md 的时间戳
</Final_Checklist>
