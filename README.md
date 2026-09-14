# Echo Project Journey

**A documented progression from a simple local LLM command wrapper to a trained model + Rust agent runtime + local inference stack.**

This repository is the central hub for the Echo / ADAPT project family.

The project started with a simple question:

> Can a local language model do more than explain how to use tools — can it actually use the computer, maintain long-running sessions, recover from failures, and complete real workflows without depending on cloud APIs?

What followed was a series of experiments across Python, Rust, tmux, FastAPI, SQLite, model fine-tuning, custom datasets, semantic memory, tool protocols, inference servers, and persistent model state.

Not every architecture worked.

Some became too complex.  
Some failed on long-running commands.  
Some exposed weaknesses in model training.  
Some worked well enough to become the foundation for the next version.

This repository exists to document that entire progression honestly.

---

# What Echo and ADAPT Are

The project eventually separated into two main pieces:

### Echo

**Echo is the model.**

Echo began as a Qwen2.5-Coder-14B-Instruct fine-tune and evolved through multiple training datasets and model experiments.

The training work focuses on:

- tool use
- multi-step workflows
- failure recovery
- CLI reasoning
- persistent sessions
- agent protocol behavior
- model identity and personality
- context management
- practical local execution

Training experiments are documented here:

[Echo Training Project](https://github.com/charlesericwilson-portfolio/Echo_training_project)

---

### ADAPT

**ADAPT is the runtime between the model and the computer.**

Instead of turning every shell command into a large JSON function schema, ADAPT lets the model communicate through a compact text protocol.

Current examples include:

```text
<command>whoami</command>
```

```text
<session name="metasploit">msfconsole</session>
```

```text
<json>
{
  "name": "browse_page",
  "arguments": {
    "url": "https://example.com"
  }
}
</json>
```

```text
<cleanup/>
```

The runtime parses the model's request, executes the action, returns the result as a tool message, and lets the model continue reasoning.

The operating system remains the primary tool ecosystem.

---

# Why I Built It

Most early agent frameworks I experimented with had one or more problems:

- cloud dependency
- large tool schemas
- fragile long-running terminal support
- poor handling of interactive tools
- tool output incorrectly treated as new user input
- complicated orchestration layers
- high API cost
- poor support for locally trained models

I wanted something much simpler:

> **Model → intent → runtime → real tool → result → model**

The goal was not to build another chatbot.

The goal was to build a local AI system that could actually work.

---

# Project Evolution

## v1 — Python Command Wrapper

[Echo Agent v1-2](https://github.com/charlesericwilson-portfolio/Echo_agentv1-2/tree/main/Echo_project/python_wrapper)

The first version was intentionally simple.

The model emitted:

```text
COMMAND: nmap -sV 192.168.1.1
```

The wrapper detected the command, executed it, and returned the output to the model.

This proved the basic idea:

> A language model does not need thousands of predefined tools if it already knows how to use a shell.

### What worked

- local inference
- simple shell execution
- low implementation complexity
- easy debugging
- direct model-to-CLI interaction

### What I learned

The simplest possible implementation is often the best place to begin.

It also exposed the first important lesson:

> Tool use is easy. Reliable long-running tool use is much harder.

A one-shot command such as `whoami` is simple.

Interactive programs such as:

- `msfconsole`
- database shells
- REPLs
- SSH sessions
- long-running scans

require persistent state.

That led to v2 and v3.

---

# v2 — Rust Port

The first wrapper was rewritten in Rust.

The protocol remained simple, but Rust provided:

- lower runtime overhead
- predictable execution
- stronger process control
- better suitability for a long-running local runtime

### Lesson learned

Rewriting the system in another language did not solve architectural problems by itself.

Rust made the wrapper faster and cleaner, but interactive programs still required persistent sessions.

That led directly to tmux integration.

---

# v3 — Persistent tmux Sessions

[Echo ADAPT v3](https://github.com/charlesericwilson-portfolio/Echo_Adapt_v3)

v3 introduced named persistent sessions.

Example:

```text
SESSION:metasploit msfconsole
```

Instead of treating every command as an isolated process, the model could return to an existing terminal session and continue interacting with it.

### What worked

- persistent named sessions
- interactive CLI applications
- command history inside the session
- reusable environments
- local execution
- simple text protocol

### Major lesson

**Persistent processes are easy to start and difficult to know when they are finished.**

A shell command may finish immediately.

A network scan may run for minutes.

An interactive application may never technically finish.

The runtime needed a reliable way to determine:

> When should output be returned to the model?

That problem became the focus of v4.

---

# v4 — Agent Proxy Experiment

[Echo Agent v4](https://github.com/charlesericwilson-portfolio/Echo_Agentv4)

v4 was the most complex early architecture.

It experimented with:

- FastAPI orchestration
- persistent tmux sessions
- SQLite state
- heartbeat monitoring
- PTY interaction
- tool-output summarization
- a separately trained small summarizer model
- session databases
- audit logs

The idea was to monitor output continuously and determine when a command had stopped producing useful information.

### What worked

- persistent sessions
- database-backed state
- tool-output monitoring
- command execution
- summarization
- session reuse

### What did not work well

Long-running and interactive commands were difficult to classify reliably.

Examples included:

- full network scans
- Metasploit modules
- commands with long silent periods
- commands that produced intermittent output

The heartbeat system could decide a task was complete too early.

PTY handling also created additional complexity.

### Major lesson

> **Adding infrastructure does not necessarily make an agent more reliable.**

v4 taught me one of the most important lessons in the entire project:

**sometimes the solution is to remove machinery.**

Instead of trying to infer everything from background monitoring, later versions moved toward simpler deterministic interaction patterns.

---

# v5 — ADAPT Rust Runtime

[Echo ADAPT v5](https://github.com/charlesericwilson-portfolio/Echo_Adapt_v5)

v5 became the current architecture.

The project moved back toward the simplicity of v1 while keeping what had been learned from v3 and v4.

The model now communicates through a small protocol supporting:

- one-shot commands
- persistent sessions
- structured JSON tools
- cleanup
- memory
- web search
- page browsing
- context management

Examples:

```text
<command>...</command>
```

```text
<session name="...">...</session>
```

```text
<end_session name="..."/>
```

```text
<json>...</json>
```

```text
<cleanup/>
```

### Important architectural rule

The model produces **one actionable tool request per assistant turn**.

The runtime:

1. stores the assistant response
2. parses the current turn
3. executes the final valid tool request
4. returns the result
5. allows the model to continue

This keeps tool execution deterministic.

### Major lessons from v5

#### Keep execution deterministic

Allowing many independent tool requests inside one generation makes ordering and failure recovery harder.

One action per turn produces a simple loop:

```text
assistant
→ tool
→ result
→ assistant
```

---

#### The model should see the same protocol during training and inference

Tool tags remain in the assistant history.

This means the model's live context resembles the data it was trained on.

The terminal can display a cleaned version while the model retains the original protocol structure.

---

#### Tool output should not masquerade as user input

One of the most important discoveries came from model training.

Many chat templates historically supported only:

- system
- user
- assistant

If tool output is inserted as a **user message**, the model may interpret it as another instruction.

This can create loops:

```text
assistant calls tool
→ result inserted as user
→ model thinks user issued another request
→ model calls another tool
→ repeat
```

Echo's training template was modified to support a true **tool role**.

The model can therefore distinguish:

```text
user instruction
```

from:

```text
result of my previous action
```

That greatly improves multi-step workflow behavior.

---

#### The operating system already contains thousands of tools

A model does not necessarily need thousands of JSON function definitions.

If the model understands Linux and CLI tools, the runtime can expose the operating system itself.

This keeps the framework small while still allowing access to:

- networking tools
- file utilities
- compilers
- scripting languages
- databases
- package managers
- developer tools
- security tools
- anything installed on the system

Structured JSON tools are still useful where fixed parameters make sense.

---

#### Safety belongs at the execution boundary

ADAPT does not depend entirely on the model choosing safe actions.

The runtime includes controls such as:

- command filtering
- workspace restrictions
- logging
- human-review paths
- controlled tool execution
- cleanup boundaries

The model can remain capable while the runtime controls what can actually happen.

---

# Model Training Journey

[Echo Training Project](https://github.com/charlesericwilson-portfolio/Echo_training_project)

The runtime and the model evolved together.

Early experiments used QLoRA and Unsloth to adapt Qwen-based models for:

- tool selection
- shell workflows
- red-team reasoning
- persistent sessions
- model personality
- multi-step tasks

Training produced several important lessons.

---

## Data order matters

Early structured datasets plateaued earlier than expected.

Randomized interleaving of related examples produced better results.

The same underlying information presented in different sequences reduced simple pattern memorization and improved behavior during testing.

### Lesson

> Dataset structure can matter as much as dataset size.

---

## Full workflows are more useful than isolated instructions

Later datasets moved toward multi-turn workflows containing:

```text
user request
→ assistant tool call
→ tool output
→ assistant reasoning
→ another tool call
→ result
→ completion
```

This teaches the model the entire interaction process instead of merely teaching individual commands.

The datasets also include failure cases.

The model can therefore see:

```text
attempt
→ error
→ inspect result
→ change strategy
→ retry
→ success
```

### Lesson

A model should not only be trained on perfect answers.

It should see how mistakes are recognized and corrected.

---

## Failure examples are training data

One of the recurring themes across Echo development has been preserving failures instead of hiding them.

Tool errors, incorrect assumptions, failed commands, and recovery attempts are useful because they teach the model what to do when reality disagrees with its expectation.

This eventually became a broader research interest:

> Can models learn more robust behavior by training on complete failure-to-success trajectories rather than only polished final answers?

---

## Training the protocol reduces prompt dependency

Instead of describing ADAPT's entire tool protocol in a giant system prompt, the model can learn the protocol directly from training examples.

The goal is similar to how a programming model learns JSON or Python syntax:

> tool behavior becomes part of the model's learned distribution rather than something explained from scratch every session.

---

# ADAPT Server

The latest architecture also introduced a dedicated local inference server.

The purpose is deliberately narrow:

> **Inference should remain inference.**

Many inference servers increasingly include:

- tool parsers
- function-call conversion
- model-specific reasoning handling
- agent semantics

That can conflict with an external runtime such as ADAPT.

ADAPT Server therefore provides a simpler contract:

```text
messages in
→ model-native GGUF template
→ inference
→ minimal reasoning/control-token normalization
→ assistant text out
```

ADAPT itself continues to own:

- tool interpretation
- state
- memory
- permissions
- execution
- orchestration

The server currently supports:

- GGUF models
- llama.cpp backend
- AMD ROCm
- NVIDIA CUDA
- CPU inference
- multi-GPU model sharding
- model-native chat templates
- configurable context length
- OpenAI-compatible chat endpoint
- reasoning-channel cleanup
- a minimal built-in browser chat interface

The built-in chat interface exists only for directly testing a loaded model.

It intentionally does not attempt to become another agent frontend.

---

# Web Retrieval Lessons

ADAPT includes web search and page browsing.

An important distinction emerged during development:

### Search is not browsing

Search should return:

- titles
- URLs
- snippets

Browsing should retrieve and expose the actual page.

The `browse_page` tool was updated to return readable page content rather than raw web structure.

The model receives:

- headings
- paragraphs
- lists
- code
- tables
- links
- resolved URLs

while avoiding unnecessary:

- HTML scaffolding
- CSS
- JavaScript
- browser chrome

### Lesson

> Give the model information, not website implementation details.

The model can then follow links and continue researching without wasting context on page structure.

---

# Memory and Context

Long-lived agents eventually encounter a second problem:

**conversation history becomes too large to keep everything active forever.**

ADAPT experiments include:

- persistent memory
- semantic search
- cross-session recall
- context summarization
- archived task state

The goal is to retrieve relevant information rather than continuously replaying the entire conversation.

This work eventually led to additional experiments around temporal context and persistent experience.

---

# PES — Persistent Experience State

PES explores whether an agent can maintain a compact evolving representation of experience across interactions.

The idea grew out of a simple observation:

A model has many individual interactions, but traditional inference treats those interactions as disconnected unless the complete history is repeatedly provided.

PES experiments explore ways to represent:

- recent experience
- task history
- context
- changing state
- persistent behavioral information

without replaying every prior token.

This remains experimental research rather than a finished component.

---

# Mixture of Adapters Experiment

[MoAdapters](https://github.com/charlesericwilson-portfolio/mixture_of_adapters)

Another experimental branch explored routing prompts between specialized LoRA adapters.

A small MLP router trained on model embeddings attempted to select between domain-specific adapters.

The project demonstrated that adapter routing is possible but also exposed an important weakness:

> Routing quality matters as much as adapter quality.

A strong adapter selected for the wrong task can perform worse than a weaker general model.

This remains an area I plan to revisit.

---

# What Changed Across the Project

The project gradually moved from:

```text
large orchestration layers
```

toward:

```text
small deterministic runtime
```

From:

```text
tool behavior defined entirely by prompts
```

toward:

```text
tool protocol learned by the model
```

From:

```text
isolated commands
```

toward:

```text
persistent interactive sessions
```

From:

```text
perfect training examples
```

toward:

```text
failure → recovery → success trajectories
```

From:

```text
external inference servers defining agent behavior
```

toward:

```text
inference server = inference
agent runtime = agent semantics
```

And from:

```text
single experiment
```

toward:

```text
model + runtime + inference + memory + training stack
```

---

# Technical Skills Demonstrated

This project family has become a practical laboratory for learning and applying:

### LLM Engineering

- QLoRA / LoRA fine-tuning
- Unsloth
- Hugging Face training
- long-context training
- custom datasets
- chat-template modification
- first-class tool roles
- workflow datasets
- failure-recovery training
- model evaluation
- overfitting analysis

### Local Inference

- GGUF
- llama.cpp
- multi-GPU inference
- AMD ROCm
- NVIDIA CUDA
- context configuration
- native chat templates
- reasoning-channel normalization

### Systems Engineering

- Rust
- Python
- async execution
- process management
- persistent tmux sessions
- CLI integration
- background task handling
- SQLite
- structured logging

### Agent Architecture

- deterministic tool loops
- raw-text tool protocols
- JSON tools
- persistent sessions
- memory
- semantic retrieval
- context summarization
- failure recovery
- safety boundaries
- human-review workflows

### Research / Experimentation

- dataset ordering experiments
- protocol learning
- adapter routing
- persistent model state
- experience-based training
- local model behavior analysis

---

# The Biggest Lessons

After multiple versions, the biggest lessons were surprisingly simple.

### 1. Simpler systems are easier to make reliable

The most complicated architecture was not the best architecture.

---

### 2. The model should understand the protocol

Prompting alone can work, but training the interaction pattern produces more natural behavior.

---

### 3. Tool results are not user messages

Message semantics matter.

---

### 4. Failure is valuable data

A failed attempt followed by successful recovery may teach more than another perfect example.

---

### 5. Long-running tools require persistent state

Real computer work cannot always be represented as isolated function calls.

---

### 6. The operating system is already a tool framework

Do not recreate every existing command as a custom API unless there is a reason.

---

### 7. Inference and agent semantics should remain separate

The model server should generate model output.

The runtime should decide what that output means.

---

### 8. Local models are much more capable than they are often given credit for

A carefully trained 14B model with the right runtime and dataset can complete surprisingly complex multi-step workflows locally.

---

### 9. Dataset quality matters more than raw dataset size

Repeated high-quality workflows with realistic failures and outcomes can provide an extremely strong training signal.

---

### 10. Build, break, understand, simplify, repeat

That became the development process for the entire project.

---

# Current Architecture

At a high level:

```text
                    ┌─────────────────────┐
                    │        Echo         │
                    │  Fine-tuned Local   │
                    │        LLM          │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    ADAPT Server     │
                    │       GGUF          │
                    │ llama.cpp inference │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │        ADAPT        │
                    │    Rust Runtime     │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
        CLI Commands     tmux Sessions      JSON Tools
              │                │                │
              └────────────────┼────────────────┘
                               ▼
                        Tool Results
                               │
                               ▼
                             Echo
```

---

# Why This Repository Exists

This repository is not intended to pretend the project appeared fully formed.

It did not.

The progression matters.

I started with a small Python wrapper because I wanted a local model to execute commands.

That exposed persistent-session problems.

Persistent sessions exposed output-completion problems.

Trying to solve those problems produced an overly complex architecture.

That complexity eventually pushed the project back toward a smaller deterministic Rust runtime.

Training the model exposed message-role problems.

Message-role problems led to custom tool semantics.

Inference-server incompatibilities eventually led to a dedicated lightweight local server.

Every version exists because the previous version taught me something.

That process is the actual project.

---

# Current Development

The current work is centered around:

- ADAPT v5
- local GGUF inference
- model/runtime integration
- improved web retrieval
- long-context local models
- workflow training
- failure-recovery datasets
- persistent memory/state experiments
- improving agent reliability without adding unnecessary architecture

The project remains experimental, open source, and intentionally transparent about both successes and failures.

---

# Project Links

- [Echo Training Project](https://github.com/charlesericwilson-portfolio/Echo_training_project)
- [Echo Agent v1-2](https://github.com/charlesericwilson-portfolio/Echo_agentv1-2)
- [Echo ADAPT v3](https://github.com/charlesericwilson-portfolio/Echo_Adapt_v3)
- [Echo Agent v4](https://github.com/charlesericwilson-portfolio/Echo_Agentv4)
- [Echo ADAPT v5](https://github.com/charlesericwilson-portfolio/Echo_Adapt_v5)
- [MoAdapters](https://github.com/charlesericwilson-portfolio/mixture_of_adapters)

---

## About the Project

This project began as a cybersecurity / AI learning project and grew into a broader exploration of local AI systems, model training, tool use, persistent state, and agent architecture.

It has been built through extensive experimentation with AI-assisted development, including collaboration with models from multiple AI providers.

The architecture, implementation choices, experiments, failures, and final decisions remain my responsibility.

— Charles (Eric) Wilson

> Build it. Break it. Understand why. Make the next version better.
