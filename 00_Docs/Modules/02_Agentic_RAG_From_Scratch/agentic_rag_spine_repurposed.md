# Repurposed Spine — Content Kit

**Source:** [agentic_rag_reading_companion.md](agentic_rag_reading_companion.md) · **Spine:** *"Learn the loop, not the library."*

Three ready-to-publish pieces, cascaded per the content brain (Substack article → LinkedIn extract → Substack Note). All in voice: practitioner-led, production-grounded, no hype.

---

## 1. Substack Article — *primary*

**Title (pick 1):**
1. **Learn the Loop, Not the Library** ← recommended
2. Agentic RAG Isn't a New Pipeline. It's One Decision Moved Into the Loop.
3. What Agentic RAG Actually Teaches You About Production

**Subtitle:** Agentic RAG isn't a new retrieval pipeline — it's one decision moved into the agent's loop, and five places production agents actually break.

**Suggested tags:** AI Engineering · RAG · LLM Agents · LangChain · Production AI

---

This week I'm sitting with a room of engineers at AI Makerspace, watching them build their first agent that does RAG. Most of them arrived braced for a hard new framework. It isn't. The framework — LangChain's `create_agent` — is maybe twenty minutes of API. The part that took me a few production deployments to actually learn lives underneath it, and no release notes will teach it to you.

Agentic RAG isn't a more complicated retrieval pipeline. It's the same pipeline with one decision — *when to retrieve* — handed to the model instead of hardcoded by you. That single move is small. What it exposes is not.

### Retrieval becomes a decision — and the tool description *is* the policy

In a fixed RAG chain you always retrieve, then generate. Predictable, easy to reason about, exactly right for an FAQ bot. In an agent, retrieval is a tool the model calls when it decides it needs context. That's the entire shift from a two-step pipeline to an agentic one.

Here's the part teams miss: when retrieval is a tool, **the tool's name and description are the retrieval policy.** The model decides whether to call it based on what you wrote in that description. Leave it vague and the agent retrieves on questions that don't need it — or skips retrieval on the ones that do. That's not a prompt you tune later. It's a contract bug, sitting in a docstring nobody reviewed.

### The demo-to-production gap is a context gap

Every agent demos well. Then it meets real traffic and the instinct is to blame the model. It's almost never the model. The LangChain docs say it flatly: most agent failures come from inadequate context, not model limitations.

Context engineering is the discipline of deciding what actually enters each model call — the system prompt, the message history, the tools you expose, the shape of what comes back. **The right information, in the right format, at the right step.** Hand the model twelve tools when it needs two and you've made its job harder. Let history grow unbounded and you'll blow the context window mid-conversation. None of that shows up in a demo. All of it shows up in production.

### Guardrails belong in middleware

Once the loop runs, you need rules around it: cap how many times it can call a tool so a stuck agent can't bill you for forty model calls, redact PII before it reaches the model, pause for human approval before a sensitive action. The clean way to do all of that is middleware — hooks that run around each step of the loop, kept separate from your agent logic.

This is where my world shows up most directly. In a regulated bank environment, **the PII redaction and the human-in-the-loop approval aren't features — they're the reason the system is allowed to exist.** Bury them inside the agent's core logic and both the logic and the controls get harder to audit. Keep them as middleware and the loop stays readable, the guardrails stay inspectable. In regulated industries, "inspectable" is not optional.

### Drop to a graph when you need to own the control flow

`create_agent` is the fast path and the right default. But the moment you need control it doesn't give you — a deterministic route that rejects out-of-scope questions *before* they cost a model call, a step that grades retrieved documents, a hard cap on retries — you stop configuring and start building the loop yourself with a graph.

The signal is simple: when the behavior you need is a change to the *structure* of the loop rather than something layered *around* it, you've outgrown the high-level API. That's not failure. That's the abstraction ladder working as designed — start high, drop down when the problem demands it.

### Trace everything, because the answer lies

An agent loop is non-deterministic. The same question can take two different paths. So "it gave a good answer" tells you almost nothing about whether the system is healthy — a clean final answer can hide three wasted retrievals and a tool error the model quietly wrote around.

**The bug is rarely in what the agent said — it's in the path it took.** In production, the trace is the first thing I open when an agent misbehaves. Turn tracing on before you need it, not after.

### What actually lasts

`create_agent` will grow new arguments. A competitor will ship a cleaner API next quarter. None of that is the part worth keeping. The part worth keeping is the judgment: make retrieval a decision, engineer the context deliberately, put the guardrails in middleware, drop to a graph when you need the control flow, and trace everything so you can see what the loop did.

Learn the loop, not the library. Then go build something that survives past the demo.

**Key takeaways:**
→ Agentic RAG moves one decision — when to retrieve — into the model's loop; the retriever's description is the policy that governs it.
→ Most agent failures are context failures, not model failures. Engineer what enters each call.
→ Frameworks change; the loop doesn't. Put guardrails in middleware, drop to a graph for control flow, and trace everything.

**CTA:** If your team is moving an agent from a slick demo to something that holds up in production, closing that gap is most of what I do. Reply here, or find me on LinkedIn.

Until next time — keep building things that matter.
— Ovo

---

## 2. LinkedIn Post — *extract*

I've shipped RAG agents into a regulated bank on Azure. The framework was never the hard part.

This week I'm helping a room of engineers build their first agent that does RAG. Most arrive braced for a hard new framework. It isn't.

Agentic RAG isn't a new retrieval pipeline. It's one decision — *when to retrieve* — moved out of your code and into the model's loop.

That one move is small. What it exposes is where production agents actually break — and none of it is the framework API:

→ **Retrieval is a decision.** The retriever's name and description ARE the policy. Vague description, wrong retrievals. That's a contract bug, not a prompt bug.

→ **Context is the real game.** Most agent failures aren't model failures. They're context failures — wrong info, wrong format, wrong step.

→ **Guardrails go in middleware.** Tool budgets, PII redaction, human approval. In regulated industries those aren't features — they're why the system is allowed to exist.

→ **Own the control flow.** `create_agent` is the fast path. The moment you need a deterministic route or a grading step, you build the loop yourself.

→ **Trace everything.** A clean answer can hide three wasted retrievals and a swallowed error. The bug is rarely in what it said — it's in the path it took.

`create_agent` will grow new arguments. A cleaner API ships next quarter. None of that is the part worth keeping.

Learn the loop, not the library.

What breaks first for you — retrieval decisions or context? I'll be in the comments.

---

## 3. Substack Note — *teaser*

Spent the week watching engineers build their first agentic RAG system. The thing that clicks slowest is also the most important:

Agentic RAG isn't a fancier retrieval pipeline. It's one decision — *when to retrieve* — handed to the model instead of hardcoded by you.

Everything hard about production agents lives downstream of that one move: the tool description quietly becomes the retrieval policy, context engineering becomes the whole game, and the trace — not the answer — becomes where you debug.

Wrote up the full version. The loop, not the library: [link]

---

## 4. YouTube / Loom Script — *~3–4 min explainer*

**Type:** Explainer (concept) · **Primary CTA:** subscribe + companion link · Tags: `[B-ROLL]` `[PAUSE]` for delivery.

**HOOK (0:00–0:30)**
[B-ROLL: an agent answering a question, tool calls flashing past]
Most engineers think "agentic RAG" is a harder version of RAG. It isn't. It's the same RAG you already know — with one decision handed to the model instead of hardcoded by you. [PAUSE] And that one change is where every production agent I've debugged actually breaks. In the next few minutes I'll show you the five places it breaks — and why none of them are the framework.

**CREDIBILITY BRIDGE (0:30–1:00)**
Quick context on why I'm the one telling you this. I ship production AI for a living — RAG agents into a regulated bank on Azure, inference systems in AdTech. I also mentor engineers building their first agents. So I've seen both sides: the demo that wins the room, and the same system three weeks later when it meets real traffic. The gap between those two is the whole video.

**WHAT YOU'LL LEARN (1:00–1:30)**
By the end you'll know the five judgments that separate a demo agent from a production one: how retrieval becomes a decision, why context is the whole game, where guardrails belong, when to drop down to a graph, and why you can't trust the final answer. [PAUSE] Stay to the end and I'll give you the one line I'd put above every agent you build.

**CORE — 1 / RETRIEVAL IS A DECISION (1:30–2:00)**
[B-ROLL: fixed RAG chain vs. an agent calling a tool, side by side]
In a fixed RAG chain you always retrieve, then generate. In an agent, retrieval is a tool the model calls when it decides it needs context. [PAUSE] Here's the catch: the tool's name and description ARE the retrieval policy. Write a vague description, and the model retrieves at the wrong times. That's not a prompt bug. It's a contract bug.

**CORE — 2 / CONTEXT IS THE GAME (2:00–2:25)**
Every agent demos well, then falls apart on real traffic, and everyone blames the model. It's almost never the model. Most agent failures are context failures — the wrong information, in the wrong format, at the wrong step. Engineer what goes into each call. That's the job.

**CORE — 3 / GUARDRAILS GO IN MIDDLEWARE (2:25–2:50)**
[B-ROLL: middleware wrapping the loop]
Tool-call budgets, PII redaction, human approval before a sensitive action — those go in middleware, around the loop, not tangled into your agent logic. In a regulated bank, that separation isn't a nicety. It's the reason the system is allowed to ship.

**CORE — 4 / OWN THE CONTROL FLOW (2:50–3:15)**
`create_agent` is the fast path, and the right default. But the moment you need a deterministic route, or a grading step, or a hard retry limit — you stop configuring and build the loop yourself in a graph. The signal: when you need to change the *structure* of the loop, not just wrap it.

**CORE — 5 / TRACE EVERYTHING (3:15–3:40)**
[B-ROLL: a LangSmith trace expanding — tool calls, latency, tokens]
An agent loop is non-deterministic. The same question can take two different paths. So a clean answer tells you almost nothing — it can hide three wasted retrievals and a tool error the model wrote around. The bug is rarely in what it said. It's in the path it took. Turn tracing on before you need it.

**RECAP (3:40–4:00)**
Five judgments. Retrieval is a decision, and the tool description is the policy. Context is the game, not the model. Guardrails live in middleware. Drop to a graph when you need the control flow. And trace everything, because the answer lies. [PAUSE] The framework will change — `create_agent` grows new arguments next quarter. None of that is the part worth keeping.

**CTA (last 20s)**
Here's the line I'd put above every agent you build: *learn the loop, not the library.* If that landed, subscribe — I post production AI, not demo AI. The full written breakdown, plus a from-scratch build of this exact loop, is linked below.

---

## 5. LinkedIn Carousel — *8 slides*

**Caption (post body):**
Agentic RAG isn't a harder version of RAG. It's one decision — *when to retrieve* — moved into the model's loop. And that one move is where production agents actually break. Five judgments I've learned shipping these into a regulated bank 👇

**Slide 1 — Cover**
- **Headline:** Agentic RAG isn't a new pipeline. It's one decision moved into the loop.
- **Sub:** 5 places production agents actually break →

**Slide 2**
- **Headline:** 1 / Retrieval is a decision
- **Body:** The retriever's name and description ARE the policy. Vague description → wrong retrievals. A contract bug, not a prompt bug.

**Slide 3**
- **Headline:** 2 / Context is the whole game
- **Body:** Most agent failures aren't model failures. They're context failures — the wrong info, the wrong format, the wrong step.

**Slide 4**
- **Headline:** 3 / Guardrails go in middleware
- **Body:** Tool budgets. PII redaction. Human approval. In regulated industries those aren't features — they're why the system is allowed to ship.

**Slide 5**
- **Headline:** 4 / Own the control flow
- **Body:** `create_agent` is the fast path. Need a deterministic route or a grading step? Stop configuring — build the loop in a graph.

**Slide 6**
- **Headline:** 5 / Trace everything
- **Body:** A clean answer can hide three wasted retrievals and a swallowed error. The bug is in the path, not the words.

**Slide 7 — Thesis**
- **Headline:** The framework changes every quarter. The loop doesn't.
- **Sub:** Learn the loop, not the library.

**Slide 8 — CTA**
- **Headline:** Follow for production AI, not demo AI.
- **Body:** I ship RAG agents in banking and inference in AdTech. Full breakdown + a from-scratch build → link in comments.

**Design notes:** dark slides, one idea per slide, oversized numerals (1–5), monospace for `create_agent`, ≤ 20 words per slide.
