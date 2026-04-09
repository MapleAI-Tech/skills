---
name: doc-align
description: 文档对齐 — 扫描、索引、整理项目中的所有 Markdown 文件，统一归档到 /docs 目录
---

<Purpose>
doc-align 是项目的"文档管家"。各种 AI 工具（Claude、Cursor、Copilot、Windsurf 等）会在项目中生成各自的 md 文件，散落各处难以维护。doc-align 通过扫描索引 + 移动归档，让所有 md 文件有迹可循、有处可归。
</Purpose>

<Use_When>
- 用户运行 `/doc-align init` — 首次初始化，扫描并索引所有 md 文件
- 用户运行 `/doc-align` 或 `/doc-align sync` — 将散落的 md 文件归档到 /docs 目录
- 用户运行 `/doc-align index` — 仅刷新 DOC_INDEX.md 索引，不做文件移动
- 用户运行 `/doc-align check` — 检查哪些 md 文件尚未归档
- 用户说"整理一下文档"、"对齐文档"、"md 文件太多了" — 自动匹配 sync 模式
</Use_When>

<Do_Not_Use_When>
- 用户想编辑某个 md 文件的内容 — 直接编辑，不需要 doc-align
- 用户想创建新的文档 — 直接创建到 /docs 即可，之后用 /doc-align sync 更新索引
- 用户在做代码相关工作 — doc-align 只处理 .md 文件
</Do_Not_Use_When>

<File_Classification>
doc-align 将 md 文件分为四类，每类有明确的处理方式：

### 第一类：AI 配置文件（保留原位，仅索引）

| 文件 | 所属工具 |
|------|---------|
| CLAUDE.md | Claude Code |
| .claude/CLAUDE.md | Claude Code (子目录) |
| .cursorrules | Cursor |
| .cursor/rules/* | Cursor (规则目录) |
| .github/copilot-instructions.md | GitHub Copilot |
| .windsurfrules | Windsurf |
| AGENTS.md | oh-my-claudecode |
| ROADMAP.md | chuxin skill |

判断规则：文件名匹配上表，或在 `.cursor/`、`.claude/`、`.github/` 目录内 → AI 配置文件。

### 第二类：代码依赖文件（保留原位，仅索引）

被代码文件（.py, .ts, .js, .go, .rs, .java 等）通过 import/read/open/fopen 等方式引用的 md 文件。
典型场景：AI 提示词文件（如 prompts/system.md 被代码读取加载）。

检测方法：使用 Grep 在代码文件中搜索 md 文件路径。搜索范围排除 .md 文件本身。

### 第三类：项目标准文档（保留原位，仅索引）

严格名单，仅以下文件名视为项目标准文档：
- README.md
- CHANGELOG.md
- CONTRIBUTING.md
- SECURITY.md
- CODE_OF_CONDUCT.md
- AUTHORS.md
- LICENSE.md

不在此名单中的根目录 md 文件，归为可归档文档。

### 第四类：可归档文档（移动到 /docs）

不属于以上三类的 md 文件，sync 时移动到 /docs 目录（扁平结构，不建子目录）。
</File_Classification>

<Project_Structure>
```
当前工作目录/
├── docs/                        # 统一文档目录（doc-align 管理）
│   ├── DOC_INDEX.md             # 文档总索引（按时间排序）
│   └── {归档的 md 文件，扁平存放}
├── CLAUDE.md                    # AI 配置 — 保留原位
├── .cursorrules                 # AI 配置 — 保留原位
├── prompts/system.md            # 代码依赖 — 保留原位
├── README.md                    # 项目标准文档 — 保留原位
└── {其他散落的 .md}             # 可归档 → sync 时移到 docs/
```

**关键规则**：
- 归档 = 移动（mv），一文件一位置，不留副本
- /docs 内部扁平存放，不建子目录
- 只处理 `.md` 扩展名的文件，不处理 `.mdx`、`.txt` 等
- 排序按修改时间（mtime）升序，最早的在前
- 排除目录：node_modules、.git、vendor、dist、build、__pycache__、.next、.nuxt、.claude
</Project_Structure>

<Commands>

### `/doc-align init` — 初始化文档索引

**核心原则：只扫描索引，不移动任何文件。让用户先看清全貌。**

#### 步骤

1. **创建 /docs 目录** — 如果不存在则创建

2. **全面扫描** — 获取项目中所有 .md 文件及时间信息：
   ```bash
   find . -name "*.md" -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/vendor/*" -not -path "*/dist/*" -not -path "*/build/*" -not -path "*/__pycache__/*" -not -path "*/.next/*" -not -path "*/.nuxt/*" -not -path "*/.claude/*" -printf "%T@\t%w@\t%p\n" | sort -n
   ```

3. **代码依赖检测** — 对扫描到的文件，使用 Grep 检测是否被代码引用：
   - 对每个非 AI 配置、非项目标准文档的 md 文件，搜索其路径（不含 `./` 前缀）是否出现在代码文件中
   - 搜索范围：`*.py *.ts *.js *.go *.rs *.java *.rb *.php *.c *.cpp *.tsx *.jsx`
   - 匹配模式：文件路径片段（如 `prompts/system.md` 或 `system.md`）
   - 被引用的文件标记为"代码依赖文件"

4. **分类标记** — 将所有文件归入四类：
   - AI 配置文件 → 保留原位
   - 代码依赖文件 → 保留原位
   - 项目标准文档 → 保留原位
   - 可归档文档 → 建议归档到 /docs

5. **生成 DOC_INDEX.md** — 按修改时间升序排列，格式：

   ```markdown
   # DOC_INDEX — 文档总索引

   > 自动生成于 {日期时间}，由 doc-align 维护
   > 排序规则：按文件修改时间升序（最早的在前）

   ## 统计

   - 总文件数: {N}
   - AI 配置文件: {N}（保留原位）
   - 代码依赖文件: {N}（保留原位）
   - 项目标准文档: {N}（保留原位）
   - 可归档文档: {N}（建议归档到 /docs）

   ## 文档列表

   | # | 文件路径 | 分类 | 修改时间 | 创建时间 | 状态 |
   |---|---------|------|---------|---------|------|
   | 1 | {路径} | {分类} | {时间} | {时间} | {保留原位/待归档} |

   ## AI 配置文件

   | 文件路径 | 所属工具 | 修改时间 |
   |---------|---------|---------|
   | {路径} | {工具} | {时间} |

   ## 代码依赖文件

   | 文件路径 | 被引用于 | 修改时间 |
   |---------|---------|---------|
   | {路径} | {引用文件列表} | {时间} |

   ## 可归档文档

   | 文件路径 | 修改时间 | 归档目标 |
   |---------|---------|---------|
   | {路径} | {时间} | docs/{文件名} |
   ```

6. **输出初始化报告** — 告知用户分类结果，提示运行 `/doc-align sync` 开始归档

### `/doc-align` 或 `/doc-align sync` — 文档对齐归档

**前提：docs/DOC_INDEX.md 存在。如果不存在，提示用户先运行 `/doc-align init`。**

#### 步骤

1. **读取索引** — 读取 `docs/DOC_INDEX.md`

2. **重新扫描 + 代码依赖检测** — 同 init 的扫描流程，发现新增文件并检测代码依赖

3. **展示归档计划** — 列出所有"可归档文档"，让用户确认后再执行：
   - 展示每个文件的路径、修改时间
   - 同名冲突时，提示冲突详情并建议加数字后缀
   - 使用 `AskUserQuestion` 确认："确认归档以上 {N} 个文件到 /docs？"

4. **执行归档** — 对确认的文件执行 `mv`：
   - `mv {原路径} docs/{文件名}`
   - 同名冲突：用 `diff` 比较内容，相同则跳过，不同则加后缀 `_2`、`_3` 等

5. **更新索引** — 刷新 DOC_INDEX.md，更新归档状态

6. **输出对齐报告**：
   ```markdown
   ## 文档对齐完成

   - 归档了 {N} 个文件到 /docs
   - 跳过了 {N} 个文件（AI 配置/代码依赖/项目文档）
   - 发现 {N} 个新文件已添加到索引
   ```

### `/doc-align index` — 刷新索引

仅刷新 DOC_INDEX.md，不执行任何文件操作。步骤同 init 的扫描和分类，但：
- 如果 DOC_INDEX.md 已存在，保留用户手动添加的备注
- 重新执行代码依赖检测
- 更新时间信息和文件列表

### `/doc-align check` — 检查归档状态

对比 DOC_INDEX.md 和实际文件系统：
- 列出已索引但文件已删除的条目
- 列出未索引的新文件
- 列出归档路径与实际不符的文件
- 检查代码依赖关系是否发生变化

</Commands>

<File_Conflict_Handling>
当归档文件与 /docs 下已有文件同名时：
1. 使用 `diff` 比较文件内容
2. 内容相同：跳过，在索引中标记"已归档"
3. 内容不同：较新版本保留原名，较旧版本加数字后缀（如 `design_2.md`），在 DOC_INDEX.md 中标注差异
</File_Conflict_Handling>

<Tool_Usage>
- 使用 `Bash` 运行 `find`、`stat` 命令获取文件列表和时间信息
- 使用 `Bash` 运行 `mv` 命令执行归档操作
- 使用 `Grep` 检测 md 文件是否被代码引用
- 使用 `Read` 读取 DOC_INDEX.md 和 md 文件内容
- 使用 `Write` 创建/覆盖 DOC_INDEX.md
- 使用 `Edit` 更新 DOC_INDEX.md 的部分内容
- 使用 `Glob` 辅助查找 md 文件（`**/*.md`）
- 使用 `AskUserQuestion` 确认归档计划
</Tool_Usage>

<Examples>
<Good>
Init 时的扫描报告：
```
扫描完成，共发现 23 个 .md 文件：
- 4 个 AI 配置文件（CLAUDE.md, .cursorrules, AGENTS.md, .github/copilot-instructions.md）→ 保留原位
- 2 个代码依赖文件（prompts/system.md, config/template.md）→ 被 Python 代码引用，保留原位
- 2 个项目标准文档（README.md, CHANGELOG.md）→ 保留原位
- 15 个可归档文档（设计文档、会议记录、API 说明等）→ 建议归档到 /docs

DOC_INDEX.md 已生成到 docs/ 目录。
运行 /doc-align sync 开始归档。
```
Why good: 四类清晰，代码依赖自动检出，用户一眼知道哪些会动哪些不会。
</Good>

<Good>
Sync 时检测到代码依赖：
```
扫描发现 src/prompts/chat.md 被 src/llm/engine.py 引用（open("src/prompts/chat.md")）
→ 自动归为"代码依赖文件"，保留原位，不归档。
```
Why good: 避免移动被代码依赖的提示词文件，防止运行时报错。
</Good>

<Good>
Sync 时处理同名冲突：
```
docs/api.md 已存在，与 src/API_NOTES.md 内容不同：
- docs/api.md: 修改于 2024-03-10，328 行
- src/API_NOTES.md: 修改于 2024-03-15，412 行

→ 将较旧版本重命名为 docs/api_2.md，较新版本归档为 docs/api.md
```
Why good: 主动发现冲突，保留两个版本，避免数据丢失。
</Good>

<Bad>
不检测代码依赖直接移动：
```
（将 prompts/system.md 移动到 docs/system.md）
→ 项目运行时报 FileNotFoundError
```
Why bad: 提示词文件被代码依赖，移动后程序直接崩溃。
</Bad>

<Bad>
静默操作不确认：
```
（直接将所有可归档文件移动到 /docs）
```
Why bad: 用户不知道文件被移动到哪里，可能破坏工作流。
</Bad>
</Examples>

<Escalation_And_Stop_Conditions>
- 如果当前目录没有任何 .md 文件，告知用户"没有发现任何 Markdown 文件，无需初始化"
- 如果 /docs 目录不存在且 init 未执行，提示先运行 `/doc-align init`
- 如果扫描到的文件超过 100 个，提醒用户"文件较多，建议先 check 再 sync"
- AI 配置文件和代码依赖文件**永远不会被移动**，只做索引记录
- 用户可以随时手动编辑 DOC_INDEX.md 添加备注
- 如果用户取消归档确认，尊重用户选择，仅更新索引
- docs/DOC_INDEX.md 本身不会被归档或移动
</Escalation_And_Stop_Conditions>

<Final_Checklist>
- [ ] DOC_INDEX.md 按修改时间升序排列（旧在前）
- [ ] AI 配置文件仅索引，不移动
- [ ] 代码依赖文件通过 Grep 自动检测，仅索引，不移动
- [ ] 项目标准文档按严格名单判断，仅索引，不移动
- [ ] sync 归档前展示计划并要求用户确认
- [ ] 同名文件冲突时有 diff 比较和后缀处理
- [ ] 归档使用 mv（移动），不留副本
- [ ] /docs 内部扁平存放，不建子目录
- [ ] 扫描时排除 node_modules、.git、vendor、dist、build 等目录
- [ ] 每次操作后刷新 DOC_INDEX.md
</Final_Checklist>
