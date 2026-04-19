---
name: doc-align
description: 文档对齐 — 扫描、索引、整理项目中的所有 Markdown 文件，统一归档到 /docs 目录
---

<Purpose>
doc-align 是项目的"文档管家"。各种 AI 工具（Claude、Cursor、Copilot、Windsurf 等）会在项目中生成各自的 md 文件，散落各处难以维护。doc-align 通过扫描索引 + 移动归档，让所有 md 文件有迹可循、有处可归。
</Purpose>

<Use_When>
- 用户运行 `/doc-align help` — 查看使用说明
- 用户运行 `/doc-align init` — 首次初始化，扫描并索引所有 md 文件
- 用户运行 `/doc-align` 或 `/doc-align sync` — 将散落的 md 文件归档到 /docs 目录
- 用户运行 `/doc-align consolidate` — 梳理文档、重命名、合并去重，保留最终版本
- 用户运行 `/doc-align index` — 仅刷新 DOC_INDEX.md 索引，不做文件移动
- 用户运行 `/doc-align check` — 检查哪些 md 文件尚未归档
- 用户说"整理一下文档"、"对齐文档"、"md 文件太多了" — 自动匹配 sync 模式
- 用户说"梳理文档"、"合并文档"、"重命名文档"、"文档太乱了" — 自动匹配 consolidate 模式
</Use_When>

<Do_Not_Use_When>
- 用户想编辑某个 md 文件的内容 — 直接编辑，不需要 doc-align
- 用户想创建新的文档 — 直接创建到 /docs 即可，之后用 /doc-align sync 更新索引
- 用户在做代码相关工作 — doc-align 只处理 .md 文件
</Do_Not_Use_When>

<File_Classification>
doc-align 将 md 文件分为四类：

### 第一类：AI 配置文件（保留原位，仅索引）
CLAUDE.md、.claude/CLAUDE.md、.cursorrules、.cursor/rules/*、.github/copilot-instructions.md、.windsurfrules、AGENTS.md、ROADMAP.md，以及在 `.cursor/`、`.claude/`、`.github/` 目录内的 md 文件。

### 第二类：代码依赖文件（保留原位，仅索引）
被代码文件通过 import/read/open/fopen 等方式引用的 md 文件。使用 Grep 在代码文件中搜索 md 文件路径来检测。

### 第三类：项目标准文档（保留原位，仅索引）
严格名单：README.md、CHANGELOG.md、CONTRIBUTING.md、SECURITY.md、CODE_OF_CONDUCT.md、AUTHORS.md、LICENSE.md

### 第四类：可归档文档（移动到 /docs）
不属于以上三类的 md 文件，sync 时按命名风格重命名后移动到 /docs（扁平结构，不建子目录）。**已存在于 /docs 的文件也需检查是否符合命名风格和语言偏好。**
</File_Classification>

<Naming_Convention>
归档时统一文件命名风格，`init` 时由用户选择，记录在 DOC_INDEX.md 头部。

**风格 A：英文 kebab-case** — `user-auth-design.md`、`api-documentation.md`
**风格 B：中文描述式** — `{主题}-{类型}.md`，如 `用户认证-技术方案.md`、`会议记录-Sprint15.md`
**风格 C：中英双语** — `{中文主题}-{english-slug}.md`，如 `用户认证-auth-design.md`

通用规则：不使用空格（用 `-` 连接），长度 5-40 字符，仅允许中文/英文/数字/连字符，同名冲突加后缀 `{name}-2.md`。
</Naming_Convention>

<Language_Alignment>
归档时统一文档语言，`init` 时由用户选择，记录在 DOC_INDEX.md 头部。

**中文** — 英文文档自动翻译为中文，专业术语保留英文原文（API、JWT 等）
**English** — 中文文档自动翻译为英文
**保持原样** — 不翻译

翻译规则：保留原文结构，代码块不翻译（仅翻译注释），专业术语首次出现可附原文，不增删内容。
</Language_Alignment>

<Project_Structure>
```
当前工作目录/
├── docs/                        # 统一文档目录（doc-align 管理）
│   ├── DOC_INDEX.md             # 文档总索引（按时间排序）
│   └── {归档的 md 文件，扁平存放}
├── CLAUDE.md                    # AI 配置 — 保留原位
├── prompts/system.md            # 代码依赖 — 保留原位
├── README.md                    # 项目标准文档 — 保留原位
└── {其他散落的 .md}             # 可归档 → sync 时移到 docs/
```

关键规则：归档 = 移动（mv），不留副本；/docs 扁平存放，不建子目录；排序按 mtime 升序；排除 node_modules、.git、vendor、dist、build、__pycache__、.next、.nuxt、.claude。
</Project_Structure>

<Commands>

### Git 安全检查点（所有命令的前置步骤）

每次执行前必须确认 git 状态：
1. `git rev-parse --is-inside-work-tree` 确认在 git 仓库中
2. `git status --porcelain` 检测工作区状态
3. 使用 `AskUserQuestion` 让用户选择：**已提交/跳过** 或 **帮我提交**（执行 `git add -A && git commit -m 'doc-align: pre-op safety checkpoint'`）
4. 用户跳过时在报告中醒目提醒未创建检查点

### `/doc-align help` — 查看使用说明

直接输出以下帮助信息，不执行任何操作：

```
## doc-align — 文档管家

扫描、索引、归档项目中散落的 Markdown 文件，统一管理到 /docs 目录。

### 命令一览

| 命令 | 说明 |
|------|------|
| `/doc-align init` | 首次初始化：扫描所有 md 文件、选择命名风格和语言偏好、生成索引 |
| `/doc-align` 或 `/doc-align sync` | 文档归档：将散落的 md 文件按命名风格重命名后移入 /docs，按语言偏好自动翻译 |
| `/doc-align index` | 刷新索引：重新扫描并更新 DOC_INDEX.md，不移动文件 |
| `/doc-align check` | 检查状态：对比索引与文件系统，发现新增/缺失/异常文件 |
| `/doc-align consolidate` | 深度整理：按主题分组、合并重复文档、去重、保留历史标注 |
| `/doc-align help` | 查看本帮助信息 |

### 推荐工作流

1. /doc-align init        ← 首次使用，选好命名风格和语言偏好
2. /doc-align sync        ← 归档散落的 md 文件到 /docs
3. /doc-align consolidate ← 可选：深度整理，合并重复文档

### 核心概念

- **四类文件分级**：AI 配置/代码依赖/项目标准文档 → 保留原位；其他 → 归档到 /docs
- **命名风格**（init 时选择）：A.英文kebab-case / B.中文描述式 / C.中英双语
- **语言偏好**（init 时选择）：中文 / English / 保持原样
- **安全优先**：所有操作前建议创建 git 安全检查点，可随时通过 git 还原

### 重复 init

可安全重复运行，提供：仅刷新索引 / 切换风格偏好 / 切换 + 追溯处理已有文件

### 自然语言触发

- "整理一下文档"、"对齐文档" → sync
- "梳理文档"、"合并文档"、"文档太乱了" → consolidate
```

### `/doc-align init` — 初始化文档索引

**核心原则：只扫描索引，不移动散落文件。但 /docs 已有文件需规范命名和翻译。**

#### 步骤

1. **创建 /docs 目录** — 如果不存在则创建

2. **全面扫描** — 获取项目中所有 .md 文件及时间信息：
   ```bash
   find . -name "*.md" -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/vendor/*" -not -path "*/dist/*" -not -path "*/build/*" -not -path "*/__pycache__/*" -not -path "*/.next/*" -not -path "*/.nuxt/*" -not -path "*/.claude/*" -printf "%T@\t%w@\t%p\n" | sort -n
   ```

3. **代码依赖检测** — 使用 Grep 在代码文件（*.py *.ts *.js *.go *.rs *.java *.rb *.php *.c *.cpp *.tsx *.jsx）中搜索 md 文件路径，被引用的标记为"代码依赖文件"

4. **分类标记** — AI 配置/代码依赖/项目标准文档 → 保留原位；其他 → 可归档

5. **选择命名风格** — 使用 `AskUserQuestion`：
   - **首次 init**：直接选择 A/B/C（见 Naming_Convention）
   - **重复 init**：读取当前风格，提供"仅刷新 / 切换风格 / 切换+追溯重命名"三选项

6. **选择语言偏好** — 同样逻辑：
   - **首次 init**：直接选择 中文/English/保持原样（见 Language_Alignment）
   - **重复 init**：读取当前偏好，提供"保持 / 切换 / 切换+追溯翻译"三选项

7. **生成 DOC_INDEX.md** — 按修改时间升序排列，头部包含：
   ```markdown
   # DOC_INDEX — 文档总索引

   > 自动生成于 {日期时间}，由 doc-align 维护
   > 命名风格：{A/B/C - 风格描述}
   > 语言偏好：{中文/English/保持原样}
   ```
   后续包含：统计（总文件数、四类各多少）、文档列表表格（按分类分组）。预估超过 200 行时分次 Write（见 Split_Write_Rule）。

8. **规范 /docs 已有文件** — **无论首次还是重复 init 都执行**：
   1. 扫描 /docs 下所有 .md 文件（排除 DOC_INDEX.md，**含子目录中的文件**）
   2. 对每个文件读取前 50 行，按选定命名风格生成规范文件名
   3. 检测语言是否匹配偏好，不匹配标记为"待翻译"
   4. 子目录中的文件提取到 /docs 根目录（扁平化）
   5. 展示处理计划：`旧文件名 → 新文件名`、`需翻译：英文 → 中文`、`子目录 → 扁平化`
   6. 使用 `AskUserQuestion` 确认
   7. 确认后执行重命名（mv）、翻译（Read → 翻译 → Write）、扁平化（mv 子目录文件 → /docs 根）

9. **输出初始化报告** — 分类结果、已选设置、规范处理结果，提示运行 `/doc-align sync`

### `/doc-align sync` — 文档对齐归档

**前提：docs/DOC_INDEX.md 存在。不存在则提示先运行 `/doc-align init`。**

#### 步骤

1. **读取索引 + 重新扫描 + 代码依赖检测** — 发现新增文件

2. **归档散落文件** — 对每个 /docs 外的可归档文档：
   - 读取前 50 行，识别主题类型
   - 按命名风格生成新文件名
   - 检测语言是否匹配偏好，不匹配标记为"待翻译"

3. **检查 /docs 已有文件** — 扫描 /docs 下所有 .md 文件（排除 DOC_INDEX.md），对每个文件：
   - 读取前 50 行，按当前命名风格生成规范文件名
   - 对比当前文件名与规范文件名，不同则标记为"需重命名"
   - 检测语言是否匹配偏好，不匹配标记为"待翻译"
   - 子目录中的文件标记为"需扁平化"
   - 如果所有文件都符合规范（无重命名/翻译/扁平化需求），跳过本步骤，直接到步骤 5

4. **展示计划 + 确认** — 合并展示步骤 2 和步骤 3 的结果：
   - 归档计划：`原路径 → docs/{新文件名}`，标注翻译需求
   - 重命名计划：`docs/旧文件名 → docs/新文件名}`，标注翻译需求
   - 扁平化计划：`docs/子目录/文件 → docs/文件`
   - 使用 `AskUserQuestion` 确认

5. **执行**：
   - 归档（从外移入）：不需翻译 `mv {原路径} docs/{新文件名}`；需翻译 Read → 翻译 → Write → rm 原文件
   - 重命名（/docs 内）：`mv docs/{旧文件名} docs/{新文件名}`
   - 翻译（/docs 内）：Read → 翻译 → Write 覆盖
   - 扁平化：`mv docs/{子目录}/{文件} docs/{文件}`
   - 同名冲突：diff 比较，相同则跳过，不同则加后缀 `{name}-2.md`
   - 大文件分次写入（见 Split_Write_Rule）

6. **更新索引 + 输出报告**

### `/doc-align index` — 刷新索引

重新扫描分类，保留用户手动添加的备注，更新时间信息和文件列表。

### `/doc-align check` — 检查归档状态

对比 DOC_INDEX.md 和文件系统：列出已删除条目、未索引新文件、路径不符文件、依赖关系变化。

### `/doc-align consolidate` — 梳理、重命名与合并去重

**核心原则：分析内容按主题重组，合并重复文档，保留最终版本并标注历史。**

**前提：docs/DOC_INDEX.md 存在。不存在则提示先运行 `/doc-align init`。**

#### 步骤

1. **读取索引 + 全面扫描 docs/ 下所有 md 文件**

2. **内容分析 + 分组** — 逐个 Read，按主题分组，为每组生成规范文件名（遵循命名风格）

3. **展示整理计划 + 确认** — 展示分组合并/重命名方案

4. **执行合并**：
   - 同组文件按时间（旧→新）合并
   - 在新文件头部添加合并来源注释（文件名、日期、摘要）
   - 被废弃的内容标注 `[已废弃]` + 废弃原因 + git 还原方法
   - 大文件分次 Write（见 Split_Write_Rule）

5. **删除旧文件 → 更新索引（含变更日志）→ 输出报告**

#### 关键规则
- 必须先完成 Git 安全检查点
- 永不丢失信息：废弃内容必须在新文档中有"已废弃"备注
- 只处理 /docs 目录下的文件，不触碰其他位置
- 执行失败时不删除旧文件

</Commands>

<File_Conflict_Handling>
归档文件与 /docs 已有文件同名时：diff 比较 → 内容相同跳过 → 内容不同则较新保留原名、较旧加后缀 `{name}-2.md`。
</File_Conflict_Handling>

<Split_Write_Rule>
当写入内容预估超过 **200 行或 8000 字符**时，必须分次写入以防截断或写入失败：

1. **预估**：生成内容前，根据源文件大小和操作类型（翻译约 1:1、合并为多文件之和）预估输出长度
2. **分次策略**：
   - 先 `Write` 写入前半部分（文件头 + 前若干章节）
   - 再 `Edit` 追加剩余部分
   - consolidate 合并多个大文件时：每合并完一个来源就 Write 一次，逐步追加
3. **翻译长文件**：按章节拆分翻译，每段翻译完立即写入，避免累积超长输出
4. **DOC_INDEX.md**：文件数多时同样分次写入（先写头部+统计，再追加表格）
</Split_Write_Rule>

<Tool_Usage>
- `Bash`：git 操作、find/stat 扫描、mv 归档、rm 删除旧文档
- `Grep`：检测 md 文件是否被代码引用
- `Read`：读取 DOC_INDEX.md 和 md 文件内容
- `Write`：创建/覆盖文件（大文件分次写入，见 Split_Write_Rule）
- `Edit`：更新 DOC_INDEX.md 部分、追加内容
- `Glob`：辅助查找 md 文件（`**/*.md`）
- `AskUserQuestion`：确认归档/整理/重命名/翻译计划、git 安全检查点
</Tool_Usage>

<Examples>
<Good>
Init 扫描报告清晰：
```
扫描完成，共发现 23 个 .md 文件：
- 4 个 AI 配置文件 → 保留原位
- 2 个代码依赖文件（prompts/system.md 被 engine.py 引用）→ 保留原位
- 2 个项目标准文档 → 保留原位
- 15 个可归档文档 → 建议归档

已选命名风格：B（中文描述式），语言偏好：中文
/docs 中已有 8 个文件需要规范重命名，已展示计划。
```
Why good: 四类清晰，代码依赖自动检出，已有文件也纳入处理。
</Good>

<Good>
Sync 归档计划展示：
```
| 原路径 | 新文件名 | 翻译 |
|--------|---------|------|
| design/auth-flow.md | 用户认证-技术方案.md | 英→中 |
| notes/api-notes.md | API接口-文档.md | — |
| specs/requirements.md | 产品需求-文档.md | 英→中 |
```
Why good: 一表展示重命名+翻译+归档，用户一目了然。
</Good>

<Good>
Consolidate 合并标注历史：
```markdown
<!--
文档历史：
- auth-design.md（2024-03-01，原始设计）
- auth-v2-notes.md（2024-03-15，V2 更新）
-->
# 用户认证 - 技术方案
...当前方案...
> **[已废弃]** 原方案采用 JWT（见 auth-v1.md），已切换为 Session。废弃原因：JWT 无法主动吊销。还原：`git log -- docs/auth-v1.md`
```
Why good: 合并来源、废弃原因、还原方法都有记录。
</Good>

<Bad>
不检测代码依赖直接移动 prompts/system.md → 项目运行报错 FileNotFoundError。
Why bad: 代码依赖的提示词文件被移动后程序崩溃。
</Bad>

<Bad>
跳过 git 安全检查直接 consolidate，合并后删除旧文件，无法还原。
Why bad: consolidate 涉及删除，没有 git 检查点意味着误操作不可逆。
</Bad>
</Examples>

<Escalation_And_Stop_Conditions>
- 没有 .md 文件 → "没有发现任何 Markdown 文件，无需初始化"
- /docs 不存在且 init 未执行 → 提示先运行 `/doc-align init`
- 文件超过 100 个 → 提醒"文件较多，建议先 check 再 sync"
- AI 配置和代码依赖文件**永远不移动**
- 用户取消确认 → 尊重选择，仅更新索引
- DOC_INDEX.md 不被归档或移动
- consolidate 必须在 git 安全检查点之后执行
- consolidate 只处理 /docs 下的文件
- 合并无法分组 → 列为"未分组"请用户确认
- 执行失败 → 不删除旧文件，报告错误
- 翻译失败 → 保留原文标记"翻译失败"，不阻塞归档
</Escalation_And_Stop_Conditions>

<Final_Checklist>
- [ ] 所有命令执行前完成 Git 安全检查点
- [ ] DOC_INDEX.md 按修改时间升序排列
- [ ] AI 配置/代码依赖/项目标准文档仅索引不移动
- [ ] init 时选择命名风格和语言偏好，记录在 DOC_INDEX.md
- [ ] init 时规范 /docs 已有文件（命名+翻译+扁平化），首次和重复都执行
- [ ] sync 归档时强制按命名风格重命名 + 按语言偏好翻译
- [ ] 归档前展示计划并要求用户确认
- [ ] 同名冲突有 diff 比较和后缀处理
- [ ] 大文件分次写入（见 Split_Write_Rule）
- [ ] consolidate 合并文档保留历史备注和废弃标注
- [ ] consolidate 执行失败时不删除旧文件
- [ ] 每次操作后刷新 DOC_INDEX.md
</Final_Checklist>
