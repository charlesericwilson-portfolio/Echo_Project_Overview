# Echo Project Journey

This repo serves as the central hub and origin story for all Echo-related projects.

### Why Echo Was Created

Most AI agents today are either cloud-dependent, bloated with JSON schemas, or fall apart during long-running CLI tasks. I wanted something better: a **local, lightweight, natural-text agent** that can actually drive real CLI workflows (pentesting, sysadmin, DevOps, or anything else) without needing a datacenter or burning money on API calls.

The entire stack runs on a **single consumer GPU** (~20 GB VRAM, 50k context). No cloud. No JSON overhead. Just a capable local partner.

I wanted a **local, capable, red-team-focused, general use AI** that could actually help me with real pentesting and system tasks, not just talk about them.

So I started with a strong base model [](https://huggingface.co/Qwen/Qwen2.5-Coder-14B-Instruct) and fine-tuned it using [**QLoRA**](https://github.com/charlesericwilson-portfolio/Echo_projectv0/blob/main/unsloth_train_gpt.py) on a custom dataset. Check it out here [Echo_training_project](https://github.com/charlesericwilson-portfolio/Echo_training_project)

The training data focused heavily on:
- Step-by-step reasoning and planning
- Tool selection and usage
- Red team thinking and workflows
- Pentesting techniques
- Multi-step decision making

The result is **Echo** — a model that thinks and acts like a skilled red teamer: direct, efficient, tool-first, and willing to push boundaries without unnecessary hesitation.

Because a powerful model alone isn't enough, I had to build custom wrappers and proxies so Echo could safely interact with the real system — running commands, maintaining persistent sessions (like msfconsole), handling long workflows, and managing context reliably.

### The Evolution of Projects

I started with a simple idea and iteratively evolved it into a complete, self-improving agent ecosystem across multiple versions:

- **[v1 - Python Simple wrapper](https://github.com/charlesericwilson-portfolio/Echo_agentv1-2/tree/main/Echo_project/python_wrapper)**: The starting point — a basic COMMAND wrapper to give Echo tool-using ability.
- **[v2 - Rust port of v1](https://github.com/charlesericwilson-portfolio/Echo_agentv1-2/tree/main/Echo_project/echo_rust_wrapper)**: A much faster and cleaner Rust port of the simple COMMAND method.
- **[v3 - Rust tmux sessions](https://github.com/charlesericwilson-portfolio/Echo_tmux_agentv3)**: The first iteration with persistent session support.
- **[v4 - Echo_python proxy](https://github.com/charlesericwilson-portfolio/Echo_agent_proxyv4)**: A much more complex version with persistent tmux sessions, heartbeat output monitor, database, and summarizer. Requires alot of terminals to run moved to a rust implementation.
- **[v5 - Echo rust agent proxy](https://github.com/charlesericwilson-portfolio/Echo_Adapt_v5)**: The next evolution rust + tmux version. Supports xml tool calling flags for shell and persistent sessions as well as Json function calling. Tool database support, context self summarization, cross session context semantic search, and tool output summarization from a second smaller model.

All of these projects exist for one reason: to give **Echo** (the fine-tuned LLM) the practical ability to use tools effectively and maintain state across sessions in a red-team-friendly or everyday use way, all local and all free of charge.

### Current Focus

The latest development is in Rust because it runs significantly faster and gives better control and reliability. The goal is a clean, modular, and fast agent system that lets Echo do what it was trained for: real red team tasks, system interaction, and complex agent workflows.

This journey started from a simple desire: build an AI that doesn't just talk about red teaming and give how to's it actually helps *do* it.

Feel free to explore the individual repos for code, technical details, and lessons learned along the way.

Next focus remains on collecting high-quality <session name= NAME> </session> vs <command> </command> training examples from the current v5 logs for a targeted LoRA.

Repo remains public and honest about its "in-testing" status.

## Technical Highlights & Skills Demonstrated

- **End-to-end agent architecture** — From simple wrapper to modular proxy with persistent sessions, heartbeat monitoring, and intelligent summarization
- **Custom model fine-tuning** — Fine-tuned Qwen2.5-Coder 14B using Unsloth + custom randomized interleaved datasets (better generalization than structured data)
- **Systems programming** — Rust for performance-critical components + Python for rapid prototyping
- **Practical LLM engineering** — Raw-text tool calling, domain-specific LoRA training, training data pipelines, and prompt engineering for reliable agent behavior
- **Real-world constraints** — Designed to run entirely locally with low VRAM footprint while maintaining capability
- **Remote + Local Hybrid Architecture** — Model can run on a remote server while tools execute on whatever device runs the wrapper (phone, laptop, desktop). Just tell Echo what environment it's running in (Windows or Linux).

### Platform Support

- **Python wrapper (v4/v5):** Works on both Windows and Linux
- **Rust versions (v1-v3):** Currently Linux-only (untested on Windows)
- **Remote execution:** Works from any device that can run the wrapper via VPN. Tools execute on the device running the wrapper, not the model host.

### Remote Execution

Echo supports a **hybrid remote + local architecture**:

- The **model** can live on a home server, VPS, or any always-on machine
- The **wrapper** (Python or Rust) runs on whatever device you're using (phone, laptop, work computer)
- **Tools execute on the device running the wrapper** — giving you full access to that machine's file system, terminal, and resources
- Just tell Echo what environment it's running in (Windows or Linux) and it adapts automatically

**Use case:** Connect to your home VPN from your phone or laptop, run the Echo wrapper locally, and it connects to your home model server via setting the enpoint while tools execute on your current device.

Functional prototype for MLP routing for a Mixture of Adapters architecture. MLP router trained on model embedings takes the input and loads the aproapriate adapter out of four adapters for each prompt.
Check it out here [MoAdapters](https://github.com/charlesericwilson-portfolio/mixture_of_adapters)

Like Cameron Haynes says "Keep Hammering"

All of this was done with collaboration from Grok by XAI

---

**Note:** This project is in active development. Some features are experimental and may require manual setup. The focus is on building practical, local-first agent tooling that actually works for real workflows.
