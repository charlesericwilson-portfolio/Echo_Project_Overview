# Echo Project Journey

This repo serves as the central hub and origin story for all Echo-related projects.

### Why Echo Was Created

Most AI agents today are either cloud-dependent, bloated with JSON schemas, or fall apart during long-running CLI tasks. I wanted something better: a **local, lightweight, natural-text agent** that can actually drive real CLI workflows (pentesting, sysadmin, DevOps, or anything else) without needing a datacenter or burning money on API calls.

The entire stack runs on a **single consumer GPU** (~20 GB VRAM, 50k context). No cloud. No JSON overhead. Just a capable local partner.

I wanted a **local, capable, red-team-focused, general use AI** that could actually help me with real pentesting and system tasks, not just talk about them.

So I started with a strong base model ([**Qwen 2.5 Coder 14B Instruct**](https://huggingface.co/Qwen/Qwen2.5-Coder-14B-Instruct)) and fine-tuned it using [**QLoRA**](https://github.com/charlesericwilson-portfolio/Echo_projectv0/blob/main/unsloth_train_gpt.py) on a custom dataset. Check it out here [Echo_training_project](https://github.com/charlesericwilson-portfolio/Echo_training_project)

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
- **[v3 - Rust tmux sessions](https://github.com/charlesericwilson-portfolio/Echo_tmux_agentv3)**: The current active version using tmux for reliable persistent sessions and cleaner output capture in testing.
- **[v4 - Echo_python proxy](https://github.com/charlesericwilson-portfolio/Echo_agent_proxyv4)**: A much more complex version with persistent tmux sessions, heartbeat monitor, database, and summarizer in development.
- **[v5 - Echo rust agent proxy](https://github.com/charlesericwilson-portfolio/Echo_rust_agent_proxy)**: The next evolution rust + tmux version of v4 in development.

All of these projects exist for one reason: to give **Echo** (the fine-tuned LLM) the practical ability to use tools effectively and maintain state across sessions in a red-team-friendly or everyday use way.

### Current Focus

The latest development is in Rust because it runs significantly faster and gives better control and reliability. The goal is a clean, modular, and fast agent system that lets Echo do what it was trained for: real red team tasks, system interaction, and complex agent workflows.

This journey started from a simple desire: build an AI that doesn't just talk about red teaming — it actually helps *do* it.

Feel free to explore the individual repos for code, technical details, and lessons learned along the way.

Next focus remains on collecting high-quality SESSION vs COMMAND training examples from the current v4, v5 logs for a targeted LoRA.

Repo remains public and honest about its "in-testing" status.

## Technical Highlights & Skills Demonstrated

- **End-to-end agent architecture** — From simple wrapper to modular proxy with persistent sessions, heartbeat monitoring, and intelligent summarization
- **Custom model fine-tuning** — Fine-tuned Qwen2.5-Coder 14B using Unsloth + custom randomized interleaved datasets (better generalization than structured data)
- **Systems programming** — Rust for performance-critical components + Python for rapid prototyping
- **Practical LLM engineering** — Raw-text tool calling, domain-specific LoRA training, training data pipelines, and prompt engineering for reliable agent behavior
- **Real-world constraints** — Designed to run entirely locally with low VRAM footprint while maintaining capability

Early interest is strongest in the simpler raw-text COMMAND and tmux-based versions. v5 is gaining traction quickly. We will be adding databases to v5 next then going to integrate lessons learned to the v4 python version soon.

Future plans creating MLP routing for a Mixture of Adapters architecture.

Like Cameron Haynes says "Keep Hammering"

All of this was done with collaboration from Grok by XAI
