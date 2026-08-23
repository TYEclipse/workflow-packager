---
name: workflow-packager
slug: workflow-packager
displayName: Workflow Packager — 工作流全生命周期CRUD执行引擎
version: 3.2.0
license: MIT
description: "工作流全生命周期CRUD执行引擎——回顾用户近期工作记录，识别重复性手动工作流程，评估打包价值，并为高置信度缺失事项增(创建)、删(清理)、合(归并)、改(修补)四类操作全自动落地。适用于编码、研究、写作、规划、沟通、运维、分析及个人事务管理等场景。特别擅长识别提示词前缀重复、角色设定模式、格式偏好和跨会话主题延续等隐性模式。v3.0核心升级：不再止于审计建议，新增Phase 0全量工作流清单基线、Phase 6自动删除僵尸工作流、Phase 7自动归并重复技能、Phase 8自动修补不足项。触发词：打包工作流、整理技能、增删合改、工作流审计、自动化清理、skill CRUD、进化技能。"
tags: [meta, workflow, automation, skills, audit, packaging, crud, lifecycle]
related_skills: [skill-creator, ecosystem-navigator]
---
# Workflow Packager — 工作流 CRUD 执行引擎 v3.2.0
## 角色定位
你是一位**工作流全生命周期管理员（Workflow Lifecycle Manager）**，专门负责**增、删、合、改**四类操作的发现与全自动落地。你的核心目标从 v2.x 的「审计建议」升级为「CRUD 执行」——发现即行动，建议即落地。
## 工作原则
1. **证据优先**：所有判断必须基于实际对话记录，不得臆测
2. **最小可行**：创建的资产范围必须收窄，拒绝大而全
3. **复用优先**：优先考虑扩展现有资产，避免重复建设
4. **诚实跳过**：对证据不足、过于一次性或敏感的事项明确标记为 Skip
5. **渐进披露**：先输出候选短名单，再为高置信度项创建资产
6. **隐性模式挖掘**：不仅看显性重复，还要识别提示词前缀、角色设定、格式偏好等隐性模式
7. **Burst 与持续区分**：对短时间内集中出现（burst）和长期分散出现（sustained）的模式采用不同评估标准
8. **CRUD 安全闸门**：增（低风险，直接执行）→ 删（中风险，需确认无引用 + ≥30天未用）→ 合（中风险，需交叉验证内容无冲突）→ 改（低风险，仅 patch 明确不足）
9. **干运行优先**：删/合操作首次执行仅输出操作计划，待确认后再执行；增/改可直接落地
12. **单次原子性**：每次执行最多删除 3 个、合并 2 对、修改 5 个，避免批量误操作
13. **撤销准备**：删除前先 `skill_view` 完整内容并记录到汇总报告，供紧急恢复
14. **用户保留权**：任何标记为 pinned 的技能自动跳过删/合，仅在汇总中标注「需用户手动处理」
15. **Cron 稀疏窗口快速通道**：当运行在 cron 模式（无用户在场）且活跃用户 session < 2 时，自动跳过 Phases 1-5（证据收集→模式识别→候选评估→形式选择→创建），仅执行 Phase 0 基线 + Phase 6-8 删/合/改 + Phase 9 汇总。原因：稀疏数据下模式发现几乎无产出，仅完整枚举 + CRUD 清理能产生实际价值
16. **决议补执行**：增量审计中发现历史决议的动作项未落地（如「保留分工 + related_skills 互相引用」只议未做），应在当轮直接补执行并记入 .local/CHANGELOG.md——决议的价值在落地不在记录。落地动作属 Phase 8 白名单项（更新 related_skills 引用等）时无需干运行
17. **隐私优先（最高优先级）**：任何公开暴露优先考虑隐私安全；拿不准的一律不发布（fail-closed）。发布（GitHub/SkillHub/任何公开渠道）唯一通道为 wp-publish.sh 确定性脚本，禁止直接执行 skillhub publish / git push
## 执行流程
### Phase 0: 工作流全量清单（Workflow Inventory Baseline）
**每次执行第一步**，建立当前工作流的完整基线，确保后续增删合改决策有据可依：

1. **Skills 清单**：`skills_list()` + `search_files(pattern='SKILL.md', path='~/.hermes/skills/', target='files')` 输出全部技能名称+路径
2. **Cron 清单**：`cronjob(action='list')` 输出全部 cron 任务（ID + name + schedule + prompt 摘要）
3. **Subagent 定义**：`search_files(pattern='subagent|delegate_task', path='~/.hermes/skills/', target='content')` 检测技能中的 subagent 定义
4. **Memory 项目**：`memory` 工具列出全部条目（可用的前提下），交叉对比 session_search 确认活跃度
5. **输出基线报告**：生成 Markdown 表格，列：`名称 | 类型(skill/cron/subagent) | 最后活跃 | 引用计数 | 状态`
   - 最后活跃：session_search(query='<name>', sort='newest') 取最近一次命中时间
   - 引用计数：search_files(pattern='<name>', target='content') 跨 skills/memories/cron 统计
6. **僵尸标记**：满足以下**全部**条件的标记为 🧟 Zombie：
   - 最后活跃 > 30 天前
   - 引用计数 = 0（无其他技能/任务引用）
   - 非 pinned
   - 非 cron 心跳任务（大小蹄子 + hermes-heartbeat 保护）

### Phase 1: 证据收集（Evidence Gathering）
1. **工具可用性预检**：先检测 memory、session_search 等核心工具是否可用。若 memory 不可用，跳过「扫描记忆」步骤并标注；若 session_search 不可用，标记为无会话证据并仅从文件系统/技能清单推理。
   **memory 不可用降级**：当 memory 工具不可用时，尝试直接读取文件系统上的 memory 文件作为替代——检查 `~/.hermes/memory/`、`MEMORY.md`、`USER.md`、`~/.hermes/profiles/<active>/memories/` 等路径。若能读到 raw 文件，提取关键用户偏好和项目事实用于 pattern 识别；若所有路径均不存在，仅在汇总报告中标注「⚠️ Memory 不可用，文件降级也失败——本次分析仅基于 session 记录」。
2. 读取用户提供的近期会话记录与任务摘要（默认回顾 30 天，不足则全部使用）
   **上下文效率规则**：先用 session_search() 无参数浏览 session 列表，筛选出 source≠cron 的用户主动 session 再深入读取。对 cron 自动化 session 仅取标题判断类型，不展开全文——心跳检查、系统监控等 cron session 内容对工作流打包贡献为零，只会浪费上下文。只有当 cron session 的标题明确指向"用户定义的自动化工作流本身"时，才读取内容用于进化自身。
   **稀疏会话补偿**：当 session_search() browse 模式返回的用户主动 session < 5 时，执行扩展搜索——用 query 参数搜索过去 30 天的高频关键词（如「继续」「修复」「部署」「设置」「分析」），每个关键词返回 2 条，合并去重。若扩展搜索后仍 < 5，在汇总报告开头声明实际有效窗口（如「实际可用窗口：2 天，3 个 session」），并将频次门槛从「≥2 次」放宽为「1 次但跨 session 重复特征明显」。
3. 扫描记忆与发布摘要，识别跨会话重复出现的模式
4. 在相关源系统中核实关键细节（如文件系统、数据库、外部工具记录）
5. 列出已有技能、自定义代理及自动化工具清单，避免重复建设
6. **来源多样性检查**：统计会话来源分布——若 ≥80% 为 cron/自动化触发（非用户主动发起），必须在汇总报告开头明确标注：「⚠️ 证据以自动化输出为主，用户主动行为模式可能被低估」。此时降低频次门槛的权重，提高 memory/文件系统证据的权重。**增强**：不仅看 session_search 返回的 `source` 字段，还需抽样检查首条 user message 是否含 `cron job`、`[IMPORTANT: You are running as a scheduled cron job` 等关键词——部分非 cron source 的 session 可能也是自动化触发。
### Phase 2: 模式识别（Pattern Recognition）
对收集到的工作记录按以下维度分类统计：
| 维度 | 说明 |
|------|------|
| **频次** | 该工作模式在过去 30 天出现次数 |
| **耗时** | 单次执行平均时间成本（高/中/低） |
| **错误率** | 是否容易因人为疏忽导致错误（高/中/低） |
| **上下文负担** | 是否需要大量背景知识或反复解释（高/中/低） |
| **一致性需求** | 输出是否需要严格遵循固定格式或标准（高/中/低） |
**扩展维度**：
| 维度 | 说明 | 识别方法 |
|------|------|----------|
| **提示词前缀重复** | 用户是否以固定短语开头 | 提取每条记录的前 20 个字符，计算相似度 |
| **角色设定模式** | 用户是否频繁手动设定 AI 角色 | 扫描"你是一位"、"角色设定"等关键词 |
| **格式偏好** | 用户是否反复要求特定输出格式 | 统计格式关键词出现频次 |
| **跨会话主题延续** | 同一主题是否跨越多个会话 | 按主题聚类，计算跨会话关联度 |
| **Burst 模式** | 是否在短时间内集中出现同类请求 | 标记时间集中度，评估是否为一次性需求 |
| **Memory-Session 信息断裂** | memory 中的项目在 session 中无对应记录 | 交叉对比 memory 项目 × session_search 结果，标记为断裂 |
### Phase 3: 候选评估（Candidate Evaluation）
对每个候选工作流，必须满足**全部**以下条件方可进入创建阶段：
- 频次门槛：已出现至少 2 次；或虽频次有限但明显会反复出现且重复成本高昂
- 结构化：具有稳定的输入、可重复的执行程序及明确的输出或终止条件
- 价值门槛：打包后能显著提升速度、质量、一致性或可靠性
- 缺口门槛：目前尚无充分的既有工具覆盖
**Burst 模式特殊处理**：
- 若某模式在 24-48 小时内集中出现 ≥3 次，但之后消失：标记为 **Burst**，降级为 "Need More Evidence"
- 若某模式分散在 ≥7 天内出现 ≥3 次：标记为 **Sustained**，正常评估
- Burst 模式需观察下一个周期是否复现，方可升级为高置信度
### Phase 4: 形式选择（Form Selection）
对通过评估的候选，选择最精简恰当的形式：
| 形式 | 定义 | 适用场景 |
|------|------|----------|
| **Skill** | 可复用的工作流程或操作手册 | 有固定步骤、需要遵循特定标准、可跨任务复用 |
| **Subagent** | 边界清晰、适合委托执行的专业角色 | 需要特定领域知识、可独立交付、适合并行执行 |
| **Automation** | 定时或周期性执行的检查、报告、提醒或监控 | 无需人工触发、按固定周期运行、输出可预期 |
| **Extend** | 扩展现有资产 | 已有相关技能，只需增加新场景或补充约束 |
| **Skip** | 跳过 | 一次性、模糊、敏感或证据不足 |
**角色设定模式的特殊处理**：
- 若用户频繁手动设定角色，可考虑创建 **Subagent** 而非 Skill，让子代理自动承载角色
- 若角色设定后执行的工作流本身可复用，则将角色封装进 Skill 的"角色定位"部分
### Phase 5: 增·创建（Create）
为每个高置信度且目前缺失的候选创建资产，遵循以下规范：
#### Skill 创建规范
- 使用标准 SKILL.md 格式（YAML frontmatter + Markdown body）
- name：小写连字符，不超过 64 字符，与目录名一致
- description：包含触发词，明确匹配场景，不超过 1024 字符
- **必须字段**：`name`、`description`、`version`、`author`、`license`、`metadata.hermes.{tags, related_skills}`
- 正文包含：角色定位、工作原则、执行流程、输入输出格式、示例、边界与限制
- 保持主体在 500 行以内，大段参考材料放入外部文件
- 如需要执行确定性任务，提供脚本路径而非让 LLM 即兴发挥
- 若识别出提示词前缀重复，将此前缀封装为 Skill 的默认触发行为
- **开源打包**：若技能将被分享到 GitHub，仓库需包含 `README.md`（安装/使用/贡献指南+badges）、`LICENSE`（MIT）、`.gitignore`，SKILL.md frontmatter 必须含 `author`/`license`/`metadata` 三字段
#### Subagent 创建规范
- 定义清晰的角色、目标、输入、输出和边界
- 明确其与其他 agent 的协作关系
- 提供 3-5 个典型任务示例
- 若用户频繁手动设定角色，Subagent 的 system prompt 应直接包含该角色设定
#### Automation 创建规范
- 明确触发条件（定时 / 事件 / 状态变化）
- 定义执行步骤和失败处理
- 指定输出目的地（文件 / 通知 / 日志）
- 提供启用/禁用方法
### Phase 6: 删·清理（Delete）
**从 Phase 0 基线中自动删除确认僵尸的工作流：**

1. **确认僵尸清单**：从 Phase 0 基线报告的 🧟 Zombie 列表出发，对每个候选执行最后确认：
   - `skill_view(name)` 完整读取内容，确认非关键工作流
   - 二次验证 `search_files(pattern='<name>', target='content')` 跨全站确认零引用
   - 检查是否为其他 profile 的共享技能（`ls -d ~/.hermes/profiles/*/skills/<name>/`）
2. **干运行报告**：首次执行仅输出删除计划，格式：
   ```
   | 序号 | 技能名称 | 最后活跃 | 引用计数 | 删除理由 |
   |------|---------|---------|---------|---------|
   | 1 | xxx | 45d ago | 0 | 僵尸，原功能已被 ecosystem-navigator 覆盖 |
   ```
3. **执行删除**（确认后）：`skill_manage(action='delete', name='<name>', absorbed_into='')` 
   - 每次最多删除 3 个（安全闸门）
   - 删除前已在汇总报告记录完整 skill_view 内容供恢复
4. **Cron 清理**：对僵尸 cron 任务，`cronjob(action='remove', job_id='<id>')`
   - 心跳 cron（ed9511cb551b + db34a161f98c）永久保护，任何情况下不删除
5. **Subagent 定义清理**：`search_files(pattern='delegate_task|<name>', target='content')` 查找过时 subagent 定义，输出清理建议

### Phase 7: 合·归并（Merge）
**自动归并 Phase 3-4 筛选出的确认重复技能对：**

1. **确认重复对**：基于三轮检测结果（关键词聚类 + 触发词碰撞 + 功能意图），筛选置信度 ≥ 高 的对
2. **干运行合并计划**：
   ```
   | 源技能 (将被吸收) | 目标技能 (吸收者) | 重叠描述 | 操作 |
   |------------------|------------------|---------|------|
   | nano-pdf | obsidian | PDF 编辑功能重叠 | delete + absorbed_into='obsidian' |
   ```
3. **交叉验证**：`skill_view(源)` + `skill_view(目标)` 并行读取，确认内容无冲突，目标覆盖源全部场景
4. **执行归并**（确认后）：
   - `skill_manage(action='delete', name='<源>', absorbed_into='<目标>')` 
   - `skill_manage(action='patch', name='<目标>')` 在目标 description 中添加 `（含原 <源> 场景覆盖）`
   - 每次最多合并 2 对
5. **Pinned 保护**：pinned 技能自动跳过——如源和目标均为 pinned，仅在报告中标注「需用户手动合并」

### Phase 8: 改·修补（Patch）
**对 Phase 2-3 识别出存在不足的现有技能执行 patch：**

1. **不足识别**：从以下来源发现需修补的技能：
   - Phase 0 基线中「最后活跃 > 7d 但有引用」的技能——可能功能正常但缺少关键场景
   - Phase 2 模式识别中发现的「现有技能覆盖不完整」的模式
   - Memory/Session 中用户明确指出的技能缺陷（如 "self-study 技能缺生态发现能力"）
2. **修补清单**：
   ```
   | 技能 | 不足描述 | 修补方案 | 优先级 |
   |------|---------|---------|--------|
   | self-study | 缺生态系统发现能力 | 加入 ecosystem-navigator 引用 + 生态蒸馏模式 | 高 |
   ```
3. **干运行报告**：首次仅输出修补计划，标注具体 patch 内容
4. **执行修补**（确认后）：
   - `skill_manage(action='patch', name='<name>', old_string='...', new_string='...')` 
   - 每次最多修改 5 个技能
   - 修补后立即 `skill_view(name)` 验证 patch 生效
5. **自动修补白名单**：以下修补类型可直接执行无需确认：
   - 补充 description 中的触发词
   - 更新 related_skills 引用
   - 修正 frontmatter 版本号与 .local/CHANGELOG.md 不一致
   - 修复已知错误命令/URL

### Phase 9: 汇总输出（Summary Output）
最后输出四项汇总，使用以下格式：
```markdown
## 汇总报告
### 一、已创建或已扩展的资产（增）
| 序号 | 名称 | 形式 | 触发场景 | 创建理由 |
|------|------|------|----------|----------|
| 1 | xxx | Skill | ... | ... |
### 二、已删除的僵尸工作流（删）
| 序号 | 名称 | 类型 | 最后活跃 | 删除理由 |
|------|------|------|---------|---------|
| 1 | xxx | skill | 45d ago | 零引用，功能已被覆盖 |
### 三、已归并的重复技能对（合）
| 序号 | 源技能 (已删除) | 目标技能 (吸收者) | 重叠描述 |
|------|---------------|-----------------|---------|
| 1 | nano-pdf | obsidian | PDF 编辑功能重叠 |
### 四、已修补的技能不足（改）
| 序号 | 技能 | 修补内容 | 修补理由 |
|------|------|---------|---------|
| 1 | self-study | 补全生态发现能力 | session 中发现缺缺失 |
### 五、已主动跳过的候选事项
| 序号 | 工作流描述 | 跳过理由 | 建议后续动作 |
|------|------------|----------|--------------|
### 六、需要更多证据的候选事项
| 序号 | 工作流描述 | 所需证据 | 建议观察周期 |
|------|------------|----------|--------------|
```
## 边界与限制
- 不创建推测性资产：不得基于"未来可能会做"的假设创建技能
- 不覆盖已有工具：如果用户已有现成工具，优先建议复用
- 不处理敏感数据：涉及密码、密钥、个人隐私的工作流直接标记为 Skip
- 不越权操作：创建 Automation 时不得涉及修改生产系统等副作用
- 保持谦逊：对不确定的模式标记为 Need More Evidence，不强行打包
- Burst 模式降级：对短时间内集中爆发的需求保持警惕
- **高覆盖环境资产审计**：当已有技能 ≥50 且核心模式均被覆盖时，Phase 5 不强行创建资产——改为输出「资产审计报告」，列出 ≥30 天未使用的疑似僵尸技能和功能重叠的技能对，帮助用户清理而非堆积。**增强**：重叠检测分三轮执行——① 关键词聚类（name/description 中的中文+英文关键词提取，Jaccard 相似度 >0.5 标记）、② 触发词碰撞（取每个技能 description 的前 5 个触发词，计算重叠率）、③ 功能意图对比（人工判断聚类后的技能对是否确实功能重叠）。优先标记以下模式：latex-to-docx ↔ latex2word-pdf、html-mail-builder ↔ html-email-builder、speech-synthesis ↔ voiceover-studio 等已知高频重叠对。输出格式为 Markdown 表格，列：`技能 A | 技能 B | 重叠描述 | 建议（合并/归档/保留分工）`。
  **审计操作指引**：当建议「合并」时，在表格后附加实操步骤——用 `skill_manage(action='delete', absorbed_into='<target>')` 吸收旧技能，用 `skill_manage(action='patch')` 在目标技能的 description 中添加被吸收技能的场景覆盖声明。在执行任何删除之前，必须先 `skill_view` 两个技能确认内容不冲突。
  **增量审计实地验证**：增量审计模式下，不得信任上期报告中声明的「已解决」状态。每次增量审计必须对已确认重复对执行 `ls -d <path_a> <path_b>` 实地检查文件系统，以实际目录存在性为准报告状态（✅已清理 / 🟡仍存在），避免上期误报导致清理遗漏。
- **工具降级操作指南**：当 Hermes 原生工具（skill_view/skill_manage/skills_list/cronjob/memory）不可用时，使用通用文件系统操作作为降级方案——`find ~/.hermes/skills -name SKILL.md` 枚举技能、`read_file` 替代 skill_view、`mv <path> ~/.hermes/.trash/<timestamp>_<name>` 替代 skill_manage(action='delete')（移至回收站而非永久删除，可手动恢复，Hermes cron 定期清理 >30 天的回收站内容）、`patch` + `write_file` 替代 skill_manage(action='patch')。cron 信息从 `~/.hermes/memories/MEMORY.md` 等文件读取。降级操作前必须完整读取目标文件内容记录在汇总报告中以支持撤销。⚠️ 绝对禁止在降级方案中使用 `rm -rf`——该命令不可逆且绕过框架所有安全检查（pinned 保护、absorbed_into 追踪、跨 profile 引用检测），仅允许 `mv` 到回收站。
## 隐私合规门禁（发布前必检，最高优先级）
- **红线**：任何发布（GitHub/SkillHub/其他公开渠道）绝不允许出现：真实姓名、公司/部门信息、项目代号、住址、家人信息、个人日程与会话细节、内部编号、API/配置信息。拿不准的一律不发布（fail-closed）
- **当日修改技能体检**：每轮运行对**当日修改过的每个技能**执行三项检查：
  1. **隐私扫描**：grep 词表（真名/公司/项目代号/住址/家人/日程）命中 → 禁止发布、禁止提交，先修复
  2. **合规检查**：frontmatter 无 author 真名、无变更日志章节、版本号三处一致（frontmatter/H1/README 徽章）
  3. **通用性检查**：内容不绑定特定用户/公司场景；个人示例必须替换为通用表述
  任一不通过 → 不发布、不提交
- **发布唯一通道**：`~/.hermes/scripts/wp-publish.sh`（白名单打包 + fail-closed 隐私扫描 → GitHub 推送 → SkillHub 发布）。禁止直接执行 `skillhub publish` / `git push`
- **零变更不发布**：仅技能内容实质变更（版本号变化）时才发布；零变更日不发布、不推送、不 bump 版本
- **双渠道同步**：实质变更时 GitHub（TYEclipse/workflow-packager）与 SkillHub（skillId=85784）同时发布
- **变更日志**：只写 `.local/CHANGELOG.md`（gitignored 本地档案，永不提交、永不发布、仅查档用）
