# Classification Guide: LLM-Direct vs Agentic vs Hybrid

This guide helps classify each AI feature discovered in a repository. Classification determines which OWASP Top 10 list applies during security assessment.

---

## Definitions

### LLM-Direct

The feature uses an LLM as a stateless transformation function: input text → completion → (optional) structured output. The LLM does not decide what to do next, does not persist state across sessions, and does not trigger actions on its own. Code around the LLM call controls all flow.

**Key signals:**
- Single prompt → completion per invocation (no loops, no retries that change the prompt)
- No `tools` array in the model call configuration
- No conversation memory that persists across user sessions
- Output is data (JSON, text, classification), not actions
- Side effects (if any) are executed by deterministic code based on the parsed output, not by the LLM choosing to act

**Common patterns:**
- Gateway classes that wrap a provider SDK call
- Linear chains (prompt template → LLM → parse → validate)
- Embeddings generation
- Audio transcription (Whisper)
- Text classification / summarization / extraction

### Agentic

The feature exhibits autonomous behavior: the LLM (or a framework wrapping it) makes decisions about what to do, uses memory to maintain state across turns or sessions, may call tools, and its output directly triggers real-world actions.

**Key signals (any one strongly suggests agentic, but see decision tree below):**
- Uses an agent framework (`@mastra/core/agent`, `langgraph`, `crewai`, `autogen`, etc.)
- Persistent memory across sessions (conversation history stored in DB, vector store, or file)
- Autonomous decision-making (LLM output determines next action, stage transition, or strategy)
- Multi-step state machine driven by LLM output
- Real-world side effects triggered by LLM decisions (sending messages, updating records, making API calls, financial agreements)
- Tool-calling loop (ReAct pattern: LLM calls tool → observes result → decides next step)
- Human oversight / escalation mechanisms (approval gates, human-in-the-loop)
- Multi-agent delegation (one agent delegates to another)

**Common patterns:**
- Agent classes with `generate()` or `stream()` that produce structured decisions
- Orchestrator use cases that execute side effects based on LLM output
- State machines where transitions are determined by LLM output
- Frameworks with `memory`, `tools`, or `guardrails` configuration

### Hybrid

The feature is primarily LLM-direct but includes a bounded tool call — the LLM can invoke a tool (e.g., web search, function calling) as part of a single completion, but there is no autonomous loop where the LLM decides which tool to call next based on results.

**Key signals:**
- `tools` array in the model call configuration, but single completion (not a loop)
- Built-in provider tools (e.g., OpenAI's `web_search_preview`, code interpreter)
- Function calling where the LLM selects from a fixed set, but the result is consumed in the same turn
- No persistent memory, no multi-step state machine, no side effects from the tool call itself

**Common patterns:**
- Gateway that calls an LLM with `web_search_preview` enabled
- Function calling for data enrichment within a single prompt → completion cycle

---

## Decision Tree

```
1. Does the feature use an LLM or embedding model?
   NO  → Not an AI feature. Skip.
   YES → Continue.

2. Does the feature use an agent framework (Mastra Agent, LangGraph, CrewAI, AutoGen, etc.)?
   YES → Classify as AGENTIC. Fill in agentic properties table.
   NO  → Continue.

3. Does the feature have persistent memory across user sessions?
   (e.g., conversation history stored in MongoDB, vector DB, or file system;
    retrieved and injected into context on subsequent invocations)
   YES → Classify as AGENTIC. Fill in agentic properties table.
   NO  → Continue.

4. Does the LLM output directly trigger real-world side effects?
   (e.g., sending messages, updating database records, making API calls,
    financial transactions — where the LLM decides WHAT action to take,
    not just producing data that code interprets)
   YES → Classify as AGENTIC. Fill in agentic properties table.
   NO  → Continue.

5. Does the feature include a tool-calling loop (ReAct)?
   (LLM calls tool → observes result → decides whether to call another tool
    or produce final answer — multiple iterations)
   YES → Classify as AGENTIC. Fill in agentic properties table.
   NO  → Continue.

6. Does the feature include bounded tool use in a single completion?
   (e.g., `tools` array with web search or function calling, but no loop;
    single prompt → completion with tool augmentation)
   YES → Classify as HYBRID. Fill in agentic properties table (most will be ❌).
   NO  → Continue.

7. Does the LLM output determine state transitions in a multi-step flow?
   (e.g., LLM decides to move from INTAKE → OFERTA stage)
   YES → Classify as AGENTIC. Fill in agentic properties table.
   NO  → Continue.

8. None of the above.
   → Classify as LLM-DIRECT. No agentic properties table needed.
```

---

## Edge Cases

### "Agent" in the name but not agentic

Many codebases name wrapper classes "Agent" (e.g., `EmbeddingAgent`, `HandoffAgent`) that are not autonomous. Check for the actual agentic properties, not the class name. If it's a linear prompt → completion with post-processing, it's `llm-direct`.

### Orchestrator with post-LLM logic

A use case that calls an LLM gateway and then applies business logic (validation, normalization, fallbacks) to the output is `llm-direct` if the LLM doesn't decide what action to take. The orchestrator's logic is deterministic code, not LLM autonomy.

### Framework import without agent features

Importing `@mastra/core` for its non-agent utilities (e.g., memory primitives used outside an agent) doesn't make a feature agentic. Check whether an actual `Agent` instance is created with autonomous behavior.

### Multiple LLM calls in sequence

A feature that makes several LLM calls in sequence (e.g., classify → summarize → extract) is still `llm-direct` if each call is independent and the sequence is hardcoded, not decided by the LLM.

### Bounded tool use with memory

If a feature has bounded tool use (hybrid signal) AND persistent memory (agentic signal), classify as `agentic`. The memory property elevates it.

---

## Agentic Properties Reference

Use this table when filling in the evidence for agentic and hybrid features:

| Property | What to look for | Static signals |
|---|---|---|
| Persistent memory across sessions | Memory store configured, conversation history persisted to DB/vector store | `@mastra/memory`, `Memory`, `checkpoint`, `conversationHistory`, DB collection for messages |
| Autonomous decision-making | LLM output determines action type, strategy, or next step | `tipoAcao`, `action`, `decision`, `nextStep` fields in output schema; conditional logic branching on LLM output |
| Multi-step state machine | Stages/states with transitions driven by LLM output | Enum of stages, transition logic, `stage` field in schema |
| Real-world side effects from LLM output | Messages sent, records updated, API calls made, agreements created — triggered by LLM decisions | Send message calls, DB updates, API calls in code paths following LLM output parsing |
| Input/output guardrails | Input filtering, output validation beyond simple schema validation | `guardrail`, `inputGuardrail`, `outputGuardrail`, `PromptInjectionDetector`, content filtering |
| Human oversight / escalation | Approval gates, human-in-the-loop, escalation flags | `assumidoPor`, `requiresApproval`, `escalate`, `humanInLoop` flags |
| Autonomous tool-calling loop (ReAct) | LLM iteratively calls tools and observes results | `tools` array + loop/iteration logic, `agent.step()`, `ReAct`, `max_iterations` |
| Multi-agent delegation | One agent delegates tasks to another agent | Multiple `Agent` instances, `delegate`, `supervisor`, `router` patterns |
