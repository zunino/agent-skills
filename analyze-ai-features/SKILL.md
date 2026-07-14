---
name: analyze-ai-features
description: Scan a repository to identify and classify all AI functionality as LLM-direct, agentic, or hybrid. Produces a structured inventory file for downstream OWASP security assessment.
---

# /analyze-ai-features

## Purpose

Scan a codebase to discover every AI/LLM feature, classify each one, and produce a structured inventory at `docs/ai-sec-analysis/ai-features.md`. This inventory is the input for the `/owasp-eval` skill.

This skill does **not** perform security analysis. It only identifies and classifies.

## Inputs

- **Repository path** — the root of the repository to analyze. If not specified, use the current workspace root.
- **Output path** (optional) — defaults to `docs/ai-sec-analysis/ai-features.md`, relative to the repository root.

## Output

A single markdown file at `docs/ai-sec-analysis/ai-features.md` (created if it does not exist). The file follows the structure defined in [Output Format](#output-format) below.

## Workflow

### Step 1 — Discover AI dependencies

Search dependency manifests for AI-related packages. Check all relevant files:

- `package.json` / `package-lock.json` / `yarn.lock` (Node.js)
- `requirements.txt` / `pyproject.toml` / `Pipfile` (Python)
- `go.mod` (Go)
- `pom.xml` / `build.gradle` (Java/Kotlin)
- `Cargo.toml` (Rust)
- `Gemfile` (Ruby)

Look for packages in these categories (non-exhaustive):

| Category | Examples |
|---|---|
| LLM provider SDKs | `openai`, `@anthropic-ai/sdk`, `google-genai`, `cohere`, `mistralai` |
| AI gateway / router | `ai` (Vercel AI SDK), `litellm`, `langfuse` |
| Agent frameworks | `@mastra/core`, `langchain`, `langgraph`, `crewai`, `autogen`, `llama-index`, `haystack`, `semantic-kernel` |
| Memory / state | `@mastra/memory`, `mem0`, `langgraph.checkpoint` |
| Embeddings / vector | `chromadb`, `pinecone-client`, `pgvector`, `weaviate-client`, `@mastra/mongodb` |
| Transcription / multimodal | `whisper`, `deepgram-sdk` |

Record each found package, its version, and category.

### Step 2 — Trace dependencies to call sites

For each AI dependency found, locate where it is imported and used:

1. Grep for import/require statements referencing the package name.
2. For each import site, trace the call chain to understand what the code does:
   - What model is called?
   - What inputs go into the prompt?
   - What happens to the output?
   - Are there tools configured?
   - Is there persistent memory?
   - Does the output trigger side effects (messages, DB writes, API calls)?

3. Group call sites into **features**. A feature is a coherent unit of AI functionality — typically a gateway class, a service, a chain, or an agent. If multiple files implement one logical capability (e.g., a gateway + a use case that orchestrates it), group them as one feature.

### Step 3 — Classify each feature

For each feature, determine its classification: `llm-direct`, `agentic`, or `hybrid`.

Read `references/classification-guide.md` for the full criteria and decision tree. The summary:

- **llm-direct**: Linear prompt → completion → (optional) structured output parsing. No autonomous decisions, no persistent memory across sessions, no tool-calling loops, no side effects from LLM output. The LLM produces data; code acts on it.
- **agentic**: Uses an agent framework or exhibits agentic properties: persistent memory, autonomous decision-making, multi-step state machines, side effects triggered by LLM output, tool-calling loops (ReAct), human oversight mechanisms, or multi-agent delegation.
- **hybrid**: Primarily LLM-direct but includes a bounded tool call (e.g., web search, function calling) without a full agentic loop. The LLM can invoke a tool but doesn't autonomously decide which tools to use in a loop.

### Step 4 — Collect evidence

For each feature, record concrete evidence:

- **File path(s)** — primary file and any orchestrator/caller
- **Model / provider** — e.g., `gpt-5-mini`, `gemini-2.5-flash-lite`
- **Framework** — e.g., `Vercel AI SDK`, `@mastra/core Agent`, `LangChain Chain`
- **Agentic properties table** — for features classified as `agentic` or `hybrid`, fill in:

| Property | Present? | Evidence |
|---|---|---|
| Persistent memory across sessions | ✅/❌ | file:lines — description |
| Autonomous decision-making | ✅/❌ | file:lines — description |
| Multi-step state machine | ✅/❌ | file:lines — description |
| Real-world side effects from LLM output | ✅/❌ | file:lines — description |
| Input/output guardrails | ✅/❌ | file:lines — description |
| Human oversight / escalation | ✅/❌ | file:lines — description |
| Autonomous tool-calling loop (ReAct) | ✅/❌ | file:lines — description |
| Multi-agent delegation | ✅/❌ | file:lines — description |

For `llm-direct` features, the agentic properties table is omitted.

### Step 5 — Write the inventory file

Write the output to `docs/ai-sec-analysis/ai-features.md` using the format below.

## Output Format

```markdown
# AI Feature Inventory — [repository name]

**Generated:** [date]
**Repository:** [path or URL]
**Analyzer:** /analyze-ai-features

---

## Summary

- Total AI features: [N]
- LLM-direct: [N]
- Agentic: [N]
- Hybrid: [N]

## AI Dependencies

| Package | Version | Category |
|---|---|---|
| [name] | [version] | [category] |

---

## LLM-Direct Features

| Feature | File | Model | Framework | Description |
|---|---|---|---|---|
| [name] | [path] | [model] | [framework] | [1-2 sentence description] |

---

## Agentic Features

### [Feature name]

**File(s):** [path(s)]
**Framework:** [framework]
**Model:** [model]
**Description:** [2-3 sentences]

| Agentic Property | Present? | Evidence |
|---|---|---|
| Persistent memory across sessions | ✅/❌ | [evidence] |
| Autonomous decision-making | ✅/❌ | [evidence] |
| Multi-step state machine | ✅/❌ | [evidence] |
| Real-world side effects from LLM output | ✅/❌ | [evidence] |
| Input/output guardrails | ✅/❌ | [evidence] |
| Human oversight / escalation | ✅/❌ | [evidence] |
| Autonomous tool-calling loop (ReAct) | ✅/❌ | [evidence] |
| Multi-agent delegation | ✅/❌ | [evidence] |

---

## Hybrid Features

### [Feature name]

**File(s):** [path(s)]
**Framework:** [framework]
**Model:** [model]
**Tool use:** [description of bounded tool call]
**Description:** [2-3 sentences]

| Agentic Property | Present? | Evidence |
|---|---|---|
| ... | ... | ... |

---

## Notes

[Any observations about the codebase's AI architecture, patterns, or anomalies discovered during analysis.]
```

## Guidelines

- **Be thorough but not redundant.** If 20 gateways follow the same pattern (e.g., `UC → Gateway → VercelGateway`), list them all in the table but describe the shared pattern once in Notes.
- **Cite file paths and line numbers** in evidence. The downstream OWASP assessment will need to locate the code.
- **When classification is ambiguous**, classify as `hybrid` and explain the ambiguity in the feature's description.
- **Do not perform security analysis.** No OWASP references, no vulnerability assessment, no mitigation recommendations. That is the job of `/owasp-eval`.
- **Do not skip features.** Even trivial features (e.g., embeddings-only) should be listed. The OWASP assessment will determine relevance.

## Next Step

After producing the inventory, the user should review it. When ready for security assessment, run `/owasp-eval` with this inventory file as input.

## Platform Installation

### Claude Code
Copy or symlink this directory to `.claude/skills/analyze-ai-features/`.

### Windsurf
Copy or symlink this directory to `.windsurf/skills/analyze-ai-features/`.

### Other platforms
Place `SKILL.md` and the `references/` directory wherever the platform expects skill instructions. The frontmatter uses standard `name` and `description` fields.
