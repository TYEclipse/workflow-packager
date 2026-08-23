# 🧹 Workflow Packager — 工作流打包师

[![Hermes Agent](https://img.shields.io/badge/Hermes%20Agent-skill-blue)](https://hermes-agent.nousresearch.com)
[![Version](https://img.shields.io/badge/version-3.2.0-green)](./SKILL.md)
[![License](https://img.shields.io/badge/license-MIT-orange)](./LICENSE)
[![中文](https://img.shields.io/badge/lang-中文-red)]()

> A meta-skill for Hermes Agent that audits your conversation history, identifies repetitive manual workflows, and packages them into reusable assets (Skills, Subagents, or Automations).

从对话历史和工作记录中挖掘重复性手动工作，评估打包价值，封装为可复用的 AI 资产。

---

## ✨ What It Does

| Phase | What happens |
|-------|-------------|
| **0. Inventory Baseline** | Full enumeration of all skills, cron jobs, subagent definitions, and memory items |
| **1. Evidence Gathering** | Scans session history, memory, user profile, and existing skills |
| **2. Pattern Recognition** | Scores each workflow across 9 dimensions: frequency, time cost, error rate, context burden, consistency, prompt prefix reuse, role-setting patterns, format preferences, cross-session theme continuity |
| **3. Candidate Evaluation** | Filters candidates against hard thresholds (≥2 occurrences, structured I/O, clear value, not already covered) |
| **4. Form Selection** | Picks the right asset type: Skill, Subagent, Automation, Extend, or Skip |
| **5. Create (增)** | Generates the asset file with proper structure, triggers, and boundaries |
| **6. Delete (删)** | Auto-removes zombie workflows (≥30d unused, zero references) |
| **7. Merge (合)** | Auto-merges duplicate skill pairs with content cross-validation |
| **8. Patch (改)** | Auto-fixes deficiencies in existing skills |
| **9. Summary** | Produces a 6-table report: created, deleted, merged, patched, skipped, and needs-evidence items |

### 🆕 v3.1 Highlights

- **CRUD Execution Engine** — 增/删/合/改 full lifecycle: no longer just audit suggestions, now auto-executes create, delete, merge, and patch operations
- **Phase 0 Inventory Baseline** — complete skills + cron + subagent + memory enumeration before every run
- **Auto-delete zombies** — removes unused skills (≥30d inactive, zero references, not pinned)
- **Auto-merge duplicates** — detects and merges overlapping skill pairs with content cross-validation
- **Auto-patch deficiencies** — fixes version inconsistencies, missing triggers, broken references
- **TRACE Quality Framework** — 5-dimension quality check (Trust, Reliability, Adaptability, Convention, Effectiveness)
- **Safety gates** — dry-run-first for delete/merge, max 3 deletes / 2 merges / 5 patches per run, heartbeat cron permanent protection

---

## 📦 Installation

### For Hermes Agent users

```bash
# Install from GitHub
hermes skills install TYEclipse/workflow-packager
```

Or manually:

```bash
# Clone to your skills directory
mkdir -p ~/.hermes/skills/meta
git clone https://github.com/TYEclipse/workflow-packager.git \
  ~/.hermes/skills/meta/workflow-packager
```

### Trigger words

The skill auto-activates when you say any of:

> 打包工作流、整理技能、回顾记录、识别重复工作、创建 agent 技能、工作流自动化、重复任务优化、skill 生成、workflow packaging、进化技能

---

## 🚀 Usage

Just ask Hermes:

```
帮我回顾最近的工作，看看有没有可以打包成技能的重复工作
```

Or more targeted:

```
扫描我最近两周的会话，识别代码审查相关的工作流，帮我打包
```

The skill runs its multi-phase CRUD pipeline and produces a summary report with concrete actions.

---

## 📁 Repository Structure

```
workflow-packager/
├── README.md           # ← you are here
├── LICENSE             # MIT
├── SKILL.md            # The skill definition (loaded by Hermes Agent)
└── .gitignore
```

---

## 🤝 Contributing

This is an evolving meta-skill. If you discover patterns it misses, edge cases it mishandles, or improvement ideas:

1. Try running it and note what broke or was wrong
2. The skill itself has a **self-evolution** mechanism — after each run it reflects and updates
3. PRs are welcome for structural improvements, new pattern dimensions, or output format refinements

---

## 📄 License

MIT — see [LICENSE](./LICENSE) for details.

---

## 🔗 Related

- [Hermes Agent](https://hermes-agent.nousresearch.com) — The AI agent platform this skill runs on
- [Hermes Agent Docs](https://hermes-agent.nousresearch.com/docs) — Full documentation
