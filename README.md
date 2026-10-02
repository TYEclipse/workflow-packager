# 🧹 Workflow Packager — 工作流打包师

[![Hermes Agent](https://img.shields.io/badge/Hermes%20Agent-skill-blue)](https://hermes-agent.nousresearch.com)
[![Version](https://img.shields.io/badge/version-3.3.24-green)](./SKILL.md)
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

### 🆕 v3.3.24 Highlights

- **Troubleshooting notes rewritten to be environment-agnostic** — the fallback-operations guide illustrated the retention sweep with concrete machine-specific paths and script names. Published documentation must not depend on any single machine's layout, so the guidance now stays at the level of the mechanism: run a periodic sweep at off-peak hours with an age threshold, compute an entry's age from the timestamp encoded in its name, and never from file mtime (a moved directory keeps its old mtime, so an mtime-based age test would delete freshly added entries)

### 🆕 v3.3.23 Highlights

- **Idempotency check now locks the real marketplace payload (`tests/test_publish.py`)** — the earlier regression case asserted against a hand-written message, so a change in how the CLI renders the marketplace's "version already exists" rejection could keep the suite green while breaking the real matching path. The exact observed wording (including the CLI's own error prefix) is now asserted, and the classify-then-report flow is exercised end-to-end with that payload
- **Version matching tightened to token boundaries (`tools/wp_publish.py`)** — the exists-marker side was already strict, but the version side was a plain substring test: a version that is a *prefix* of another (3.3.2 inside 3.3.21) could be read as "this version is already live" and silently swallow a genuine publish failure, which is exactly the failure mode the check exists to prevent. The version must now appear as a standalone token, with assertions locking both directions

### 🆕 v3.3.22 Highlights

- **Idempotent re-publish recognised (`tools/wp_publish.py`)** — republishing a version that is already live is rejected by the marketplace with a non-zero exit, and that response used to be reported as a failure even though the artifact was already published. A non-zero exit is now classified as a failure only when it is *not* an "already exists" response: when both an exists-marker and the exact version string appear in the output, the step is reported as idempotent success, so no pointless human review is triggered
- **Two-part matching keeps real failures loud** — an exists-marker alone or the version string alone is never enough. Ordinary failures also quote the version being published, and unrelated errors can carry similar wording; both signals must appear together before the exit code is cleared
- **Visibility caveat carried into the pipeline** — a successful marketplace call is not the same as the version being publicly visible: security reports are generated asynchronously and the public version list lags by minutes, so an immediate read-back can still show the previous version. The publish step now prints how to confirm delivery (a same-version re-publish rejected as "already exists", or a later read-back of the list) and warns against treating a first read-back miss or a 404 on the signature endpoint as a failed publish
- **Regression coverage** — 2 assertions lock in the idempotent path, its two negative cases (marker without matching version, plain failure) and the visibility hint printed on both the success and idempotent paths

### 🆕 v3.3.21 Highlights

- **Rotted counts removed from the playbook** — the phase instructions quoted absolute machine-specific totals (activity entries vs on-disk skills, and the size of the upstream optional-skill tree) as if they were stable reference values. They are not: the skill tree grows, shrinks, renames and archives every round, so a figure frozen into the document silently turns into a wrong expectation, and a later run that trusts it misjudges whether the tree is consistent. Both sites now record only the **anomaly classes** worth guarding (name≠directory, skills absent from the activity table) and state plainly that absolute numbers must come from the tool's live output, never from this document
- **Same rule stated where it bites** — the reasoning is placed inline at both sites rather than in a general preface, because the failure mode is a run reading exactly one of those two lines and quoting its number in a report

### 🆕 v3.3.20 Highlights

- **Pin face counted separately from jurisdiction (`tools/zombie_sweep.py`)** — "a large pinned set is hollowing out the delete gate" had become the standing explanation for an empty delete batch, and it was wrong: being pinned and being curator-governed are independent facts, and on a real tree *every* pinned skill sat inside the curator's jurisdiction, so the binding constraint was jurisdiction, not pinning. The summary line now reports `pinned N (of which M reachable by this pipeline)`, and when N>0 with M=0 the tool states the attribution itself, so the reasoning cannot drift back to the convenient explanation
- **Regression coverage** — 2 assertions lock in the split counting and the conditional hint (the hint must be absent whenever a pinned skill really does fall inside the pipeline's reach)

### 🆕 v3.3.19 Highlights

- **Deterministic duplicate-candidate scoping (`tools/duplicate_scan.py`)** — rounds ① and ② of the overlap check (keyword clustering and trigger-word collision) used to be re-derived by eye on every run, so the same near-duplicate pair kept being "rediscovered" and then shelved again; the detection stage was not reproducible, so the decision could never converge. It is now one scored, replayable pass: description tokens 0.40, real headings 0.25, name 0.20, tags 0.15, with components that carry no data on either side dropped and the remaining weights renormalised, so a skill without tags is not scored down for the fields it does not have
- **Hard noise gate** — a pair sharing no tokens at all (description ∪ name ∪ tags) is discarded rather than surfacing as a coincidence hit, and nested `SKILL.md` files inside bundle-style skills never enter the pool, since their sections are not independent skills
- **Jurisdiction annotation** — supplied with the authoritative unmanaged list, each side is labelled curator or unmanaged and the pair is classified executable / report-only / mixed, keeping pairs that are governed on both sides in report-only mode exactly as the merge phase requires
- **Optional local allowlist** — verified non-duplicates can be registered with a mandatory rationale; suppressed pairs leave the candidate gate but stay visible in both the table and the JSON, and the file is never read implicitly, so a private local list cannot leak into a public artifact
- **Frontmatter parsing hardened** — block, inline and nested (metadata → hermes) list styles are all recognised; empty and inline lists used to re-enter the same line and hang the entire run, which is now locked down by a regression assertion
- **Regression coverage** — 27 assertions lock in tokenisation (Latin words plus CJK bigrams), weight renormalisation, the shared-token gate, nested-file exclusion, all three frontmatter styles, jurisdiction/scope mapping, allowlist semantics, rendering and the 0/1/2 exit-code contract; the tool itself sits at 100% statement coverage

### 🆕 v3.3.18 Highlights

- **Attachment gate for merges (`--assets`)** — a clean section-by-section result is not proof that everything moved: a source skill can carry a reference file the absorbing bundle never had (real case: the source shipped a Rust quality-gate reference file the bundle did not), and deleting the source would have destroyed it with no warning. The gate now inventories the references / templates / scripts / assets / docs subdirectories on both sides: **absent on the target blocks the deletion**, while **present-but-different is advisory only** (upstream may have improved it — never silently overwritten)
- **Bundle targets are compared file by file** — when the absorbing skill is a thin routing `SKILL.md` plus Markdown files under its references directory, comparing against that thin index flags every source section as unique and buries the genuinely unique ones. The documented procedure is to match the source document to its counterpart reference file first, then compare
- **Regression coverage** — 13 assertions lock in asset inventory (hidden/`__pycache__` skipped), missing vs differs semantics, assets-only invocation, JSON shape and the exit-code contract

### 🆕 v3.3.17 Highlights

- **Deterministic merge gate (`tools/merge_coverage.py`)** — "descriptions overlap" was being treated as "content is fully covered", but the duplicate skill is usually *longer* than the one absorbing it (a real case: the local duplicate carried a pre-commit verification pipeline and a large-file release pattern that the consolidated hub lacked). Before any source skill may be deleted, the tool compares it section by section against the absorbing document and lists every section the source alone owns. Any remaining `MISSING` entry blocks the deletion until that content is migrated
- **Heading extraction is code-fence aware** — reference documents are full of shell and rust comments starting with `#`; scraping every line that starts with a hash counted those comments as sections and inflated coverage. Fences (``` and ~~~, including unterminated ones) are tracked, and only real H2–H4 headings outside them are compared
- **Short headings never fuzzy-match** — normalized titles are compared by equality, then by containment (with a minimum length), then by similarity ratio. Short titles are excluded from the ratio pass because unrelated short sections score around 0.88 on similarity and would be silently reported as covered
- **Regression coverage** — 19 assertions lock in fence handling, numbering and punctuation-insensitive matching, the short-heading floor, duplicate-heading de-duplication, JSON output and the `--fail-on-missing` exit contract

### 🆕 v3.3.16 Highlights

- **Deterministic zombie scoping (`tools/zombie_sweep.py`)** — the deletion scope was previously a judgement call, and the activity table's `provenance` field (who *created* a skill) was mistaken for curator jurisdiction, which silently kept the delete batch empty for several releases. Jurisdiction now comes from the authoritative unmanaged list, and a single zero-LLM tool decides jurisdiction + name↔directory bridging + the two-certificate staleness rule + functional reference counting, emitting only `ELIGIBLE` items
- **Nested-bundle guard** — a candidate directory may itself contain further registered skills (bundle-style layout). Removing it wholesale would take live, still-referenced children with it, so such directories are reported for manual merging and can never be marked eligible
- **Baseline self-harm fixed** — undo/archival copies kept inside the skills tree were being enumerated as skills, inflating disk counts and raising phantom reconciliation warnings. Copy destinations must live outside the tree, and the reconciliation tool now skips those directories defensively
- **Cross-cutting regression coverage** — 19 new assertions lock in the scoping rules: the two-certificate rule and its boundary day, never-active semantics, functional-vs-non-functional references (word boundaries, self-references, historical report echoes), unreadable-mtime reporting, and scan tolerance for missing paths, broken links and oversized files

### 🆕 v3.3.15 Highlights

- **Release tags now reach the public repo** — the publish pipeline pushed only the branch, so the remote carried no version tags at all while the local checkout had many. A tag ref (`HEAD:refs/tags/v<version>`) is now pushed alongside the branch on the same remote, giving every published artifact a citable version anchor
- **Tag rejection degrades loudly, never silently** — if the remote refuses the tag (for example tag protection), the release still completes, but the warning is printed and the skipped tag is visible in the run output
- **Publish-path regression coverage** — two new assertions lock the behaviour in: a tag ref must be pushed and the leading `v` normalised, and a refused tag must not fail the release

### 🆕 v3.3.14 Highlights

- **Nested-frontmatter parsing fix** — `related_skills` is commonly nested under `metadata.hermes`; a column-zero anchor silently missed every nested form. On a real library that hid 15 metadata references and under-reported the dangling set from 23 points to 8. Parsing now honours indentation and ends a block list at the first sibling key
- **Release-note gate** — a fifth deterministic check fails the publish gate when the README carries no `### 🆕 v<VERSION> Highlights` section, so a published artifact always describes its own release
- **Cron-remnant cleanup channel** — the delete phase exports expired one-shot job definitions and their run history to a local archive before removing them through the official CLI, keeping the operation reversible
- **Homograph allowlist entries** — four same-shaped but non-referential hits are recorded with a mandatory rationale, keeping the dangling-reference audit focused on real broken links

### 🆕 v3.3.12 Highlights

- **False-positive suppression for the dangling-reference audit** — a deterministic allowlist (`tools/refs_audit_allowlist.json`) records same-shaped hits that are *not* skill references (API enum values, JSON field names, LaTeX package names, built-in tool-group names, illustrative slugs). Every entry carries a mandatory rationale
- **Suppressed ≠ silent** — suppressed hits are excluded from the `--fail-on-found` gate but stay visible: counted in the table, listed with rationale under `--show-suppressed`, and exported in `--json` as `suppressed_count` / `suppressed`
- **Fail-open by design** — a missing or broken allowlist file yields an empty table, so real dangling references are never swallowed; `--no-allowlist` shows the raw hit set

### 🆕 v3.3.11 Highlights

- **Inventory name↔directory reconciliation** — `tools/inventory_check.py` deterministically bridges activity-table entries to on-disk skill directories via the frontmatter `name`, because renames, space-containing display names, duplicate names and symlinked skills silently make age lookups return "no age"
- **Age-judgement safety** — an unresolvable age is treated as "cannot judge" (never as "not yet expired"): such entries are excluded from any delete batch and only reported
- **Symlinked skill directories followed** — enumeration follows directory symlinks and reads the real directory mtime
- **Reinstall-source verification corrected** — the official optional suite is now probed at its real on-disk checkout path instead of an assumed location, so "no upstream source" verdicts rest on an actual check

### 🆕 v3.3.9 Highlights

- **Dangling-reference audit** — `tools/refs_audit.py` deterministically lists active skills that still point at archived ones (metadata + prose mentions), a silent side effect of archival curators
- **Self-hit exclusion in reference counting** — the audit document's own example tokens no longer count as references, so "zero references" stays provable
- **Stricter patch whitelist** — documented facts and citations are only auto-fixed with an explicit evidence trail; ambiguous citations are reported, never rewritten

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
├── VERSION             # single source of truth for the version number
├── run_tests.sh        # regression suite + coverage ratchet gate
├── tools/              # deterministic helpers (privacy gate, publisher, ref audit)
├── tests/              # regression suite (pure asserts, no pytest dependency)
└── .gitignore
```

### 🛠 Bundled Tools

| Tool | Purpose |
|------|---------|
| `bash run_tests.sh` | regression suite + coverage ratchet (must be green before any commit) |
| `python3 tools/refs_audit.py` | dangling-reference audit: active skills still pointing at archived ones (`--show-suppressed` / `--no-allowlist`) |
| `tools/refs_audit_allowlist.json` | data file for the audit: same-shaped non-references (enum values, field names, package names) with a mandatory rationale, excluded from the gate but still reported |
| `python3 tools/inventory_check.py` | name↔directory reconciliation (renames, space-containing names, duplicates, symlinked skills) |
| `python3 tools/zombie_sweep.py` | deterministic zombie scoping for the delete phase: jurisdiction + staleness double-certificate + functional reference counting, eligible-only output, with the pin face counted separately (total vs reachable) |
| `python3 tools/duplicate_scan.py` | deterministic duplicate-candidate scoping for the merge phase: weighted scoring, shared-token gate, jurisdiction annotation, optional local allowlist |
| `python3 tools/wp_gate.py` | fail-closed privacy & compliance gate over the publish payload |
| `python3 tools/wp_publish.py` | the only publish path (GitHub + SkillHub); treats an "already exists" rejection as idempotent success and prints the delivery-confirmation guidance |
| `python3 tools/bump_version.py` | keeps VERSION, frontmatter, H1 and badge in sync |
| `python3 tools/coverage_gate.py` | coverage ratchet enforcement |

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
