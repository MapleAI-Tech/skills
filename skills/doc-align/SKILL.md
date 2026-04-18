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

不属于以上三类的 md 文件，sync 时按命名风格重命名后移动到 /docs 目录（扁平结构，不建子目录）。
</File_Classification>

<Naming_Convention>
doc-align 归档时统一文件命名风格。在 `init` 时由用户选择，记录在 DOC_INDEX.md 头部，后续 `sync`、`consolidate` 均遵循此风格。

### 风格选项

**风格 A：英文 kebab-case**
- 格式：`{descriptive-english-name}.md`
- AI 根据文档内容生成英文 slug
- 示例：`user-auth-design.md`、`api-documentation.md`、`sprint15-meeting-notes.md`

**风格 B：中文描述式**
- 格式：`{主题}-{类型}.md`
- AI 根据文档内容识别主题和类型
- 示例：`用户认证-技术方案.md`、`接口文档-API.md`、`Sprint15-会议记录.md`

**风格 C：中英双语**
- 格式：`{中文主题}-{english-slug}.md`
- AI 同时生成中文主题词和英文 slug
- 示例：`用户认证-auth-design.md`、`接口文档-api-docs.md`、`会议记录-meeting-notes.md`

### 通用规则（所有风格通用）

- 文件名中不使用空格，用 `-` 连接
- 长度控制在 5-40 字符
- 不使用特殊字符（仅允许中文字符、英文字母、数字、连字符 `-`）
- 同名冲突时加数字后缀：`{name}-2.md`、`{name}-3.md`
- DOC_INDEX.md 不受命名风格约束
</Naming_Convention>

<Language_Alignment>
doc-align 归档时统一文档语言。在 `init` 时由用户选择，记录在 DOC_INDEX.md 头部，后续 `sync`、`consolidate` 均遵循此语言偏好。

### 语言选项

**选项 A：中文**
- 所有归档文档统一为中文，英文文档在归档时自动翻译
- 专业术语保留英文原文（如 API、JWT、OAuth、React）
- 示例：`# User Authentication Design` → `# 用户认证设计方案`

**选项 B：English**
- 所有归档文档统一为英文，中文文档在归档时自动翻译
- 示例：`# 用户认证设计方案` → `# User Authentication Design`

**选项 C：保持原样**
- 不翻译，保留文档原始语言
- 仅做文件归档和命名规范化

### 翻译规则

- 翻译保留原文结构（标题层级、列表、表格、代码块）
- 代码块内容不翻译，仅翻译代码块内的注释
- 专业术语在首次出现时可附原文，如「用户认证（User Authentication）」
- 翻译不增删内容，只做语言转换
- 文档已是指定语言时，跳过翻译
</Language_Alignment>

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

### Git 安全检查点（所有命令的前置步骤）

**文档操作涉及文件移动、合并、删除，必须先确保有 git 还原点。**

每次执行 `/doc-align` 的任何命令时，在执行实际操作之前，必须先完成此步骤：

#### 步骤

1. **检测 git 仓库** — 使用 `Bash` 运行 `git rev-parse --is-inside-work-tree`，确认当前在 git 仓库中

2. **检测工作区状态** — 运行 `git status --porcelain`：
   - 如果工作区干净（无输出）→ 提示用户："工作区干净，建议先提交一个安全检查点。是否执行 `git add -A && git commit -m 'doc-align: pre-op safety checkpoint'`？"
   - 如果工作区有未提交更改 → 提示用户："检测到未提交的更改。为防止文档操作中数据丢失，建议先提交当前状态：\n```\ngit add -A\ngit commit -m 'doc-align: pre-op safety checkpoint'\n```\n如需还原，可使用 `git log` 找到提交记录并 `git checkout` 恢复文件。"

3. **使用 `AskUserQuestion` 确认**：
   - 选项 A："已提交/确认跳过，继续执行" — 用户自行提交后选择此项，或明确选择跳过
   - 选项 B："帮我执行提交" — 自动执行上述 git commit 命令
   - 如果用户选择跳过，在后续操作报告中醒目提醒："⚠️ 未创建 git 安全检查点，如需还原文件请依赖手动备份"

4. **继续执行后续命令** — 完成安全检查点后，继续执行用户请求的实际命令

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

5. **选择命名风格** — 检查 `docs/DOC_INDEX.md` 是否已存在：

   **首次初始化（DOC_INDEX.md 不存在）**
   使用 `AskUserQuestion` 让用户选择归档文件的命名风格：
   - **风格 A：英文 kebab-case** — 如 `user-auth-design.md`、`api-documentation.md`
   - **风格 B：中文描述式** — 如 `用户认证-技术方案.md`、`会议记录-Sprint15.md`
   - **风格 C：中英双语** — 如 `用户认证-auth-design.md`、`接口文档-api-docs.md`

   **重复初始化（DOC_INDEX.md 已存在）**
   1. 读取 DOC_INDEX.md 头部，获取当前命名风格
   2. 使用 `AskUserQuestion` 提示用户："检测到已有索引，当前命名风格为「{X}」。请选择操作："
      - **仅刷新索引** — 保留当前命名风格，只更新文件列表和时间信息
      - **切换命名风格** — 更换风格，仅影响后续归档的文件，/docs 已有文件名不变
      - **切换 + 追溯重命名** — 更换风格，并对 /docs 已有文件追溯应用新命名
   3. 如果用户选择了"切换"相关选项，继续用 `AskUserQuestion` 选择新风格（A/B/C）
   4. 记录选择结果（是否需要追溯重命名、新命名风格），传递给后续步骤

6. **选择语言偏好** — 与步骤 5 相同的逻辑，检测 DOC_INDEX.md 是否已存在：

   **首次初始化（DOC_INDEX.md 不存在）**
   使用 `AskUserQuestion` 让用户选择文档语言偏好：
   - **中文** — 所有归档文档统一为中文，英文文档自动翻译
   - **English** — 所有归档文档统一为英文，中文文档自动翻译
   - **保持原样** — 不翻译，保留文档原始语言

   **重复初始化（DOC_INDEX.md 已存在）**
   1. 读取 DOC_INDEX.md 头部，获取当前语言偏好
   2. 使用 `AskUserQuestion` 提示用户："当前语言偏好为「{X}」。请选择操作："
      - **保持当前偏好** — 不变
      - **切换语言偏好** — 更换偏好，仅影响后续归档的文件
      - **切换 + 追溯翻译** — 更换偏好，并对 /docs 已有文件追溯翻译
   3. 如果用户选择了"切换"相关选项，继续用 `AskUserQuestion` 选择新偏好
   4. 记录选择结果（是否需要追溯翻译、新语言偏好），传递给后续步骤

7. **生成 DOC_INDEX.md** — 按修改时间升序排列，格式：

   ```markdown
   # DOC_INDEX — 文档总索引

   > 自动生成于 {日期时间}，由 doc-align 维护
   > 排序规则：按文件修改时间升序（最早的在前）
   > 命名风格：{A/B/C - 风格描述}
   > 语言偏好：{中文/English/保持原样}

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

8. **追溯处理（可选）** — 当步骤 5 或 6 用户选择了"追溯"相关选项时执行：
   1. 扫描 /docs 目录下所有 .md 文件（排除 DOC_INDEX.md）
   2. 需要追溯重命名时：对每个文件读取前 50 行内容，按新命名风格生成规范文件名
   3. 需要追溯翻译时：检测每个文件语言是否匹配新偏好，不匹配的标记为"待翻译"
   4. 展示处理计划：`旧文件名 → 新文件名`、`{文件名} 需翻译：英文 → 中文`
   5. 使用 `AskUserQuestion` 确认："确认对 /docs 中 {N} 个文件追溯处理？"
   6. 用户确认后执行：
      - 重命名：`mv docs/{旧文件名} docs/{新文件名}`
      - 翻译：读取文件内容 → 翻译 → 使用 `Write` 覆盖写入翻译后内容

9. **输出初始化报告** — 告知用户分类结果、已选命名风格和语言偏好、追溯处理结果（如有），提示运行 `/doc-align sync` 开始归档

### `/doc-align` 或 `/doc-align sync` — 文档对齐归档

**前提：docs/DOC_INDEX.md 存在。如果不存在，提示用户先运行 `/doc-align init`。**

#### 步骤

1. **读取索引** — 读取 `docs/DOC_INDEX.md`

2. **重新扫描 + 代码依赖检测** — 同 init 的扫描流程，发现新增文件并检测代码依赖

3. **生成归档命名 + 语言检测** — 对每个"可归档文档"，读取文件内容：
   - 读取文件前 50 行内容，识别文档主题和类型
   - 按照用户选择的命名风格（A/B/C）生成新文件名
   - 检测文档语言是否匹配 DOC_INDEX.md 中记录的语言偏好，不匹配则标记为"待翻译"
   - 同名冲突时，按通用规则加数字后缀

4. **展示归档计划** — 展示重命名 + 翻译 + 归档计划，让用户确认后再执行：
   - 展示每个文件的：原路径 → 新文件名（如 `design/old-name.md → docs/user-auth-design.md`）
   - 标注需要翻译的文件：`{文件名} 需翻译：英文 → 中文`
   - 使用 `AskUserQuestion` 确认："确认按以上计划归档 {N} 个文件（其中 {M} 个需翻译）？"

5. **执行归档** — 对确认的文件执行归档：
   - 不需翻译的文件：`mv {原路径} docs/{新文件名}`
   - 需要翻译的文件：读取原文 → 翻译内容 → `Write` 写入 `docs/{新文件名}` → `rm {原路径}`
   - 同名冲突：用 `diff` 比较内容，相同则跳过，不同则加后缀 `{name}-2.md`

6. **更新索引** — 刷新 DOC_INDEX.md，更新归档状态

7. **输出对齐报告**：
   ```markdown
   ## 文档对齐完成

   - 归档了 {N} 个文件到 /docs（已按「{命名风格}」重命名）
   - 翻译了 {N} 个文件（{语言方向}，如 英文→中文）
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

### `/doc-align consolidate` — 梳理、重命名与合并去重

**核心原则：分析文档内容，按主题重新组织，合并重复文档，保留最终版本并标注历史。**

**前提：docs/DOC_INDEX.md 存在。如果不存在，提示用户先运行 `/doc-align init`。**

#### 适用场景
- 文档命名混乱（如 `会议记录.md`、`meeting-notes.md`、`2024-03会议.md` 实为同一主题）
- 同一主题被拆分到多个文件中
- 文档内容过时但包含需要保留的决策信息
- 用户说"文档太乱了"、"帮我整理一下文档"、"合并重复文档"

#### 步骤

1. **读取索引 + 全面扫描** — 读取 DOC_INDEX.md，并重新扫描 docs/ 目录下所有 md 文件

2. **内容分析 + 分组** — 逐个 Read 文件内容，按主题分组：
   - 识别文件主题（设计文档、API 文档、会议记录、需求文档、技术方案等）
   - 将主题相同或高度重叠的文件归为同一组
   - 为每个分组生成规范文件名，遵循 DOC_INDEX.md 中记录的命名风格（A/B/C）

3. **展示整理计划** — 用 `AskUserQuestion` 向用户展示分组和重命名计划：
   ```markdown
   ## 文档整理计划

   ### 分组 1：用户认证方案
   - `docs/auth-design.md` → 合并到 `docs/用户认证-技术方案.md`
   - `docs/login-flow.md` → 合并到 `docs/用户认证-技术方案.md`
   - `docs/auth-v2-notes.md` → 合并到 `docs/用户认证-技术方案.md`

   ### 分组 2：API 接口文档
   - `docs/api.md` → 重命名为 `docs/API接口文档.md`
   - `docs/api-v2.md` → 合并到 `docs/API接口文档.md`

   ### 独立文档（仅重命名）
   - `docs/notes.md` → `docs/开发笔记.md`

   确认后将：创建新文件 → 删除旧文件
   ```

4. **用户确认** — 使用 `AskUserQuestion`：
   - "确认按上述计划整理文档？旧文件将被删除，内容合并到新文件中。"

5. **执行合并与重写** — 对每个分组：
   a. **合并内容** — 将同组文件内容按时间顺序（旧→新）合并到新文件
   b. **标注被取代的历史内容** — 对于被合并的旧内容，在新文件中添加清晰的标注：
      ```markdown
      ---
      <!--
      文档历史：
      - 本文件由以下文档合并而来：
        - auth-design.md（2024-03-01，原始认证设计）
        - login-flow.md（2024-03-10，登录流程补充）
        - auth-v2-notes.md（2024-03-15，V2 方案更新）
      -->

      # 用户认证 - 技术方案

      <!--
      ========== 历史方案（来源：auth-design.md，已合并，2024-03-01）==========
      以下为早期认证设计方案，已被 V2 方案取代。
      保留供参考，最新方案请见下方 "当前方案" 章节。
      -->
      ...旧方案内容...

      <!-- ========== 当前方案（来源：auth-v2-notes.md，2024-03-15）========== -->
      ...最新方案内容...
      ```
   c. **对于内容完全过时被抛弃的文档**，在新文件对应位置添加备注：
      ```markdown
      > **[已废弃]** 原方案采用 JWT 无状态认证（见 auth-v1.md），已于 2024-03-15 切换为 Session 方案。
      > 废弃原因：JWT 无法主动吊销 Token，存在安全隐患。
      > 原文档已删除，详细信息可通过 git 历史查看：`git log -- docs/auth-v1.md`
      ```
   d. **写入新文件** — 使用 `Write` 创建新文件

6. **删除旧文件** — 确认新文件写入成功后，使用 `Bash` 删除已合并的旧文件：
   ```bash
   rm docs/auth-design.md docs/login-flow.md docs/auth-v2-notes.md
   ```

7. **更新索引** — 刷新 DOC_INDEX.md：
   - 新增条目标记为"合并新建"
   - 删除条目标记为"已合并删除"
   - 在索引末尾添加变更日志：
     ```markdown
     ## 变更日志

     | 日期 | 操作 | 详情 |
     |------|------|------|
     | {日期} | consolidate | 合并 auth-design.md + login-flow.md + auth-v2-notes.md → 用户认证-技术方案.md |
     | {日期} | consolidate | api-v2.md 内容合并到 API接口文档.md，旧文件已删除 |
     | {日期} | consolidate | notes.md → 开发笔记.md（重命名） |
     ```

8. **输出整理报告**：
   ```markdown
   ## 文档整理完成

   - 创建了 {N} 个新文档（合并/重命名）
   - 删除了 {N} 个旧文档（内容已合并到新文档）
   - 保留历史备注 {N} 处（标注了被取代的旧方案）
   - 未处理的文件 {N} 个（AI 配置/代码依赖/项目标准文档，不在处理范围内）

   ### 已删除的旧文档（可通过 git 还原）
   - docs/auth-design.md → 内容已合并到 docs/用户认证-技术方案.md
   - docs/api-v2.md → 内容已合并到 docs/API接口文档.md

   如需还原：`git checkout HEAD -- docs/旧文件名.md`
   ```

#### 关键规则
- **必须先完成 Git 安全检查点**（见顶部前置步骤），consolidate 会删除文件，必须有还原手段
- **永不丢失信息**：即使旧文档内容被废弃，也必须在新文档中添加"已废弃"备注，说明旧方案内容、废弃原因、git 还原方法
- **AI 配置文件、代码依赖文件、项目标准文档不参与 consolidate**，只处理 /docs 目录下的可归档文档
- 合并时按时间顺序排列内容，最新的内容在前或单独成章节
- 每个被删除的文件都必须在变更日志中有记录

</Commands>

<File_Conflict_Handling>
当归档文件与 /docs 下已有文件同名时：
1. 使用 `diff` 比较文件内容
2. 内容相同：跳过，在索引中标记"已归档"
3. 内容不同：较新版本保留原名，较旧版本加数字后缀（如 `design-2.md`），在 DOC_INDEX.md 中标注差异
</File_Conflict_Handling>

<Tool_Usage>
- 使用 `Bash` 运行 `git` 命令检测仓库状态、创建安全检查点提交
- 使用 `Bash` 运行 `find`、`stat` 命令获取文件列表和时间信息
- 使用 `Bash` 运行 `mv` 命令执行归档操作
- 使用 `Bash` 运行 `rm` 命令删除已合并的旧文档（仅 consolidate 命令）
- 使用 `Grep` 检测 md 文件是否被代码引用
- 使用 `Read` 读取 DOC_INDEX.md 和 md 文件内容
- 使用 `Write` 创建/覆盖 DOC_INDEX.md 和合并后的新文档
- 使用 `Edit` 更新 DOC_INDEX.md 的部分内容
- 使用 `Glob` 辅助查找 md 文件（`**/*.md`）
- 使用 `AskUserQuestion` 确认归档计划、整理计划、git 安全检查点
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

<Good>
Consolidate 合并文档时标注历史：
```markdown
# 用户认证 - 技术方案

<!--
文档历史：
- 本文件由以下文档合并而来：
  - auth-design.md（2024-03-01，原始认证设计）
  - auth-v2-notes.md（2024-03-15，V2 方案更新）
-->

## 当前方案（V2）
...最新方案内容...

## 历史方案（V1，已废弃）
> **[已废弃]** 原方案采用 JWT 无状态认证，已于 2024-03-15 切换为 Session 方案。
> 废弃原因：JWT 无法主动吊销 Token，存在安全隐患。
> 原文档可通过 git 历史查看：`git log -- docs/auth-design.md`
```
Why good: 即使旧文档被删除，关键信息（方案内容、废弃原因、还原方法）仍可在新文档中找到。
</Good>

<Bad>
Consolidate 时直接删除不标注：
```
（合并内容后直接删除 auth-design.md，新文件中没有任何关于旧文件的说明）
```
Why bad: 用户日后无法追溯决策历史，无法知道为什么旧方案被废弃，也无法通过文档找到 git 还原方法。
</Bad>

<Bad>
跳过 git 安全检查直接 consolidate：
```
（用户请求 consolidate，直接开始合并删除操作，未确认 git 状态）
```
Why bad: consolidate 涉及文件删除，没有 git 检查点意味着误操作无法还原，风险极高。
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
- **consolidate 命令必须在 git 安全检查点完成之后才能执行**，如果用户跳过检查点，必须在操作报告中醒目标注风险
- **consolidate 只处理 /docs 目录下的文件**，不会触碰项目根目录或其他位置的文件
- 如果 consolidate 合并时无法确定主题分组，将文件列为"未分组"，请用户手动确认后再处理
- 如果 consolidate 执行失败（如写入新文件出错），**不要删除旧文件**，保持旧文件不动并报告错误
- 如果翻译失败（如文档内容为混合语言无法确定主语言），保留原文并标记"翻译失败"，不阻塞归档
</Escalation_And_Stop_Conditions>

<Final_Checklist>
- [ ] 所有命令执行前完成 Git 安全检查点（检测 git 状态、建议提交）
- [ ] DOC_INDEX.md 按修改时间升序排列（旧在前）
- [ ] AI 配置文件仅索引，不移动
- [ ] 代码依赖文件通过 Grep 自动检测，仅索引，不移动
- [ ] 项目标准文档按严格名单判断，仅索引，不移动
- [ ] init 时询问用户选择命名风格（A/B/C）和语言偏好，记录在 DOC_INDEX.md
- [ ] 重复 init 时检测已有 DOC_INDEX.md，提供刷新/切换/追溯三个选项（命名+语言）
- [ ] 追溯处理（重命名+翻译）前展示计划并要求用户确认
- [ ] sync 归档时根据命名风格生成规范文件名，不保留原始文件名
- [ ] sync 归档时检测语言偏好，不匹配的文档自动翻译
- [ ] sync 归档前展示重命名 + 翻译 + 归档计划并要求用户确认
- [ ] 同名文件冲突时有 diff 比较和后缀处理
- [ ] 归档使用 mv（移动），不留副本
- [ ] /docs 内部扁平存放，不建子目录
- [ ] 扫描时排除 node_modules、.git、vendor、dist、build 等目录
- [ ] 每次操作后刷新 DOC_INDEX.md
- [ ] consolidate 时每个被删除的旧文档都在新文档中有历史备注（文件名、时间、内容摘要）
- [ ] consolidate 时废弃内容有"已废弃"标注，包含废弃原因和 git 还原方法
- [ ] consolidate 执行失败时不删除旧文件
- [ ] consolidate 变更日志记录在 DOC_INDEX.md 中
</Final_Checklist>
