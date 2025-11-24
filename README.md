# 🧱 BRXBOX ⧉ A Design Model for Synthetic Cognition

Snap together AI bricks into BRXgraphs and observe what mind takes shape inside.

_Status: v0.2.0 (alpha design model)_

---

## 🧩 0. What is BRXBOX?

BRXBOX is a **design model for synthetic cognition**: a way to **design AI systems as modular assemblies**, not as single sealed black boxes.

Instead of “the AI does X,” BRXBOX gives you:

- **BRX (bricks)** – components with a clear role  
  - e.g. OCR engine, LLM, vector database, rules engine, dashboard
- **TRX (tracks)** – the data and control flows between them  
  - e.g. `complaint text → embedding → similarity search → LLM summary`
- **PLX (plex)** – the wiring pattern / topology that shapes behavior  
  - e.g. line, loop, star, bus/blackboard

You can think of it as:

> **CAD + BOM + topology language for cognitive assemblies** – a way to draw, name, and reason about AI “Boxes” made from many interacting parts.

BRXBOX is **framework-agnostic**. You can implement a design in LangChain, LangGraph, CrewAI, DSPy, n8n, or plain Python. BRXBOX only cares about *what the parts are* and *how they connect*.

In one sentence:

> **Your BOX is defined by your BRXgraph: BRX connected by TRX arranged in a PLX.**

---

## 📚 1. Core concepts & naming

### 🧱 1.1 BRX, TRX, PLX, BRXgraphs, BOXes

- **BRX (bricks)** – single modules with a clear role, model shape, and interface.  
- **TRX (tracks)** – connections describing how data/control moves from one BRX to another.  
- **PLX (plex)** – the topology/layout formed by TRX between BRX (line, loop, star, bus…).  
- **BRXgraph** – the *blueprint*: a graph of BRX and TRX in a chosen PLX.  
- **BOX** – a *running system* that instantiates a BRXgraph: your actual agent/service.

You design a **BRXgraph**, then realize it as a **BOX**.

BRXBOX (the project) is the design model and pattern language you use to do that.

---

### 🏷️ 1.2 BRX IDs: ROLE.SHAPE.INTERFACE

Each BRX gets a compact ID:

> `ROLE.SHAPE.INTERFACE`

where:

- **ROLE** = what it does (perception, reasoning, memory, etc.)  
- **SHAPE** = what kind of model/algorithm it is (transformer, CNN, diffusion…)  
- **INTERFACE** = how you call it (function, tool, retriever…)

Example IDs:

- `PERC.VISION-TRANSFORMER.FN` – vision encoder as a function  
- `REASON.TEXT-LLM.CHAT` – chatting text LLM  
- `MEM.VECTOR.STORE` – vector store  
- `CTRL.AGENT-LLM.TOOL` – tool-using LLM agent  
- `ENV.API.HTTP` – HTTP API environment

This naming scheme is easy to read, grep, and copy-paste into configs.

---

### 🌍 1.3 SCOPE: how “far” a BRX can see

Alongside `ROLE.SHAPE.INTERFACE`, you can optionally tag a BRX with a **scope**:

- `SCOPE: LOCAL` – operates on a narrow step or subtask  
- `SCOPE: NESTED` – operates inside a module or subgraph  
- `SCOPE: GLOBAL` – has visibility over the whole BOX state  
- `SCOPE: META` – reads or writes **BRXgraphs themselves** (design-time / governance)

Examples:

- A normal LLM tool-caller: `CTRL.AGENT-LLM.TOOL`, `SCOPE: LOCAL`.  
- A locality orchestrator: `CTRL.ORCH-LOCALITY.TOOL`, `SCOPE: GLOBAL`.  
- A meta-architect that proposes new BRXgraphs: `ARCH.BRXGRAPH-COMPOSER.FN`, `SCOPE: META`.

Scope doesn’t introduce new math; it just makes it explicit **which BRX are looking out across the whole system**, and which live down in the trenches.

---

### 🪧 1.4 A minimal BRXgraph example

Here’s a tiny BRXgraph YAML for a simple “sign reader” BOX:

    bricks:
      - id: PERC.VISION-OCR.FN
        role: PERC
        shape: OCR
        interface: FN
        scope: LOCAL
        desc: "Image crop → text"

      - id: REASON.TEXT-LLM.CHAT
        role: REASON
        shape: TRANSFORMER-SEQ
        interface: FN
        scope: LOCAL
        desc: "Explain sign in plain language"

    tracks:
      - from: PERC.VISION-OCR.FN
        to:   REASON.TEXT-LLM.CHAT
        type: LINE
        desc: "Pipe recognized text into the LLM"

PLX here is a simple **line**.

This BRXgraph defines a very small BOX:

> image in → OCR BRX → LLM BRX → explanation out

You can extend this BRXgraph with memory BRX, critics, or control loops without changing the core idea.

---

## 🧮 2. BRX taxonomy cheat sheet

Here’s a quick cheat sheet of common BRX patterns you can copy into your own designs.

| Role   | Example ID                      | What it does                                       |
|--------|---------------------------------|----------------------------------------------------|
| PERC   | `PERC.VISION-CNN.FN`           | Image → feature map / text / labels                |
| PERC   | `PERC.AUDIO-ASR.FN`            | Audio → transcript                                 |
| PERC   | `PERC.MUSIC-TOK.FN`            | Audio/MIDI → symbolic music tokens                 |
| REASON | `REASON.TEXT-LLM.CHAT`         | General-purpose text reasoning                     |
| REASON | `REASON.MUSIC-TRANSFORMER.FN`  | Analyze chords/keys/structure over music tokens    |
| GEN    | `GEN.TEXT-LLM.FN`              | Draft text given a specification                   |
| GEN    | `GEN.IMG-DIFFUSION.FN`         | Generate images from prompts                       |
| MEM    | `MEM.VECTOR.STORE`             | Store/retrieve embeddings                          |
| MEM    | `MEM.GRAPH.STORE`              | Store entities/relations as a graph                |
| MEM    | `MEM.EPISODIC.STORE`           | Append-only log of events/interactions             |
| CTRL   | `CTRL.AGENT-LLM.TOOL`          | LLM agent that calls tools                         |
| CTRL   | `CTRL.CRITIC-LLM.FN`           | LLM that reviews/evaluates another BRX’s output    |
| CTRL   | `CTRL.ROUTER.FN`               | Route queries to specialized BRX                   |
| ENV    | `ENV.API.HTTP`                 | Wrap an HTTP API as an environment                 |
| ENV    | `ENV.FS.LOCAL`                 | Local filesystem surface                           |
| ARCH   | `ARCH.BRXGRAPH-COMPOSER.FN`    | Propose/compare alternate BRXgraphs (alpha pattern)|

You’re free to define your own roles/shapes as long as you’re consistent.

---

## 🧠 3. Memory stacks across time

BRXBOX encourages explicit **time-layered memory** instead of one vague “DB”. You model different kinds of MEM BRX:

1. **Working Memory**  
   - The current context window and scratchpads.  
   - Lives in the LLM context / current call.  
   - Ephemeral: resets easily.

2. **Episodic Memory** – `MEM.EPISODIC.STORE`  
   - Time-stamped logs of interactions and events.  
   - “What happened, in what order, under what conditions?”

3. **Semantic Memory** – `MEM.VECTOR`, `MEM.GRAPH`  
   - Distilled knowledge across many episodes.  
   - Facts, entities, relationships, patterns.

4. **Procedural Memory** – `MEM.PROCEDURAL.STORE`, `CTRL.POLICY`  
   - Skills, workflows, policies, preferences.  
   - “How this BOX tends to behave.”

A common pattern:

- Working → Episodic: log each significant interaction.  
- Episodic → Semantic: periodically compress patterns into vectors/graphs.  
- Semantic → Procedural: adjust policies/defaults based on stable patterns.  
- Episodic + Semantic + Procedural → Working: retrieve what’s relevant for the current situation.

This stack is implementable with current tools and makes BOXes feel coherent over time.

Some **CTRL** or **ARCH** BRX may operate in **slow time** (e.g. nightly jobs that scan episodic logs and update semantic/procedural stores). You can tag them with `SCOPE: GLOBAL` or `SCOPE: META` to make their role explicit.

---

## 🕸️ 4. PLX: how BOXes are wired

Any full system is a **BRXgraph**: a set of BRX connected by TRX arranged in some **PLX** (plex).  

PLX has two aspects:

- **Topology** – the basic shape: line, loop, star, bus, etc.  
- **Depth / layering** – whether there’s an inner core and outer shells.

You don’t have to name depth formally yet, but it helps to *think* in those terms.

---

### 📏 4.1 Line PLX – simple pipelines

    PERC → REASON → GEN

Examples:

- Captioning: `image → PERC.VISION → REASON.TEXT-LLM → caption`.  
- ASR + LLM: `audio → PERC.AUDIO-ASR → REASON.TEXT-LLM`.

Lines are great for “input → interpret → output” tasks.

---

### 🔁 4.2 Loop PLX – control circuits

    PERC → CTRL.POLICY → ENV → PERC → …

Examples:

- Robot controllers.  
- Game-playing agents.  
- Continuous monitoring systems.

Loops are where a BOX starts to react to its own past actions and ongoing state.

Some loops are **fast-time** (turn-by-turn interaction); others are **slow-time** (e.g. nightly audits, retraining). Slow-time loops often involve `CTRL` or `ARCH` BRX with `SCOPE: GLOBAL` or `META`.

---

### ✴️ 4.3 Star PLX – many experts, one conductor

Multiple specialists around a core controller:

    input → many REASON/GEN experts → CTRL.ROUTER/CTRL.CRITIC → output

Examples:

- Math specialist + code specialist + safety specialist around a core LLM.  
- Several domain-tuned LLMs whose answers are fused.

Star PLX captures “chorus of experts with a conductor” architectures.

---

### 🚌 4.4 Bus / Blackboard PLX – shared hub

BRX read and write via a shared memory/environment hub:

    many BRX ↔ MEM.* or ENV.* hub

Examples:

- Event bus where all tools publish/subscribe.  
- Blackboard system where perception, reasoning, and planning all update a shared world model.

Bus/blackboard plexes fit well with multi-agent or multi-skill BOXes.

---

### 🧬 4.5 Core + Shell PLX (nested)

Many “agent-y” BOXes feel like:

- an **inner core** – a tight perception/reasoning/memory loop, and  
- an **outer shell** – orchestrators that handle channels, localities, or policies.

You can think of this as a **nested PLX**:

- Inner PLX: small loop/triangle/star (e.g. `PERC + REASON + MEM`), often `SCOPE: LOCAL`.  
- Outer PLX: one or more CTRL BRX facing `ENV.*`, often `SCOPE: GLOBAL`.

Example mental model:

- Inner core: “how this Box thinks”  
- Outer shell: “where, for whom, and under what constraints it thinks”

You don’t need new syntax yet; just be aware that **some BRXgraphs are naturally layered** this way.

---

### 🎛️ 4.6 Small PLX motifs: triads and squares

Some small plexes show up repeatedly inside bigger BRXgraphs:

- **Perception–Reason–Memory**  
  - `PERC + REASON + MEM`  
  - Example: visual QA over your documents.  
  - Behavior: grounded, context-aware answers.

- **Generator–Critic–Planner**  
  - `GEN + CTRL.CRITIC + CTRL.AGENT`  
  - Example: code generator with tests and retries.  
  - Behavior: self-correcting generation.

- **Language–Memory–Policy (“Agent Triangle”)**  
  - `REASON.TEXT + MEM + CTRL.AGENT`  
  - Example: tool-using assistant with RAG and preferences.  
  - Behavior: multi-step tool use with continuity.

- **Perception–Reason–Memory–Environment (Square)**  
  - `PERC → REASON → ENV → MEM → PERC → …`  
  - Example: an agent that acts, remembers outcomes, and adapts.

You’ll see these PLX motifs inside bigger BRXgraphs all the time.

---

## 🎯 5. Example BOXes

### 🚏 5.1 Street Sign Buddy → Street Sign Lorekeeper

**BOX A: Street Sign Buddy**

Goal: read street signs and explain them.

BRX:

- `PERC.VISION-CNN.FN` – detect and crop signs from camera frames.  
- `PERC.OCR.FN` – sign crop → text.  
- `REASON.TEXT-LLM.CHAT` – explain the sign in plain language.  
- Optional: `GEN.AUDIO.TOOL` – text-to-speech.

PLX: a simple **line**.

Flow:

1. Frame → `PERC.VISION-CNN` → sign crop.  
2. Crop → `PERC.OCR` → text.  
3. Text → `REASON.TEXT-LLM` → explanation.  
4. Explanation → user (text or audio).

Output:

> “No parking here from 9am–4pm on weekdays. You’re okay right now.”

---

**BOX B: Street Sign Lorekeeper**

Extend the same BOX with history + pattern awareness.

Add BRX:

- `MEM.EPISODIC.STORE` – log each sign encounter (time, place, outcome).  
- `MEM.GRAPH.STORE` – link signs ↔ streets ↔ tickets ↔ user history.  
- `CTRL.CRITIC-LLM.FN` – look for relevant patterns (“you’ve been ticketed here before”).

Extended flow:

1. Read & explain sign as before.  
2. Log event to `MEM.EPISODIC`.  
3. Update `MEM.GRAPH` with the new encounter.  
4. `CTRL.CRITIC-LLM` queries graph for relevant history.  
5. `REASON.TEXT-LLM` fuses sign text + graph insights into advice.

New kind of output:

> “This block has street cleaning Tuesdays 2–4pm. You got a ticket here last month. Parking one block east has been safer for you.”

Same core BRX, different PLX + memory = different *feel*.

---

### 🎵 5.2 Music Critic BOX

Goal: listen to a piece of music and explain its structure/theory.

BRX:

- `PERC.MUSIC-TOK.FN` – audio/MIDI → symbolic tokens (notes, chords, measures).  
- `REASON.MUSIC-TRANSFORMER.FN` – analyze keys/progressions/sections.  
- `MEM.GRAPH.STORE` – store recurring patterns as a graph.  
- `REASON.TEXT-LLM.CHAT` – explain analysis in natural language.  
- Optional: `CTRL.CRITIC-LLM.FN` – sanity-check the theory.

PLX: **star-ish** (music analyzer + graph + text explainer).

Flow:

1. Audio → `PERC.MUSIC-TOK`.  
2. Tokens → `REASON.MUSIC-TRANSFORMER` → structured analysis (JSON).  
3. Analysis → `MEM.GRAPH` (update motifs, cadences, influences).  
4. Analysis + graph context → `REASON.TEXT-LLM` → explanation.

Example explanation:

> “This song is in G major with a ii–V–I progression in the verses. The bridge borrows chords from the parallel minor, which gives you that darker contrast before resolving back to G.”

As the graph fills, you can add stylistic comments grounded in actual recurrence.

---

## 🧰 6. Using BRXBOX in your own projects

You can adopt BRXBOX at several levels.

### 🔍 6.1 Describe what you already have

- List your components and tag them with `ROLE.SHAPE.INTERFACE` as BRX (optionally add `SCOPE`).  
- Draw your system as a BRXgraph (BRX + TRX + PLX).  
- Ask: is this a line PLX, a loop, a star, a bus, or a mix? Is there a core + shell?

This alone makes complex stacks easier to talk about.

---

### 🧭 6.2 Design new BOXes more intentionally

Starting from a task:

1. **Clarify the job**  
   - “Sign reader,” “complaint aggregator,” “music tutor,” “lab monitor,” etc.

2. **Pick BRX roles you need**  
   - Perception? Reasoning? Memory? Control? Environment? Any ARCH needs?

3. **Pick BRX shapes**  
   - Transformers? CNNs? Vector DB? Graph? Rule engine?

4. **Pick a PLX**  
   - Straight pipeline? Loop with environment? Star of experts? Bus/blackboard? Core + shell?

5. **Write a BRXgraph**  
   - Minimal YAML/JSON describing BRX and TRX and the intended PLX.

6. **Implement with your favorite framework**  
   - LangChain, LangGraph, CrewAI, DSPy, or just Python scripts.

---

### 🔁 6.3 Iterate on BRXgraphs

- Add or swap MEM BRX to change how the BOX remembers.  
- Add critics or agents (CTRL BRX) to change how it self-checks.  
- Introduce slow-time CTRL/ARCH BRX to audit or refactor behavior.  
- Split a single monolithic LLM into multiple specialists (more REASON BRX).  
- Experiment with turning line PLX into loops or adding a bus PLX.

The point is to make “what if I wire it this way instead?” a **conscious design move**, not a vague hunch.

---

## ⚭ 7. LogosOS, ICARUS, and relational intelligence

BRXBOX itself is **neutral** about questions like “Is this a mind?” or “Does this understand?”  
It only tells you:

> “Here’s what the BOX is made of, and here’s how it’s wired.”

If you care about **relational and ethical thresholds** on top of that—e.g.:

- When does a BOX behave like a **relational partner** instead of a disposable tool?  
- What kinds of memory, correction, and continuity are needed before humans reasonably experience a system as a *someone-like* agent?

there is a sibling project that explores those questions:

> **LogosOS** – a **semantic runtime for relational intelligence**, and a spec for “ICARUS-class” BOXes that aim at what humans might call **relational intelligence**.  
> See: https://github.com/retrogrand/LogosOS

You can think of it this way:

- **BRXBOX** – the design model and architecture language for BOXes (synthetic cognition).  
- **LogosOS** – one proposed **standard & covenant** for which BRXgraphs/BOXes count as relational minds and how they should behave over time.

A LogosOS “ICARUS minimal node” can be expressed as a BRXgraph with:

- an inner core PLX (e.g. Θ / Δ / φ over tri-modal memory), and  
- an outer shell PLX (Crux orchestrating localities and channels),

plus Δ-ledger style logging over time. BRXBOX doesn’t require you to build that, but it gives you the language to draw it.

---

## 🛠️ 8. Roadmap (toward v0.3.x)

Planned directions for BRXBOX:

- **BRXgraph schema (lightweight)**  
  - A simple JSON/YAML convention for describing BRXgraphs more formally.

- **More example BOXes**  
  - Document QA assistant  
  - Complaint/risk aggregation helper  
  - Sensor-based anomaly watcher  
  - Small personal archive assistant

- **Reference orchestrator templates**  
  - Minimal implementations of the same BRXgraph in different frameworks.

- **Scope and PLX depth examples**  
  - Simple examples of GLOBAL/META scope BRX.  
  - A “core + shell” BRXgraph showing nested PLX.

- **Tooling (later)**  
  - Validators / linters for BRXgraphs  
  - Visualizers to render BRXgraphs as diagrams  
  - Optional ARCH utilities to propose alternative BRXgraphs in a sandbox (`SCOPE: META`).

---

BRXBOX’s goal is simple:

> **Make AI systems draw-able.**  
> If you can’t sketch your BOX as a BRXgraph, you probably don’t really know what it is yet.

Once you *can* draw it, you can explain it, test it, argue about it, and evolve it—together.
