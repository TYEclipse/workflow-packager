---
name: workflow-packager
slug: workflow-packager
displayName: Workflow Packager — 工作流全生命周期CRUD执行引擎
version: 3.4.2
license: MIT
description: "工作流全生命周期CRUD执行引擎——回顾用户近期工作记录，识别重复性手动工作流程，评估打包价值，并为高置信度缺失事项增(创建)、删(清理)、合(归并)、改(修补)四类操作全自动落地。适用于编码、研究、写作、规划、沟通、运维、分析及个人事务管理等场景。特别擅长识别提示词前缀重复、角色设定模式、格式偏好和跨会话主题延续等隐性模式。v3.0核心升级：不再止于审计建议，新增Phase 0全量工作流清单基线、Phase 6自动删除僵尸工作流、Phase 7自动归并重复技能、Phase 8自动修补不足项。触发词：打包工作流、整理技能、增删合改、工作流审计、自动化清理、skill CRUD、进化技能。"
tags: [meta, workflow, automation, skills, audit, packaging, crud, lifecycle]
related_skills: [skill-creator, ecosystem-navigator]
---
# Workflow Packager — 工作流 CRUD 执行引擎 v3.4.2
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
18. **断轮续接（v3.3.19）**：本流水线可能因内存围栏/网关重启在任意一步死掉。Phase 0 第一步即查「未提交产物 + 门禁状态」——`git status --short` 有未追踪/未提交文件，或本仓测试/覆盖率门禁红，都说明上一轮**半途而废**：先把它收尾（缺测试补测试、缺文档补文档、缺版本号补 bump）或明确回滚，再进本轮 CRUD。否则每轮都会在同一处重新踩坑（实测：一轮写完新工具即崩，其后数轮都在重复「发现残留→修到一半→又崩」，门禁始终红着，发布全停）

## 生命周期所有权与退役阶梯（v3.3.33）
> 外部依据：Hermes Curator 官方文档（agent 件周巡 active→stale→archived，永不自动删除，pinned 与 cron 引用豁免）；TMLR 2026《Dynamic Agent Skills》八阶段生命周期（维护/治理＝provenance + rollback）；Ansible 模块生命周期（deprecated 保留 4 个发行周期后只留 tombstone）；Microsoft Graph 弃用策略（至少 24 个月通知）；Skill Graveyard（2026）：退役必须同时看使用遥测 + 语义冗余 + **不可替代性守卫**，只按「久未用」删会误伤稀有但唯一的技能。

1. **单一所有者（single owner）**：每个技能只有一个生命周期所有者。`curator list-unmanaged` 在册 ⇒ 本流水线负责；不在册 ⇒ curator 周巡负责（bundled/hub 归上游）。**「出范围」不是「没权限」，是「另一个自动所有者负责」**——报告必须写清所有者，禁止把托管件平铺成待办欠账。所有者停摆时，本流水线做看门狗：只触发所有者（`hermes curator run`），不代它删。
2. **五级退役阶梯**：`active → stale（14d 未用）→ archived（30d 未用，可 restore）→ 冷归档（技能树外 .trash，保留 ≥30d）→ purge（仅显式，永不自动）`。删除是最后一级，不是第一动作；任何一级都可回滚。
3. **证据分级**：年龄只是入场券。必须同时满足「届满双证 + 零功能引用 + 非 pinned + 无嵌套在册子件 + 可逆通道可用」。只按年龄退役＝把「季节性但关键」的技能误杀。
4. **不可替代性守卫**：退役前先问「它的独有内容有没有后继」。有后继 ⇒ 先 `merge_coverage` 核对并移植独有章节/附件，再退役；无后继但内容仍有价值 ⇒ 保留或冷归档，不硬删。禁止用「描述相似」代替内容覆盖核对。
5. **墓碑与替代指针**：退役件必须留下可查记录（归档清单 + SHA256 + 原路径 + 替代者/恢复命令），让未来的自己知道「它去哪了、怎么回来」；Ansible 的 tombstone、本仓 `.local/CHANGELOG.md` 都是同一原则。
6. **配置即代码 + 单写者**：cron 定义、技能版本、发布通道各只有一个权威源；改动后必须**读回全文**校验（`cronjob update` 是覆盖式），并只允许目标字段变化。定时任务必须有幂等、漏跑检测、重叠控制和完成信号——「到点会跑」不是可靠性设计。
7. **自动化工作流＝技能 + 定时任务**：本流水线对两者都有管理权；发现 cron 定义里的过时数字/能力描述/失效路径，直接修正并回读验证，不把「定义腐烂」留给用户。

## 执行流程
### Phase 0: 工作流全量清单（Workflow Inventory Baseline）
**每次执行第一步**，建立当前工作流的完整基线，确保后续增删合改决策有据可依：

1. **Skills 清单**：`skills_list()` + `search_files(pattern='SKILL.md', path='~/.hermes/skills/', target='files')` 输出全部技能名称+路径
2. **Cron 清单**：`cronjob(action='list')` 输出全部 cron 任务（ID + name + schedule + prompt 摘要）。**工具不可用时改读 `~/.hermes/cron/jobs.json`**——顶层为任务数组，`schedule` 是对象三态（`kind=cron`→`expr`、`kind=interval`→`minutes`、`kind=once`→`run_at`），`enabled=false` 即停用（已跑完的一次性任务常留此残迹，属待清理项而非僵尸）；心跳/守护类任务按任务名识别并永久保护
3. **Subagent 定义**：`search_files(pattern='subagent|delegate_task', path='~/.hermes/skills/', target='content')` 检测技能中的 subagent 定义
4. **Memory 项目**：`memory` 工具列出全部条目（可用的前提下），交叉对比 session_search 确认活跃度
5. **输出基线报告**：生成 Markdown 表格，列：`名称 | 类型(skill/cron/subagent) | 最后活跃 | 引用计数 | 状态`
   - 最后活跃（权威源）：优先取环境活动度表（如 `hermes curator usage`：use/view/patch/act + last_activity，覆盖面含 never）——比会话检索更可靠，因为技能使用不一定留下可检索的会话痕迹；会话侧用 session_search(query='<name>', sort='newest') 交叉验证
     **`--json` 优先（实测可用）**：`hermes curator usage --json` 一次返回 `name / provenance / pinned / state / archived_at / activity_count / use_count / view_count / patch_count / last_activity_at(ISO) / created_at`——活动度、pinned 名单、归属三合一，免解析文本表（`hermes curator status` **不支持** `--json`，误传会退回主 CLI 帮助，故 pinned 也用 usage --json 取）
     **名称↔目录口径核对（判龄前置，v3.3.10 新增）**：活动度表的 `name` 与磁盘目录名**并非总是一致**——技能被重命名、显示名含空格、同名占多个目录、符号链接技能不在表内；此时按 name 拼路径取 mtime 会**静默拿到 None**，判龄随即走进「无龄」歧路（届满双证判不了 → 要么误保留、要么误删）。跑本仓确定性工具（零 LLM/零网络，桥接键＝frontmatter 的 `name`，不用目录名反查）：`hermes curator usage --json > /tmp/usage.json && python3 tools/inventory_check.py --usage /tmp/usage.json`（`--json` 机读 / `--fail-on-mismatch` 当门禁：命中「有名无目录」或「同名多目录」即 exit 1）。本机实测：usage 条目数与磁盘 SKILL.md 数常有小幅不等、名≠目录名可达两位数、偶有符号链接技能无活动度条目——这三类一律**不进删除批**，只报告（具体条数以当轮 `inventory_check.py` 输出为准，正文不写死）。**别把历史计数当口径**：技能树每轮都在变（新增/改名/归档），绝对数字写进本文档必腐烂——一律以工具**实时输出**为准，只记异常类别不记总数
   - 引用计数：search_files(pattern='<name>', target='content') 跨 skills/memories/cron 统计；`.usage.json`、hub 索引缓存、历史报告回显均属**非功能引用**，统计时须排除。**审计文档自伤**：本仓 SKILL 文档本身会把候选技能名写成举例 token（v3.3.9 实证：候选名在本仓 SKILL.md 里出现 2 处），**核实计数前先剔除本仓自身文档与 `.local/CHANGELOG.md` 的命中**——否则「零引用」永远不成立，僵尸判定被自己的报告喂成假阴性
   - cron 侧活动：`grep -c '<name>' <cron 配置>` 命中数 = 0 即视为 cron 零引用
6. **托管域心跳（v3.3.33，只读）**：`hermes curator usage --json > /tmp/usage.json && hermes curator list-unmanaged > /tmp/unmanaged.txt && python3 tools/curator_health.py --usage /tmp/usage.json --unmanaged /tmp/unmanaged.txt`。退出码 0=健康、3=逾期、4=暂停、2=不可读。**逾期且未暂停 ⇒ 直接 `hermes curator run`（prune-only、可逆、跑前自动备份），再复跑一次本工具确认归零**；暂停 ⇒ 只报告，绝不自动恢复；不可读 ⇒ 报告「无法确认托管域心跳」，不猜健康。这一条把「托管域」从无主黑箱变成有主可核（具体件数一律以当轮工具输出为准，正文不写死）。
7. **僵尸标记**：满足以下**全部**条件的标记为 🧟 Zombie：
   - 最后活跃 > 30 天前
   - 引用计数 = 0（无其他技能/任务引用）
   - 非 pinned
   - 非 cron 心跳任务（心跳守护类任务一律保护，按任务名识别，其 job_id 不写入本文档）

### Phase 1: 证据收集（Evidence Gathering）
1. **工具可用性预检**：先检测 memory、session_search 等核心工具是否可用。若 memory 不可用，跳过「扫描记忆」步骤并标注；若 session_search 不可用，**不要直接判「无会话证据」**——按「工具降级操作指南」直读 state.db 的 `sessions`/`messages` 表（可等价还原用户主动会话列表与首条用户消息），拿到证据后照常走评估流程。
   **memory 不可用降级**：当 memory 工具不可用时，尝试直接读取文件系统上的 memory 文件作为替代——检查 `~/.hermes/memory/`、`MEMORY.md`、`USER.md`、`~/.hermes/profiles/<active>/memories/` 等路径。若能读到 raw 文件，提取关键用户偏好和项目事实用于 pattern 识别；若所有路径均不存在，仅在汇总报告中标注「⚠️ Memory 不可用，文件降级也失败——本次分析仅基于 session 记录」。
2. 读取用户提供的近期会话记录与任务摘要（默认回顾 30 天，不足则全部使用）
   **确定性入口（v3.3.28）**：窗口计数与首条 user 消息直接跑 `python3 tools/session_stats.py --json`（只读 state.db，零 LLM/零网络）——它一次给出 1d/2d/7d/30d 非 cron 计数、30 天来源分布，以及两条闸门的现成判定：**稀疏窗口**（1d < 2 ⇒ 走快速通道）与**自动化为主**（cron 占比 ≥ 80% ⇒ 报告开头须标注「证据以自动化输出为主，用户主动行为模式可能被低估」）；库不可读时**退出码 2 而非空统计**（「读不到」绝不伪装成「没有会话」）。省掉手写 SQL 的两个老坑（时间列口径、来源筛选）
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
   - **curator 归属复核（口径修正 v3.3.16 / 三分 v3.3.33）**：管辖权 ≠ 创建渠道。`hermes curator usage --json` 的 `provenance`（agent/bundled/hub）说的是**谁创建的**，**不代表 curator 管辖**；把两者混同会让删除批连续多轮为空（v3.3.9–v3.3.15 实证漏判）。权威口径＝ `hermes curator list-unmanaged`：**在该名单里 ⇒ curator 不可达 ⇒ 属本流水线删除范围**；不在名单里 ⇒ 按所有者三分：① `pinned`（用户/守卫固定）② `curator 周巡`（agent 件，自动 active→stale→archived，可 restore）③ `上游 bundled+hub`（随发行/由 hub 管理，curator 永不触碰 hub，bundled 需 `prune_builtins`）。**报告禁止把三者加总成「出范围 N」当欠账**——托管≠无主。范围圈定跑确定性工具（零 LLM/零网络）：`hermes curator usage --json > /tmp/usage.json && hermes curator list-unmanaged > /tmp/unmanaged.txt && python3 tools/zombie_sweep.py --usage /tmp/usage.json --unmanaged /tmp/unmanaged.txt`（`--json` 机读 / `--fail-on-eligible` 当提醒门禁）——它把「管辖 + 桥接 + 届满双证 + 功能引用计数 + 可重装源」一次判完，只把全过者列进 `ELIGIBLE`，无源者列 `COLD`
   - **pin 面口径复核（v3.3.20 新增）**：`pinned` 总数与「本流水线可达域内的 pinned」是**两个口径**——把前者拿来解释「删除闸门被架空」是连续多轮的错归因（实测：本机 `pinned ∩ unmanaged = 0`，即 pinned 全部落在 curator 管辖内——数量以当轮 `usage --json` 与 `zombie_sweep` 摘要行为准，正文不写死；删除批为空的真因是**管辖边界**而非 pin）。`tools/zombie_sweep.py` 已在摘要行拆开计数（`pinned N（其中属本流水线可达域 M）`），并在 `M=0 且 N>0` 时直接打印归因提示；汇总报告里一律引用该计数，不再凭印象叙述 pin 面
   - **嵌套技能目录禁令（v3.3.16 新增 / 扁平化 v3.3.33）**：候选目录内若还有**更深一层的 SKILL.md**（聚合包式布局：父件目录同时装着别的在册技能），一律**禁止整目录删除**——子件可能仍被引用，父件的「零引用」毫无意义，整目录搬走会连带抹掉在册技能。正确处理是**先拆包、后退役**：① `skill_view`/`find` 列出子件；② 把子件整体 `mv` 到同级分类目录（如 `skills/<category>/<child>/`），保持目录名=声明名；③ `skills_list` + `python3 tools/inventory_check.py --usage /tmp/usage.json` 复核子件仍在册、无同名冲突；④ 再对父件单独走冷归档/删除。顺序颠倒＝不可逆误删。
   - **基线自伤（v3.3.16 修正）**：撤销备份/归档副本**必须放在技能树之外**（如 `~/.hermes/.trash/` 或 `~/.hermes/backups/`）——放进仓内 `.local/` 会让副本里的 SKILL.md 被枚举成「技能」，磁盘计数虚增、口径核对假报警；`tools/inventory_check.py` 已把 `.local`/`.trash`/`node_modules` 列入 SKIP_DIRS 兜底
   - **届满双证**：活动度（act=0 且 last_activity = never 或 >30d）与磁盘 mtime（>30d）须双双达标；二者之一未达 → 留在观察队列，不进删除批（避免把「刚创建、尚未用到」的技能误判为僵尸）。**边界从严**：mtime 恰好落在 30d 的当日按「未届满」处理，次日复核（同日边界误删不可逆）
   - **可重装核验**：删除前确认该技能存在上游可恢复源（官方 optional 套件在位、hub 渠道可重装等）——无源可重装者**不硬删**，走冷归档。**核验四查（2026-09-22 修正：官方 optional 套件的就地路径**不是** `~/.hermes/optional-skills`，实测为 `<hermes-agent 检出目录>/optional-skills/<domain>/<leaf>`，本机逾百个（计数随上游版本变动，勿记死数，实查为准 —— 路径与规模一律实查不假设）**：① `hermes curator usage --json` 的 `provenance` 字段（bundled/hub/agent）② hub 索引缓存 `~/.hermes/skills/.hub/index-cache/hermes-index.json` 命中（条目含 `source=official` 且 `path=optional-skills/…` ⇒ 官方套件）③ `~/.hermes/skills/.skills_store_lock.json` 的 `zip_url/source/version` ④ 官方套件就地存在性（`ls -d <hermes-agent 检出目录>/optional-skills/*/*/<name>`）。②③ 是**子串匹配**，须人工复核命中条目（短名易被别的条目描述误命中，如 `github-issues` 命中别的 skill 正文）；四查皆无 ⇒ **不直接删**，降级 `COLD` 冷归档（技能树外 `.trash`，可逆，见下条）。**v3.3.30 起四查已内建进 `tools/zombie_sweep.py`（默认开启）**：hub 索引按**整名精确匹配**（不再靠人肉复核子串）、安装锁取键名、optional 套件扫叶子名，四查全无从删除批降级为 `COLD` 并计入 `无源冷归档` 计数（**v3.3.33 起不再封顶保留**）；四个面**全都读不到**时报「源不可核」→ `REPORT` 保留（fail-closed，绝不退化成「读不到＝无源＝可删/可归档」）。`--no-reinstall-check` 可关闸（仅供测试/对比），`--optional-dir` 可指非默认套件根
   - **冷归档执行（COLD，v3.3.33）**：无上游可重装源的合格件移入 `~/.hermes/.trash/<YYYY-MM-DD>_<name>/`，并写 `MANIFEST.json`：原名、原路径、退役理由、逐文件 SHA256、替代者（有则写）、恢复命令（`mv ~/.hermes/.trash/<...> <原路径>`）、保留截止日（≥30d）。移入后**回读**：`test -f <新路径>/SKILL.md` + `sha256sum -c` 全过才算完成；30d 内不得 purge，purge 只能显式做且先复核无引用。冷归档 ≠ `skill_manage delete`：不走技能注册表删除，保留完整目录以便原样搬回。**保留期届满复核走确定性工具**：`python3 tools/trash_purge.py`（默认只读；`--json` 机读 / `--fail-on-expired` 当提醒门禁 / `--purge --confirm-purge` 才真删，双闸缺一即拒）。它按**条目名里的入站日期**判龄（`mv` 会保留技能自身的旧 mtime，按 mtime 判龄会把刚进站的老内容算成到期——不可逆误删），两种历史命名（`YYYY-MM-DD_名` / `YYYYMMDD_HHMMSS_名`）都认，`MANIFEST.purge_after` 优先于「名字日期+保留天数」；解析不出的条目记 `UNPARSED`（判龄不可能）永不 purge；**同名技能仍在技能树 ⇒ `HOLD_IN_TREE` 不 purge**（疑似已恢复，副本只是残留）；回收站根不可读退出码 2（读不到不伪装成没有）。
   - **mtime 判龄纪律**：判龄用技能目录自身 mtime（`stat -c %y`），不要用移动/复制后产生的容器时间戳；**先做名称↔目录桥接**（`tools/inventory_check.py`）——禁止拿活动度表的 name 直接拼目录路径，重命名/含空格/同名多目录的技能会得到 age=None，那属「**判龄不可能**」而非「未届满」，两者都不得进删除批；符号链接技能目录须跟随链接取 mtime（`stat -L`），别读链接自身的元数据（v3.3.10 实证：本机 4 例名≠目录名 + 1 例符号链接技能无活动度条目）
   - **悬空引用复核（curator 归档的副作用，v3.3.9 新增）**：归档 curator 按阈值归档技能后，活跃技能里指向被归档技能的引用会**静默变悬空**（`related_skills` 元数据 + 正文反引号提及），不报错但会让后续会话去找一个已不在活跃目录的技能。跑本仓确定性工具（零 LLM/零网络）：`python3 tools/refs_audit.py --ignore workflow-packager`（`--json` 机读 / `--fail-on-found` 可当门禁 / `--show-suppressed` 附带列出已抑制项），拿到「归档技能 ← 引用方」清单后并入汇总报告；**假阳性先降噪再判**——反引号提及不总是指向技能（API 枚举值、JSON 字段名、LaTeX 宏包名、内置工具组名、举例 slug 都同形），确认非引用者登记进 `tools/refs_audit_allowlist.json`（键＝归档名×引用方 精确对、判据必填；被抑制命中不计入门禁、但表格与 `--json` 仍如实单列，读不到白名单时 fail-open 照常报全量）；**有明确活跃后继者**才落地改写引用（属 Phase 8 白名单项），**无后继者**一律只报告、交用户二选一（`hermes curator restore <name>` 复活 or 更新引用），不擅自改写用户意图
     - **嵌套 frontmatter 必检（v3.3.13 修）**：`related_skills` 常写在 `metadata:` → `hermes:` 之下（键前有缩进）——只锚定列零的解析会**静默漏掉全部嵌套写法**（实证：审计从 23 点误报成 8 点，15 条元数据悬空引用隐身）。核对时必须同时看到 `kind=related_skills` 与 `kind=body×N` 两类行；只有 body 行没有 related_skills 行 ⇒ 先怀疑解析器，别怀疑技能树
     - **执行权归心跳守护（v3.3.13）**：本机 `skill-guard`（心跳仓 240m 轮，零 LLM）已接管悬空引用的**裁决与执行**（强引用→自动 pin；归档件仍被引用→自动 restore；例外清单 `never_restore` 为政策裁决）——本流水线的 `refs_audit.py` 只做**只读报告**，不与其抢跑；恢复（`hermes curator restore`）属其自动链，本流水线不手工执行
     - **命中分级降噪（v3.3.32）**：`refs_audit.py` 数的是**全部引用点**（`related_skills` 元数据 + 正文提及），口径比心跳仓 `skill-guard` 的**强引用图谱**宽——两处数字不等是口径差，**不是漏修**。实测差三类：① 引用方仅在 `related_skills` 元数据里指向归档件、正文无功能引用 ⇒ `skill-guard` 记 `strong=0`，**永不裁决**（既非待办也非故障）；② 归档件属上游 bundled（`.bundled_manifest` 命中）或登记在 `never_restore` ⇒ 记 `nonfunc`，restore 会被设计拒绝（恢复即遮蔽上游版本）；③ 仅正文提及（举例、历史记录、同名英文词）⇒ 非功能引用。故汇总报告**禁止**把 refs_audit 总数当「待处理欠账」平铺复述，须按三类分别计数；只有「强引用 → 可恢复归档件」是真待办，且归 `skill-guard` 自动链。分级核对走确定性入口（只读，输出「悬空」与「非功能性」两张清单，各带 `strong=` 与 `refs_by`）：`python3 <心跳仓>/scripts/skill-guard.py scan`—— **别再用「总数没降」当异常信号**（八轮同值实证：数字不变恰恰因为三类都不可动作）
   - **归档/活跃口径（防基线漂移）**：curator 归档 = 技能目录整体移入 `<skills-root>/.archive/`，故 `usage --json` 计数与磁盘 SKILL.md 计数会**同步下降**（v3.3.9 实证：一日内 211→161 活跃、.archive 50 个）——两处数字对不上时先查 `.archive`，别误判成技能丢失
2. **干运行报告**：首次执行仅输出删除计划，格式：
   ```
   | 序号 | 技能名称 | 最后活跃 | 引用计数 | 删除理由 |
   |------|---------|---------|---------|---------|
   | 1 | xxx | 45d ago | 0 | 僵尸，原功能已被 ecosystem-navigator 覆盖 |
   ```
3. **执行删除**（确认后）：`skill_manage(action='delete', name='<name>', absorbed_into='')` 
   - 每次最多删除 3 个（安全闸门）
   - 删除前已在汇总报告记录完整 skill_view 内容供恢复
3b. **执行冷归档（COLD）**：对 `zombie_sweep` 列出的冷归档批，跑 `python3 tools/cold_archive.py --name '<名>' --reason '<理由>' [--successor '<后继>']`（每次 ≤3 件）。工具会拒绝同名多目录/嵌套包/目标覆盖，并在移动后逐文件 SHA256 回读；失败自动搬回。**先做不可替代性判断**：有后继先 `merge_coverage` 到 0 独有再归档；无后继则在理由里写清「原样保留、可搬回」。
4. **Cron 清理**：对僵尸 cron 任务，`cronjob(action='remove', job_id='<id>')`
   - 心跳 cron 任务（按任务名识别：心跳/守护类任务）永久保护，任何情况下不删除；**job_id 属内部编号，不写入本文档**
   - **CLI 通道（v3.3.13）**：`cronjob` 工具不可用时的等效通道＝`hermes cron remove <id>`（同一官方 CRUD 面，另有 `hermes cron list/doctor/runs`）。先 `hermes cron doctor` 看健康（仅检活跃任务）、`hermes cron runs <id>` 核最后一轮状态，**再把 job 定义 + runs 导出到本仓 `.local/removed-cron-jobs-<date>.json` 留恢复依据**（确定性入口：`python3 tools/export_cron_jobs.py .local/removed-cron-jobs-<date>.json <id> [id...]`——一次落盘「任务定义 + run 历史」，查不到的 id 显式打 MISSING 并告警而非静默跳过；存储不可读时退出码 2，不产出空留档），然后才 remove——`remove` 会连带丢掉 ran 历史，留档是可逆性的前提
   - **清理对象口径**：`enabled=false` 且 `run_at` 已过去的一次性任务＝**跑完的残迹**（非僵尸、非失败），属待清理项；仍在未来的停用任务＝用户主动暂停，**不动**
   - **定义腐烂修正（v3.3.33）**：cron prompt 里的测试条数、能力描述、环境结论都是可验证事实；发现过时即用 `hermes cron edit <id> --prompt '<完整新版>'` 修正（**整份覆盖**，禁止只传增量），改完从 `~/.hermes/cron/jobs.json` 读回全文，做「排除运行期字段」的新旧 diff——只允许 prompt 变化；报告写「改了什么/回读结果」。注意：`update`/`edit` 的 prompt 是覆盖式，截断不会报错。
5. **Subagent 定义清理**：`search_files(pattern='delegate_task|<name>', target='content')` 查找过时 subagent 定义，输出清理建议

### Phase 7: 合·归并（Merge）
**自动归并 Phase 3-4 筛选出的确认重复技能对：**

1. **确认重复对**：基于三轮检测结果（关键词聚类 + 触发词碰撞 + 功能意图），筛选置信度 ≥ 高 的对
   - **候选圈定走确定性工具（v3.3.19 新增）**：①② 两轮不再逐轮肉眼扫全树——先 `hermes curator usage --json > /tmp/usage.json`、`hermes curator list-unmanaged > /tmp/unmanaged.txt`，再跑 `python3 tools/duplicate_scan.py --usage /tmp/usage.json --unmanaged /tmp/unmanaged.txt`（`--json` 机读 / `--min-score` 调阈值 / `--top` 限量 / `--allowlist` 读本地白名单 / `--fail-on-candidates` 可当门禁）。打分＝描述词元 0.40 + 真标题 0.25 + 名称 0.20 + 标签 0.15，两侧**无任何共享词元**一律丢弃（短名同形不构成重复证据）；某成分两侧都缺数据则剔除并按剩余权重归一（「没有标签」不得被当 0 压分）。检测因此**可复跑**（同输入同输出），决议不再因每轮重扫而漂移；③ 功能意图对比仍由本流程判定
   - **多重复（≥3 个同域技能）处理**：先定唯一目标再拆对——目标取「最新且信息最全者」（看文件 mtime + 描述里的规则版本号，旧版数字已废止的一方作源）；源技能独有的附加文件（references 子目录等）必须随并入目标目录（否则丢内容），并入后核对目标是否覆盖源的全部场景；n 个同域技能 = n-1 对，受「每轮 ≤2 对」闸门限制，超出部分顺延次轮
   - **curator 管辖提示**：源技能若被 curator 管辖（**以 `hermes curator list-unmanaged` 为准，不看 usage 表 provenance**），curator 的 consolidate 通道（可能处于 off）与本流水线都可归并——报告里写明现状（如 `consolidate: off`），让用户选择用哪条通道，不默认抢跑
   - **覆盖核对必须走确定性工具（v3.3.17 新增）**：「描述重叠」≠「内容全被覆盖」，而源技能常比吸收者更长（实测：本地重复件比上游合并件多出「预提交验证流水线」「发布大文件坑位」等独有章节）。删源前跑 `python3 tools/merge_coverage.py --pair <源 SKILL.md>::<目标文档> --fail-on-missing`（`--json` 机读）——它逐章节比对并把源独有章节列成 MISSING（只算真标题：H2-H4，围栏代码块内的 `# 注释` 一律忽略）；**有 MISSING ⇒ 本轮禁止删源**，先把独有章节与独有附件文件移植进目标，再复核到 0 独有。**短标题不做模糊匹配**（`A Section` vs `B Section` 相似度约 0.88，模糊匹配会把无关章节判成覆盖）
   - **聚合包目标要按文件核对（v3.3.18 新增）**：目标是「薄 SKILL.md 索引 + 分文件（references 子目录下的 Markdown）」的聚合包时，**不能拿包内薄 SKILL.md 当比对目标**——它只有几十行路由表，逐章节核对会把源全部章节判成 MISSING（全是假阳性，反而把真独有内容淹掉）。正确做法是先判「源这份文档对应包内哪个分文件」（按主题/标题重合度），再逐文件跑核对；一个源技能可能对应多个分文件，逐个跑到 0 独有
   - **附件核对（v3.3.18 新增，与章节核对并列必跑）**：章节全绿 ≠ 内容搬全——源技能自带的附件（references 子目录下的独立 Markdown）可能根本没进目标包（实证：源附件在目标包不存在，标题核对 0 独有，直接删源即永久丢件）。删源前跑 `python3 tools/merge_coverage.py --assets <源技能目录>::<目标技能目录> --fail-on-missing`：逐个核对 references / templates / scripts / assets / docs 五类子目录下文件的存在性与字节一致性——**缺失 = 阻塞**（先随包移植），**内容不同 = 提示不阻塞**（上游可能已改进，交人复核，不擅自覆盖）。附件的独有内容常被正文以指针引用，移植后要顺手把包内路由表/支持资产清单补上该文件，否则文件在包内也无人知道
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
   - 更新 related_skills 引用（含 `tools/refs_audit.py` 报出的悬空引用——仅在活跃后继明确时改写；无后继者只报告不擅改；**白名单只降噪、不改判定**——`refs_audit_allowlist.json` 仅登记已确认非引用的同形命中，判据必填，真坏链一律不许登记）
   - 修正技能正文里已被证伪的事实数字/口径（同一技能内部自相矛盾，或与已声明的 SSOT 冲突时；须在会话中留下证据来源）
   - 修正 frontmatter 版本号与 .local/CHANGELOG.md 不一致
   - 修复已知错误命令/URL

### Phase 9: 汇总输出（Summary Output）
最后输出六项汇总，使用以下格式。**报告开头固定三行**（缺一即不合格）：
1. `可删 X / 冷归档 Y / 保留 Z / 托管 T / 只报告 R`
2. `托管域拆分：pinned a ｜ curator 周巡 b（stale c，到期归档 d）｜ 上游 bundled+hub e`——**禁止**把 T 平铺成「出范围 T 件待处理」。
3. `CURATOR: ok|overdue|paused|unknown（last_run=… age=…h；managed=… stale=…）`——来自 `tools/curator_health.py`，逾期时写明是否已触发 `hermes curator run` 及复检结果。

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
### 七、托管域健康（只读）
| 项 | 值 | 说明 |
|----|----|------|
| curator 心跳 | ok/overdue/paused/unknown | last_run / age / interval+grace |
| 托管域 | managed N（active/stale） | 含 pinned；stale 中到期归档 M |
| 所有者三分 | pinned a ｜ curator b ｜ 上游 c | 三者之和=托管总数，不是欠账 |
```

## 边界与限制
- 不创建推测性资产：不得基于"未来可能会做"的假设创建技能
- 不覆盖已有工具：如果用户已有现成工具，优先建议复用
- 不处理敏感数据：涉及密码、密钥、个人隐私的工作流直接标记为 Skip
- 不越权操作：每个资产只有一个生命周期所有者；托管件由 curator 周巡归档，本流水线只监督其心跳并在逾期时触发，不双写、不代删；创建 Automation 不得涉及修改生产系统等副作用
- 保持谦逊：对不确定的模式标记为 Need More Evidence，不强行打包
- Burst 模式降级：对短时间内集中爆发的需求保持警惕
- **高覆盖环境资产审计**：当已有技能 ≥50 且核心模式均被覆盖时，Phase 5 不强行创建资产——改为输出「资产审计报告」，列出 ≥30 天未使用的疑似僵尸技能和功能重叠的技能对，帮助用户清理而非堆积。**增强**：重叠检测分三轮执行——① 关键词聚类（name/description 中的中文+英文关键词提取，Jaccard 相似度 >0.5 标记）、② 触发词碰撞（取每个技能 description 的前 5 个触发词，计算重叠率）、③ 功能意图对比（人工判断聚类后的技能对是否确实功能重叠）——**①② 自 v3.3.19 起改跑确定性工具 `tools/duplicate_scan.py`**（打分可复跑，避免同一对重复技能跨轮被反复「重新发现」；③ 仍由本流程判定）。优先标记以下模式：latex-to-docx ↔ latex2word-pdf、html-mail-builder ↔ html-email-builder、speech-synthesis ↔ voiceover-studio 等已知高频重叠对。输出格式为 Markdown 表格，列：`技能 A | 技能 B | 重叠描述 | 建议（合并/归档/保留分工）`。
  **审计操作指引**：当建议「合并」时，在表格后附加实操步骤——用 `skill_manage(action='delete', absorbed_into='<target>')` 吸收旧技能，用 `skill_manage(action='patch')` 在目标技能的 description 中添加被吸收技能的场景覆盖声明。在执行任何删除之前，必须先 `skill_view` 两个技能确认内容不冲突。
  **增量审计实地验证**：增量审计模式下，不得信任上期报告中声明的「已解决」状态。每次增量审计必须对已确认重复对执行 `ls -d <path_a> <path_b>` 实地检查文件系统，以实际目录存在性为准报告状态（✅已清理 / 🟡仍存在），避免上期误报导致清理遗漏。
- **工具降级操作指南**：当 Hermes 原生工具（skill_view/skill_manage/skills_list/cronjob/memory）不可用时，使用通用文件系统操作作为降级方案——`find ~/.hermes/skills -name SKILL.md` 枚举技能、`read_file` 替代 skill_view、`mv <path> ~/.hermes/.trash/<timestamp>_<name>` 替代 skill_manage(action='delete')（移至回收站而非永久删除，可手动恢复；**回收站留存复核与显式 purge 有确定性工具**——`python3 tools/trash_purge.py`（默认只读，`--purge --confirm-purge` 才真删；年龄按条目名里的入站时刻算，**不是文件 mtime**（`mv` 会保留技能自身的旧 mtime，按 mtime 判龄会误删刚进站的老内容）））、`patch` + `write_file` 替代 skill_manage(action='patch')。**会话证据**用只读 URI 直连 SQLite 替代 session_search：`sqlite3.connect('file:~/.hermes/state.db?mode=ro', uri=True)` → 查 `sessions` 表按 `source!='cron'` 筛用户主动会话，再逐会话取 `messages` 中首条 `role='user'` 记录的前 200 字符用于模式识别；⚠️ 时间列 `started_at`/`last_activity_at` 是 **unix 浮点秒**，必须与 `strftime('%s','now')` 相减比较——改用 `datetime('now')` 做字符串比较会**静默返回 0 行**（实测踩坑，会误判成「无会话」）。**Cron 信息**从 `~/.hermes/cron/jobs.json` 读取（字段口径见 Phase 0 第 2 步）。降级操作前必须完整读取目标文件内容记录在汇总报告中以支持撤销。⚠️ 绝对禁止在降级方案中使用 `rm -rf`——该命令不可逆且绕过框架所有安全检查（pinned 保护、absorbed_into 追踪、跨 profile 引用检测），仅允许 `mv` 到回收站。
## 本仓版本与质量门禁
- **版本号唯一权威 = 仓根 `VERSION`**；SKILL.md frontmatter、SKILL.md H1、README 版本徽章三处必须与之一致。同步 `python3 tools/bump_version.py --sync`，升级 `python3 tools/bump_version.py patch|minor|major --note "说明"`
- **有代码就要有测试**：`bash run_tests.sh` = 测试全绿 + 覆盖率不低于基线（`.coverage-baseline.json`，持平放行）
- **提交门禁** `.githooks/pre-commit`：改 `tools/`、`tests/`、`.githooks/`、`run_tests.sh`、`.coveragerc`、`SKILL.md`、`README.md` 时，VERSION 必须变，且发布门禁通过、测试全绿、覆盖率不低于基线
- **发布只能走** `~/.hermes/scripts/wp-publish.sh`；`.githooks/pre-push` 拒绝直接 push
- 应急通道：`git commit --no-verify`（在 `.local/CHANGELOG.md` 记理由）

## 隐私合规门禁（发布前必检，最高优先级）
- **红线**：任何发布（GitHub/SkillHub/其他公开渠道）绝不允许出现：真实姓名、公司/部门信息、项目代号、住址、家人信息、个人日程与会话细节、内部编号、API/配置信息。拿不准的一律不发布（fail-closed）
- **当日修改技能体检**：每轮运行对**当日修改过的每个技能**执行三项检查：
  1. **隐私扫描**：grep 词表（真名/公司/项目代号/住址/家人/日程）命中 → 禁止发布、禁止提交，先修复
  2. **合规检查**：frontmatter 无 author 真名、无变更日志章节、版本号四处一致（`VERSION`/frontmatter/H1/README 徽章，`python3 tools/bump_version.py --show` 自查）；**README 亮点同步**（门禁⑤，v3.3.14 起强制）：README 必须含本版 `### 🆕 v<VERSION> Highlights`——徽章会自动跟 bump 走，手写亮点不会，缺了即拦（发布产物要自述本版变更）
  3. **通用性检查**：内容不绑定特定用户/公司场景；个人示例必须替换为通用表述
  - **一键跑法（零 LLM）**：`python3 tools/privacy_scan_today.py [--hours 24] [--fail-on-hit] [--json]` —— 自动枚举窗口内改动过的技能、逐件出「隐私 / 密钥 / frontmatter 事实」三列，并给 ⚠️/✅ 标记（**标记覆盖三类风险：隐私命中、密钥命中、author 真名**——v3.3.31 修正：早前只看后两类，词表已命中的行仍显示 ✅，人读口径与 `--fail-on-hit` 结论相反；命中行下附命中文件名，不回显词表 token 本身）；词表缺失或为空一律 fail-closed（退出码 2），绝不退化成「零命中＝干净」；`--fail-on-hit` 可当门禁（有命中即 exit 1）。**非源码一律不计**：备份/归档/虚拟环境子树不进枚举；缓存树（`__pycache__`/`.pytest_cache`/`.ruff_cache`/`.mypy_cache`/`.ipynb_checkpoints`）与工具链生成物（`coverage.json`/`CACHEDIR.TAG`/`.coverage*`）既不计文件数也不扫——副本与生成物会把体检刷成假命中，而 `.pyc` 里拼接式构造的假 token 会意外相邻成真形态（v3.3.26 实跑踩坑）；生成物排除按**文件名/类**判、不做路径白名单，形近的真源码照扫（v3.3.27/v3.3.29 分别按类排除字节码缓存与生成物）
   - ⚠️ **词表字面 token 也会自伤**：公开 payload 的正文字符串同样受扫描——写文档时出现词表中的字面片段（如目录名带斜杠的写法）会直接把门禁打成 ❌（本仓 v3.3.8 实证：一句新措辞触发 1 FAIL）。**处理＝改写措辞，不是改词表**；只有当某模式确实是通用词、误伤常态化时，才在 `.local/CHANGELOG.md` 记理由后收紧/放宽词表
  任一不通过 → 不发布、不提交
- **发布唯一通道**：`~/.hermes/scripts/wp-publish.sh`（白名单打包 + fail-closed 隐私扫描 + 发布前跑测试门禁 → GitHub 推送 → SkillHub 发布）。禁止直接执行 `skillhub publish` / `git push`
- **零变更不发布**：仅技能内容实质变更（版本号变化）时才发布；零变更日不发布、不推送、不 bump 版本
- **双渠道同步**：实质变更时 GitHub（TYEclipse/workflow-packager）与 SkillHub（skillId=85784）同时发布；发布管道在分支推送成功后**同日把版本 tag 推到远端**（`HEAD:refs/tags/v<version>`）——只推分支会让公开仓永远没有版本锚点。tag 推送失败不阻断发布，但必须显式告警，且**以回读 `git ls-remote --tags` 为准**，不以「管道说成功了」为准
- **发布状态判定口径（白名单打包的副作用）**：公开仓 `main` 是**白名单打包生成的全新单提交快照**（提交信息 `v<版本> (sanitized release)`），与本仓本地历史**无共同祖先**——故 `git rev-list --left-right --count origin/main...HEAD` 会报出「本地领先几十个提交」的假象，**不代表有未发布内容**；本地 main 也未设 upstream，`git status -sb` 只显示 `## main`。判断「已发布到哪一版」只看两处：远端 tag 列表 + 克隆回读文件里的版本号；**禁止**用提交数差推断，也禁止据此 force-push「追赶」
- **SkillHub 送达判定口径**（管道已内建，别用记忆代替）：返回成功 ≠ 公网立即可见——服务端异步出安全报告，公网版本列表滞后数分钟（实测首轮已 2xx，当时列表仍停在上一版、签名端点 404，数分钟后才出现）。判定「已送达」用两招：**同版本二次发布被服务端以「已存在」拒绝**（管道识别为幂等、计成功并打印复核提示）或**稍后回读版本列表**；**别把首次回读未见 / 签名端点 404 当发布失败**。识别幂等的判据是「已存在类标记 ＋ 本次版本号」双命中——只命中一个照旧报失败（防真失败被静默吞掉）
- **变更日志**：只写 `.local/CHANGELOG.md`（gitignored 本地档案，永不提交、永不发布、仅查档用）
