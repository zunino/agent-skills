---
name: owasp-eval
description: Assess AI features against the OWASP Top 10 for LLM Applications 2025 and the OWASP Top 10 for Agentic Applications 2026. Takes an AI feature inventory (from /analyze-ai-features) and produces a security report with severity ratings, evidence, and mitigation recommendations.
---

# /owasp-eval

## Purpose

Evaluate each AI feature in a repository against the relevant OWASP Top 10 list, producing a structured security assessment report with severity ratings, concrete evidence, and actionable mitigation recommendations.

This skill consumes the inventory produced by `/analyze-ai-features`. It does **not** re-scan the repository for AI features — that is the job of the first skill.

## Inputs

- **Inventory file** — path to the `ai-features.md` file produced by `/analyze-ai-features`. If not provided, default to `docs/ai-sec-analysis/ai-features.md` relative to the repository root.
- **Repository path** — the root of the repository being assessed. Used to verify findings against actual code. If not specified, use the current workspace root.
- **Output path** (optional) — defaults to `docs/ai-sec-analysis/owasp-report.md`, relative to the repository root.

## Prerequisites

An AI feature inventory file must exist. If it does not, instruct the user to run `/analyze-ai-features` first.

## Output

A single markdown file following the format in [Output Format](#output-format) below.

## Reference Documents

The OWASP Top 10 documents are bundled in `references/`:

- `references/owasp-top10-llm.md` — OWASP Top 10 for LLM Applications 2025 (items LLM01–LLM10)
- `references/owasp-top10-agentic.md` — OWASP Top 10 for Agentic Applications 2026 (items ASI01–ASI10)

**Critical: lazy-load these documents.** Do not read both files upfront. Instead:

1. Read the inventory file first.
2. Determine which classifications are present (`llm-direct`, `agentic`, `hybrid`).
3. Read only the relevant OWASP reference file(s):
   - If only `llm-direct` features exist → read `references/owasp-top10-llm.md` only.
   - If only `agentic` features exist → read `references/owasp-top10-agentic.md` only.
   - If `hybrid` features exist → read both.
   - If both `llm-direct` and `agentic` features exist → read both.

This keeps context focused and avoids loading unnecessary material.

## Workflow

### Step 1 — Read and parse the inventory

1. Read the inventory file.
2. Parse the feature list, extracting for each feature:
   - Feature name
   - File path(s)
   - Classification (`llm-direct`, `agentic`, `hybrid`)
   - Description
   - Agentic properties table (if present)
3. Note the summary counts (how many features per classification).

### Step 2 — Load relevant OWASP references

Based on the classifications present (see [Reference Documents](#reference-documents) above), read the appropriate OWASP reference file(s).

For each OWASP item in the reference, extract:
- Item ID (e.g., `LLM01`, `ASI06`)
- Title
- Description
- Common examples of vulnerability
- Prevention and mitigation strategies

### Step 3 — Assess each feature

For each feature in the inventory, assess it against every item in the relevant OWASP list. Use the feature's file path(s) to locate and examine the actual code.

#### Assessment procedure per OWASP item per feature:

1. **Read the OWASP item's description and examples** from the reference document.
2. **Examine the feature's code** — read the files listed in the inventory. Look for:
   - Patterns described in the OWASP item's "Common Examples"
   - Presence or absence of the mitigations listed in "Prevention and Mitigation Strategies"
   - Configuration, dependencies, and data flows relevant to the item
3. **Determine status:**

| Status | Meaning |
|---|---|
| `mitigated` | The code implements appropriate controls for this item. No significant vulnerability found. |
| `partially-mitigated` | Some controls exist but gaps remain. Describe what is mitigated and what is not. |
| `vulnerable` | No meaningful controls exist for this item. The feature is exposed to the risk. |
| `n/a` | The item does not apply to this feature (e.g., LLM04 Data Poisoning for a feature using only pre-trained models via API with no fine-tuning). Explain why it doesn't apply. |

4. **Collect evidence** — for non-`n/a` items, cite:
   - File path and line numbers
   - Code patterns or configuration that demonstrate the finding
   - Specific control present or missing

5. **Write mitigation recommendations** — for `partially-mitigated` and `vulnerable` items:
   - Reference the relevant prevention strategies from the OWASP document
   - Adapt them to the specific codebase context
   - Be concrete: name files, functions, and configurations that should change
   - Estimate effort: `low` / `medium` / `high`

### Step 4 — Handle hybrid features

Hybrid features (primarily LLM-direct with bounded tool use) must be assessed against **both** OWASP lists:

- Assess against all LLM01–LLM10 items (as with any LLM-direct feature)
- Assess against all ASI01–ASI10 items, but for items that clearly don't apply (e.g., ASI07 Insecure Inter-Agent Communication when there's no multi-agent system), mark as `n/a` with brief justification

The tool-use aspect of hybrid features is particularly relevant to:
- LLM01 (indirect prompt injection via tool output)
- LLM06 (excessive agency — tool scope)
- ASI02 (tool misuse — even bounded tool use has misuse potential)

### Step 5 — Assess shared/cross-cutting concerns

Some OWASP items apply at the application level rather than per-feature. After assessing individual features, evaluate cross-cutting items:

- **LLM03 Supply Chain:** Assess the full AI dependency tree from the inventory's "AI Dependencies" section. Check for version pinning, SBOM, integrity verification.
- **LLM10 Unbounded Consumption:** Assess rate limiting, timeouts, input size limits, and quotas across **all** AI features collectively. A single missing rate limiter on one endpoint affects the whole application.
- **ASI04 Agentic Supply Chain:** Assess agent framework dependencies specifically.

Group these into a "Cross-Cutting Assessment" section in the report.

### Step 6 — Produce the report

Write the output using the format below.

## Output Format

```markdown
# OWASP Security Assessment Report — [repository name]

**Date:** [date]
**Repository:** [path or URL]
**Inventory:** [path to inventory file used]
**References:** OWASP Top 10 for LLM Applications 2025 · OWASP Top 10 for Agentic Applications 2026

---

## 1. Introduction

[Brief description of the project and the scope of this assessment. Mention which OWASP lists were applied and why, based on the feature classifications in the inventory.]

---

## 2. Feature Inventory Summary

- Total AI features: [N]
- LLM-direct: [N] (assessed against OWASP LLM Top 10 2025)
- Agentic: [N] (assessed against OWASP Agentic Top 10 2026)
- Hybrid: [N] (assessed against both lists)

[Reference the full inventory file for the detailed feature list.]

---

## 3. LLM-Direct Features Assessment

[For each llm-direct feature, produce a subsection.]

### [Feature name]

**File:** [path]
**Model:** [model]
**Framework:** [framework]

| OWASP Item | Status | Summary |
|---|---|---|
| LLM01: Prompt Injection | [status] | [1-sentence summary] |
| LLM02: Sensitive Information Disclosure | [status] | [1-sentence summary] |
| LLM03: Supply Chain | [status] | [1-sentence summary] |
| LLM04: Data and Model Poisoning | [status] | [1-sentence summary] |
| LLM05: Improper Output Handling | [status] | [1-sentence summary] |
| LLM06: Excessive Agency | [status] | [1-sentence summary] |
| LLM07: System Prompt Leakage | [status] | [1-sentence summary] |
| LLM08: Vector and Embedding Weaknesses | [status] | [1-sentence summary] |
| LLM09: Misinformation | [status] | [1-sentence summary] |
| LLM10: Unbounded Consumption | [status] | [1-sentence summary] |

#### Findings

[For each item that is not `mitigated` or `n/a`, provide detailed findings:]

##### [OWASP Item ID] — [Title] — [Status]

**Vulnerabilities:**
1. [Description with file:line evidence]

**Mitigations:**
1. [Concrete recommendation adapted to this codebase] — Effort: [low/medium/high]

---

## 4. Agentic Features Assessment

[For each agentic feature, produce a subsection with the same structure but using ASI01–ASI10.]

### [Feature name]

**File(s):** [path(s)]
**Framework:** [framework]
**Model:** [model]
**Agentic properties:** [brief summary from inventory]

| OWASP Item | Status | Summary |
|---|---|---|
| ASI01: Agent Goal Hijack | [status] | [1-sentence summary] |
| ASI02: Tool Misuse and Exploitation | [status] | [1-sentence summary] |
| ASI03: Identity and Privilege Abuse | [status] | [1-sentence summary] |
| ASI04: Agentic Supply Chain Vulnerabilities | [status] | [1-sentence summary] |
| ASI05: Unexpected Code Execution (RCE) | [status] | [1-sentence summary] |
| ASI06: Memory & Context Poisoning | [status] | [1-sentence summary] |
| ASI07: Insecure Inter-Agent Communication | [status] | [1-sentence summary] |
| ASI08: Cascading Failures | [status] | [1-sentence summary] |
| ASI09: Human-Agent Trust Exploitation | [status] | [1-sentence summary] |
| ASI10: Rogue Agents | [status] | [1-sentence summary] |

#### Findings

[Same detailed finding format as LLM-direct section.]

---

## 5. Hybrid Features Assessment

[For each hybrid feature, assess against both lists. Use two status tables — one for LLM01–LLM10, one for ASI01–ASI10.]

### [Feature name]

**File(s):** [path(s)]
**Tool use:** [description of bounded tool call]

**LLM Top 10 Assessment:**

| OWASP Item | Status | Summary |
|---|---|---|
| LLM01–LLM10 | ... | ... |

**Agentic Top 10 Assessment:**

| OWASP Item | Status | Summary |
|---|---|---|
| ASI01–ASI10 | ... | ... |

#### Findings

[Detailed findings for both lists.]

---

## 6. Cross-Cutting Assessment

[Items that span multiple features:]

### Supply Chain (LLM03 / ASI04)

[Assessment of the full AI dependency tree, version pinning, SBOM status, etc.]

### Unbounded Consumption (LLM10)

[Assessment of rate limiting, timeouts, input size limits across all AI endpoints.]

---

## 7. Priority Matrix

| Priority | OWASP Item | Feature | Action | Effort |
|---|---|---|---|---|
| [critical/high/medium/low] | [item ID] | [feature name] | [mitigation summary] | [low/medium/high] |

Sort by priority (critical first), then by effort (low first within same priority).

---

## 8. Conclusion

### Strengths
- [Architectural patterns or controls that are well-implemented]

### Critical Vulnerabilities
- [Items rated `vulnerable` with highest impact]

### Overall Security Rating

[Choose one: Low / Moderate / Good / Strong]

[2-3 paragraph narrative explaining the rating, referencing the key findings and the project's architectural maturity.]
```

## Guidelines

- **Verify against actual code.** Do not assess from the inventory description alone. Read the files cited in the inventory and examine the code to confirm or refute each finding.
- **Be specific in evidence.** Cite `file:line` for every finding. Vague statements like "the system lacks input validation" are not useful — name the file, the function, and what validation is missing.
- **Adapt mitigations to the codebase.** Don't copy-paste OWASP's generic prevention strategies. Translate them into concrete changes for the specific code patterns you find.
- **Mark `n/a` generously but justify it.** If an item doesn't apply, say why in one sentence. This shows the assessment was considered, not skipped.
- **Assess what's there, not what could be.** If a feature has a guardrail that's partial, acknowledge what it does before explaining what it doesn't.
- **The priority matrix is the most important output for action.** Ensure it's complete, sorted, and actionable.
- **Do not re-scan for AI features.** Work from the inventory. If you discover a feature not in the inventory, note it in the report's Notes section but do not assess it — recommend the user re-run `/analyze-ai-features`.

## Platform Installation

### Claude Code
Symlink this directory to `.claude/skills/owasp-eval/`.

### Windsurf
Symlink this directory to `.windsurf/skills/owasp-eval/`.

### Other platforms
Place `SKILL.md` and the `references/` directory wherever the platform expects skill instructions. The frontmatter uses standard `name` and `description` fields.
