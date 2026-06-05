# 🔁 The Agent Loop — A Practitioner's Reading Companion

**Session 2 / Module 2: Agentic RAG with LangGraph and LangChain**

The Recommended Reading for this module looks like a stack of documentation links. It isn't. Read in the right order, those ten links are the difference between an agent that demos well on a Tuesday and one that survives contact with production on a Friday at 2am.

I come at this from production — RAG agents shipped into a regulated bank environment on Azure, inference systems for AdTech. So here's the bias I'm bringing: the framework API is the easy part. `create_agent` is a quarter's worth of syntax. The judgment underneath it — when retrieval should be a decision, what context the model actually needs, where the guardrails go, and how to see what the loop did — is what lasts. This companion walks the reading list through that lens.

> [!NOTE]
> **How to use this.** Each section maps one or more of the assigned readings to *why it matters* and *where it shows up in the notebook you're building this week*. The links go to the primary sources — read those. This is the map, not the territory.

---

## 🕴️ 1. Start here: what an "agent" actually is

**Read:** [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) (Yao et al., Oct 2022)

For two years "agent" meant whatever the person saying it wanted to sell you. As of September 2025 the industry mostly [converged on one definition](https://simonwillison.net/2025/Sep/18/agents/):

> An LLM agent runs tools in a loop to achieve a goal.

Hold onto that. The unit of work is the *loop*, not the model.

ReAct is where the loop got its spine. Before it, "reasoning" (chain-of-thought) and "acting" (tool calls) were studied as separate tricks. Yao et al. interleaved them: the model writes a short reasoning trace, takes an action, reads the result, reasons again.

> reasoning traces help the model induce, track, and update action plans as well as handle exceptions, while actions allow it to interface with and gather additional information from external sources.

Here's why that's not just academic. Pure chain-of-thought hallucinates because it never checks itself against the world. Pure action without reasoning can't recover when a step fails. Interleaving them is what lets an agent *notice* a weak retrieval and try a different query — the exact behavior you want from agentic RAG. The paper backs it with numbers: +34% absolute success over baselines on ALFWorld and +10% on WebShop, with one or two in-context examples.

**In your notebook:** every time the cat-health agent decides to call `retrieve_cat_health_guidelines`, reads the result, and answers — that's ReAct. You didn't invent a new pattern. You implemented a 2022 one on a 2025 framework.

---

## 🧱 2. The ground you're standing on: LangChain & LangGraph 1.0

**Read:** [LangChain & LangGraph 1.0 release](https://www.langchain.com/blog/langchain-langgraph-1dot0) (Oct 2025) · [Philosophy](https://docs.langchain.com/oss/python/langchain/philosophy) · [Component Architecture](https://docs.langchain.com/oss/python/langchain/component-architecture)

A lot of agent tutorials age badly because the framework moves under them. That changed with 1.0. The parts that matter for you:

- **`create_agent` is now the standard loop** — call the model with tools → run the tools it asked for → repeat until a final answer. One function, the whole ReAct loop.
- **Middleware is a first-class system**, not a workaround (more in §5).
- **`.content_blocks` standardizes the message format across providers.** Swapping OpenAI for Anthropic used to break your streaming and your UI. Now it doesn't.
- **LangChain is built *on* LangGraph's runtime.** You "start with LangChain's high-level APIs and drop down to LangGraph when you need more control." That one sentence is the entire architecture of this session.
- **Stability: no breaking changes until 2.0.** Legacy code moved to `langchain-classic`. This is the first time it's been safe to build a *curriculum* — or a product — on this surface without a rewrite every six months.

The [Philosophy](https://docs.langchain.com/oss/python/langchain/philosophy) doc explains the *why*: standard interfaces and model-agnosticism so you "avoid lock-in," and a deliberate ladder of abstraction — LangChain (high-level), LangGraph (low-level orchestration), Deep Agents (batteries-included). It's not about convenience *or* control; it's a dial you turn per use case.

**The lesson:** you're not learning "the LangChain API." You're learning an abstraction ladder. The skill is knowing which rung to stand on.

---

## 🧰 3. Retrieval stops being a step and becomes a decision

**Read:** [Retrieval](https://docs.langchain.com/oss/python/langchain/retrieval)

Session 1 was a fixed pipeline:

```
question → always retrieve → generate
```

Predictable, low-variance, easy to reason about. The LangChain docs name three architectures, and the jump you're making this week is from the first to the second:

| Architecture | Who decides to retrieve | Where it fits |
|---|---|---|
| **2-step RAG** | The pipeline. Always. | FAQ and docs bots — predictable latency, high control |
| **Agentic RAG** | The model, at reasoning time | Open-ended questions where retrieval isn't always needed |
| **Hybrid RAG** | The model *plus* validation (grade, rewrite, re-retrieve) | Production systems that can't afford a wrong answer |

The mechanics don't change — loaders, splitters, embeddings, a vector store ([Qdrant](https://qdrant.tech/) here), a retriever. What changes is *who holds the decision*. In agentic RAG the retriever is a tool, and **the tool's name and description are the retrieval policy.** Write a vague description and the model retrieves at the wrong times. That's not a prompt bug — it's a contract bug.

**In your notebook:** the move from "always retrieve" to "the agent calls `retrieve_cat_health_guidelines` when it decides it needs context." The Advanced Build — grade the docs, rewrite weak queries, cap the retries — is Hybrid RAG. That's not extra credit. That's the version that ships.

---

## 🪟 4. The part nobody warns you about: context engineering

**Read:** [Context](https://docs.langchain.com/oss/python/concepts/context) · [Context engineering in agents](https://docs.langchain.com/oss/python/langchain/context-engineering)

If you take one thing from this whole list, take this. The demo-to-production gap is mostly a context gap. The docs say it without flinching: most agent failures come from "inadequate context, not model limitations."

First, untangle a naming collision. **Runtime context ≠ the LLM context window.** Runtime context is dependency injection — handing your tools a database connection, a user id, config. The docs split it three ways: *static* (passed once), dynamic *state* (changes during a run — LangGraph's short-term memory), and cross-conversation *store* (persists across sessions — long-term memory).

Then there's *context engineering* — deciding what actually goes into each model call. Three surfaces:

- **Model context** (transient): system prompt, message history, available tools, the model itself, the response schema. Reset every call.
- **Tool context** (persistent): what tools can read and write — state, store, config.
- **Life-cycle context** (persistent): what middleware does *between* calls — summarize, guard, log.

Techniques worth stealing: dynamic system prompts that change with conversation length; **selective tool exposure** (don't hand the model twelve tools when it needs two); model switching (cheap model early, bigger model once the conversation gets long); `SummarizationMiddleware` to trim history before it blows the window.

This is old wisdom with a new name. Dex Horthy's [12-Factor Agents](https://github.com/humanlayer/12-factor-agents) puts it plainly: *"Everything that makes agents good is context engineering."* Own your context window. Own your control flow.

**In your notebook:** your `AGENT_SYSTEM_PROMPT`, the source-labeled chunks the tool returns, the single retrieval tool you expose (that *is* selective tool exposure — you gave it exactly one). Small decisions. They're the whole ballgame.

---

## 🛡️ 5. Middleware: where the guardrails live

**Read:** [Middleware](https://docs.langchain.com/oss/python/langchain/middleware/overview)

Middleware is how you change what the loop does *without rebuilding the loop*. It exposes hooks around each step — before the model call, after it, around tool execution — and ships with built-ins you'll reach for constantly:

- `ToolCallLimitMiddleware` / `ModelCallLimitMiddleware` — budgets, so a stuck agent can't bill you for 40 model calls
- `SummarizationMiddleware` — context-window management
- `HumanInTheLoopMiddleware` — pause for approval before a sensitive tool runs
- PII redaction — strip sensitive data before it reaches the model

Here's where my world shows up. In a regulated industry, the PII redaction and the human-in-the-loop approval aren't *features* — they're the reason the system is allowed to exist at all. Middleware is where that lives, cleanly separated from your agent logic.

**In your notebook:** Activity #1 adds `ToolCallLimitMiddleware(run_limit=1)` to cap retrieval — watch what it blocks. The mental model (I wrote this into the [lesson explainer](agentic_rag_lesson_explainer.md) too): **middleware acts *around* the loop; LangGraph changes the *structure* of the loop.** Which is exactly the next reading.

---

## 🕸️ 6. When `create_agent` isn't enough: thinking in LangGraph

**Read:** [Thinking in LangGraph](https://docs.langchain.com/oss/python/langgraph/thinking-in-langgraph)

`create_agent` is the high rung. The moment you need control it doesn't give you — a deterministic route, a grading step, a hard loop limit — you drop down to LangGraph and build the loop yourself out of three pieces:

- **Nodes** — functions that change state (a model call, a retrieval, a router)
- **Edges** — where state goes next
- **State** — the docs call it "the notebook your agent uses to keep track of everything it learns"

The doc teaches a decomposition habit worth internalizing: map the workflow as discrete steps → identify the operation types → design state to hold *raw* data → build single-responsibility nodes → wire them with minimal explicit edges. One principle underneath it pays off forever: **raw data in state, formatted prompts in nodes.** Don't bake a prompt into your state object; let each node format what it needs. And smaller nodes mean more checkpoints — so when a step fails, you recover with less repeated work.

This is also where human-in-the-loop gets serious: LangGraph's `interrupt()` pauses the graph indefinitely, saves state, and resumes exactly where it stopped. You can't do that with a plain Python `while` loop.

**In your notebook:** BOR2 rebuilds the same agent with `StateGraph`, `MessagesState`, a model node, `ToolNode`, and `tools_condition`. Activity #2 (deterministic scope routing) and the Advanced Build (grader, rewriter, loop limit, guardrail) are graph-level changes — things you literally cannot express with middleware alone. That gap is the signal you've outgrown the high rung.

---

## 🔭 7. You can't fix what you can't see: observability

**Read:** [LangSmith Observability Concepts](https://docs.langchain.com/langsmith/observability-concepts)

An agent loop is non-deterministic. The same question can take a different path twice. So "it gave a good answer" tells you almost nothing about whether the *system* is healthy — a plausible answer can hide three wasted retrievals and a tool error the model quietly papered over.

LangSmith gives you the trace. The hierarchy is simple: **Projects** hold **Traces** (one per request), which hold **Runs** (one per step — a model call, a retrieval). A Run is a Span in OpenTelemetry terms. Per trace you see the tool calls, latency, token usage, errors, and any feedback scores you attach.

**The lesson — and it's the one I keep repeating to the junior engineers I mentor:** never trust only the final answer. In production the trace is the first thing I open when an agent misbehaves, because the bug is almost never in the words it said — it's in the path it took.

**In your notebook:** set `LANGSMITH_TRACING=true`, run the three test questions, and *read the trace*. Did it retrieve when it should have? Did it skip retrieval on the off-topic question? The streaming output in the notebook is the same instinct in miniature.

---

## 🗺️ How the reading maps to what you build

| Reading | Shows up as |
|---|---|
| ReAct | The retrieve → reason → answer loop itself |
| LangChain/LangGraph 1.0 · Philosophy · Component Architecture | `create_agent`, the abstraction ladder, the whole stack |
| Retrieval | Session 1 → Session 2; the retriever-as-tool; Advanced Build = Hybrid RAG |
| Context · Context engineering | Your system prompt, the single exposed tool, source-labeled chunks |
| Middleware | Activity #1 (`ToolCallLimitMiddleware`); the BOR1 logger + call limit |
| Thinking in LangGraph | BOR2 explicit graph; Activity #2 routing; the Advanced Build |
| Observability | `LANGSMITH_TRACING`; streaming the agent's decisions |

And if you ran the optional [from-scratch notebook](02_Cat_Health_Agentic_RAG_From_Scratch.ipynb): every one of these — the loop, the tool contract, context assembly, the step records — is a function you wrote by hand. The frameworks don't add intelligence. They add the plumbing this reading list describes.

---

## ⏱️ A reading order that respects your time

You won't read ten links tonight. So:

1. **If you read nothing else:** [ReAct](https://arxiv.org/abs/2210.03629) (the loop) and [Context engineering in agents](https://docs.langchain.com/oss/python/langchain/context-engineering) (why it breaks).
2. **Before you touch the notebook:** the [LangChain 1.0 blog](https://www.langchain.com/blog/langchain-langgraph-1dot0) and [Retrieval](https://docs.langchain.com/oss/python/langchain/retrieval).
3. **When you hit BOR2:** [Thinking in LangGraph](https://docs.langchain.com/oss/python/langgraph/thinking-in-langgraph).
4. **When something misbehaves:** [Observability Concepts](https://docs.langchain.com/langsmith/observability-concepts).
5. **For depth, later:** [Philosophy](https://docs.langchain.com/oss/python/langchain/philosophy) · [Component Architecture](https://docs.langchain.com/oss/python/langchain/component-architecture) · [Context](https://docs.langchain.com/oss/python/concepts/context) · [Middleware](https://docs.langchain.com/oss/python/langchain/middleware/overview).

---

## The lesson

The loop is the same everywhere you'll meet it — in a 2022 research paper, in `create_agent`, in a LangGraph `StateGraph`, in 200 lines of standard-library Python:

```
model decides → application executes → result returns to model → repeat or answer
```

The framework will change. `create_agent` will grow new arguments; a competitor will ship a cleaner API next quarter. None of that is the part worth keeping. The part worth keeping is the judgment this reading list is quietly teaching: make retrieval a decision, engineer the context deliberately, put the guardrails in middleware, drop to a graph when you need to own the control flow, and trace everything so you can see what the loop actually did.

Learn the loop, not the library. Then go build something that survives past the demo.

---

*Companion to Session 2. Source readings are linked inline — read the primaries; this is the map.*
