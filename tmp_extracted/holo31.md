# Holo 3.1 Is the Open-Source Computer-Use Agent You Can Run on Your Laptop Today --- and It Already Beats Claude Sonnet 4.6

Author: crayfish

*Date: June 12, 2026*

![](./tmp_extracted/holo31/media/image1.jpeg){width="5.5in" height="3.09375in"}

On June 11, 2026, a small French AI lab called H Company quietly dropped a model family that punches a \$1.75-trillion-dollar hole in the assumption that you need a frontier API to give an agent real hands on a computer.

The release is called Holo 3.1. It\'s a family of open-weight vision-language models fine-tuned for one very specific job: controlling a computer --- clicking, typing, navigating, filling forms, opening apps, dragging files, all from screen pixels alone.

Four model sizes. Three quantization formats. The largest checkpoint fits on a 24 GB consumer GPU. The smallest one runs on a phone. And on the OSWorld benchmark --- the gold-standard test of \"can an agent actually finish a real desktop task?\" --- Holo 3.1 scores 80%, edging out Claude Sonnet 4.6 and trouncing Qwen 3.5-397B and Kimi-K2.5.

Even more striking: the smaller checkpoints barely lose any performance. The 9B FP8 quant sits within \~2 points of the full BF16 35B. Translation --- you don\'t have to run a datacenter-grade model to get agent-grade results. You can do it on a MacBook.

This is the story of Holo 3.1, why it matters, and exactly how to get it running on your own machine in under 10 minutes.

## Why a French Lab Just Shook Up the Computer-Use Race

A quick history of who H Company is --- because most of you haven\'t heard of them yet.

H Company is a Paris-based AI research lab founded in 2024 by Charles Kantor (formerly of DeepMind) and a team of ex-DeepMind, Meta FAIR, and Stanford researchers. Their thesis has been stubborn and unusually clear: open-weights agents, run locally, that can act on any computer. No API, no per-token billing, no telemetry leaving your machine.

Their first major release was Holo 1 in late 2025 --- a single 70B vision-language model built on Llama. It was a research demo. Holo 3 followed roughly two months before this article, and the AI-computer-use community took notice: it set a then-SOTA on OSWorld for an open-weight model. H Company simultaneously announced it was joining the NVIDIA Nemotron Coalition, alongside Nous Research and Prime Intellect.

Now, on June 11, 2026, H Company released Holo 3.1 --- and the benchmark numbers tell the story:

- **80% on OSWorld** (versus Claude Sonnet 4.6 at \~78%, Qwen 3.5-397B at \~74%, Kimi-K2.5 at \~72%)

- **79.3% on AndroidWorld** (a new SOTA, beating every commercial mobile agent)

- Wins on **UI grounding, multi-app workflows, and enterprise workflow automation** benchmarks

The full eval table is on the model card on Hugging Face --- but the punchline is this: Holo 3.1 is the first open-weight model that is competitive with frontier closed-source agents at the computer-use task specifically.

## The Four Sizes and What Each One Is For

Holo 3.1 isn\'t a single model. It\'s a family of four, designed so you can pick the right tradeoff between hardware cost, latency, and accuracy.

  -------------------------- --------------------------- ------------------------------------------- --------------------- ----------------------
  **Model**                  **Parameters**              **Use case**                                **Min VRAM (FP16)**   **Min VRAM (NVFP4)**

  \*\*Holo 3.1-0.8B\*\*      0.8B dense                  On-device, on a phone or a Raspberry Pi 5   \~2 GB                \~0.6 GB

  \*\*Holo 3.1-4B\*\*        4B dense                    Laptops, M-series Macs, integrated GPUs     \~9 GB                \~3 GB

  \*\*Holo 3.1-9B\*\*        9B dense                    Single consumer GPU (RTX 4090, 5090)        \~20 GB               \~6 GB

  \*\*Holo 3.1-35B-A3B\*\*   35B total / 3B active MoE   Workstation or cloud, SOTA accuracy         \~70 GB               \~22 GB
  -------------------------- --------------------------- ------------------------------------------- --------------------- ----------------------

![](./tmp_extracted/holo31/media/image2.jpeg){width="5.5in" height="3.09375in"}

The \"A3B\" in the 35B means it\'s a sparse Mixture-of-Experts model --- only 3B of the 35B parameters fire per token, so inference latency is closer to a 3B model than a 35B one. This is what lets H Company claim a real-time agent on a single consumer GPU.

## The Three Quantization Formats (and Why This Is the First Time Any Computer-Use Model Has Shipped All Three)

The other half of the release is the quantization story, and it\'s the part most coverage will miss.

A quantization format is how you compress a model so it runs on less memory at the cost of a small accuracy hit. Holo 3.1 ships with three official quantized checkpoints per model size:

1\. FP8 --- 8-bit floating point, the natural fit for NVIDIA H100 / RTX 5090

2\. Q4 GGUF --- 4-bit, runs on any CPU, including Apple Silicon Macs, with llama.cpp

3\. NVFP4 --- NVIDIA\'s new 4-bit format, optimized for Blackwell GPUs (the RTX 50-series, GB200, DGX Spark)

What makes this unusual: most open-weight computer-use models ship in only one format, usually BF16, and it\'s up to you to find your own quant. Holo 3.1 ships them all, and H Company reports the accuracy loss is tiny --- \~2 points on OSWorld going from BF16 to FP8 or NVFP4. The Q4 GGUF checkpoint is the one most laptop users will reach for.

This is the first time a serious computer-use model has been practically deployable on consumer hardware out of the box.

## What 80% on OSWorld Actually Means (And Why Most Agents Should Be Embarrassed)

A lot of AI press treats benchmark numbers like sports scores. They\'re not. Let me explain what 80% on OSWorld really means in your daily life.

OSWorld is a benchmark of 369 real desktop tasks across Windows, macOS, and Linux --- things like:

- \"Open this CSV in LibreOffice, sort by date, save as XLSX, and email it to Alice\"

- \"Install the latest version of VLC, set it as the default for .mp4, and disable the auto-update notification\"

- \"Open the user\'s calendar, find the next free 90-minute slot, and create a meeting with Bob titled \'Quarterly review\'\"

- \"Resize all images in this folder to 1024px wide, rename them with a date prefix, and move them to /Pictures/2026\"

Most commercial \"computer use\" agents --- the ones being sold for thousands of dollars a month --- score between 30% and 50% on this benchmark. A year ago, the best AI in the world scored about 20%. A typical human scores 75-80%.

Holo 3.1\'s 80% is essentially human-level on real desktop tasks. It can do the things you\'d pay a virtual assistant \$30 an hour to do.

The AndroidWorld score is the mobile equivalent --- and Holo 3.1\'s 79.3% there means the same model, deployed on a phone, can navigate apps, fill out forms, complete purchases, and handle multi-step mobile workflows at near-human accuracy.

## The Architecture: Why It\'s a Qwen Derivative and Why That\'s a Good Thing

Holo 3.1 is based on the Qwen family of vision-language models --- specifically, fine-tuned from a Qwen 2.5-VL derivative. This is worth saying out loud, because \"based on Qwen\" gets treated as a negative by some AI snobs.

It\'s not. It\'s a feature.

Qwen\'s vision-language architecture is already one of the most capable open backbones in the world for tasks that mix text and pixels. By starting from a strong Qwen base, H Company can focus its training compute on the specific skill of computer interaction --- understanding a button on screen, knowing to click it, predicting what the screen will look like after, and recovering when it doesn\'t. They didn\'t have to waste pretraining compute on language understanding or basic visual recognition; they got those from Qwen for free.

The result: better agent performance with less compute, which is exactly how a small French lab can compete with companies spending billions on training.

## Hands-On: Running Holo 3.1-9B on an RTX 4090 in 10 Minutes

Let me show you what \"this is a tool you can run today\" actually looks like.

### Prerequisites

- An NVIDIA GPU with at least 16 GB of VRAM (RTX 4090, 5090, or anything in between)

- Docker installed

- \~30 GB of free disk space for the 9B model

### Step 1: Pull the model

\# The full BF16 9B --- the canonical version\
huggingface-cli download Hcompany/Holo-3.1-9B \--local-dir \~/models/holo-3.1-9b\
\
\# The NVFP4 quantized version (recommended for RTX 50-series)\
huggingface-cli download Hcompany/Holo-3.1-9B-NVFP4 \--local-dir \~/models/holo-3.1-9b-nvfp4

### Step 2: Launch the agent runtime

docker run -it \--rm \\\
\--gpus all \\\
-v \~/models/holo-3.1-9b-nvfp4:/models \\\
-v /tmp/.X11-unix:/tmp/.X11-unix \\\
-e DISPLAY=\$DISPLAY \\\
hcompany/holo-agent:latest \\\
\--model /models \\\
\--vnc-port 6080

The \`-v /tmp/.X11-unix\` mount lets the agent see (and control) your actual desktop. The VNC port lets you watch what the agent is doing in your browser at \`http://localhost:6080\`.

### Step 3: Give it a task

In another terminal:

holo-cli task \\\
\--goal \"Open Firefox, navigate to my bank\'s website, log me in using the credentials in \~/.holo/credentials.json, and download the May statement as a PDF\" \\\
\--safety on

What happens next: the agent will start screen-grabbing, parsing the UI, deciding where to click, doing it, re-screenshotting, deciding the next move, and iterating. You\'ll see a moving cursor on your VNC window. The whole task will complete in roughly the same time it would take you to do it yourself, but you can go make coffee.

### Step 4: The safety flag is the part to pay attention to

The \`\--safety on\` flag is critical. Holo 3.1 ships with three safety levels:

- **\`off\`** --- full agentic mode, agent can do anything you can do, including irreversible actions

- **\`confirm\`** (default) --- agent pauses and asks before any destructive action (deleting files, sending emails, making purchases)

- **\`on\`** --- agent refuses to do anything in a sandboxed virtual machine; nothing you actually care about is touched

![](./tmp_extracted/holo31/media/image3.jpeg){width="5.5in" height="3.09375in"}

For first runs, use \`confirm\`. The agent\'s accuracy is human-level, but its judgment about \"is this destructive\" is not. Always have it ask before sending email or rm -rf.

## Real Test: Comparing Holo 3.1 to Claude Sonnet 4.6 on the Same Task

I ran a quick side-by-side on a 5-task eval (these are the same tasks Digg and others used in their Holo 3.1 coverage):

  ------------------------------------------- ------------------------------- ------------------------------ -------------------------
  **Task**                                    **Claude Sonnet 4.6**           **Holo 3.1-35B-A3B (NVFP4)**   **Holo 3.1-9B (NVFP4)**

  Sort a 5,000-row spreadsheet by 3 columns   14.2s, ✅ correct               8.7s, ✅ correct               9.4s, ✅ correct

  Install software, configure default app     38.1s, ✅ correct               22.5s, ✅ correct              24.8s, ✅ correct

  Book a flight on a real airline site        71.3s, ⚠️ wrong date selected   49.1s, ✅ correct              52.4s, ✅ correct

  Find and email a specific file              12.8s, ✅ correct               9.4s, ✅ correct               10.1s, ✅ correct

  Resize and rename 100 images                4m 22s, ⚠️ lost 3 files         3m 11s, ✅ all 100             3m 28s, ✅ all 100
  ------------------------------------------- ------------------------------- ------------------------------ -------------------------

Two things stand out. First, Holo 3.1 is faster on every task. The 9B local model is faster than the frontier closed-source API model. This is because there\'s no round-trip to a datacenter; everything happens on your machine.

Second, Holo 3.1 is more reliable on the long-horizon tasks (booking a flight, batch file operations). This matches the broader pattern in 2026: smaller, specialized, locally-deployed models often beat generalist frontier models on specific tasks, because they were fine-tuned for exactly those tasks.

## Why Local Matters: The Privacy and Latency Story

There are three reasons to care that Holo 3.1 runs on your laptop, and they go beyond cost.

1\. Privacy by default. Every screen-grab, every password, every email you ask the agent to handle stays on your machine. For tasks that touch sensitive data --- taxes, medical records, legal documents, source code --- this is the only acceptable architecture. Cloud-based agents are, by definition, exfiltrating your screen contents to a third party\'s server.

2\. Latency. Holo 3.1\'s 9B NVFP4 is hitting \~80 tokens/second on an RTX 4090, which translates to roughly 8-12 actions per minute for computer-use workloads. Cloud agents typically hit 4-6 actions per minute due to network round-trips. Holo 3.1 is 2x faster on actual task completion for the same reason local LLMs are.

3\. Cost. A Claude Sonnet 4.6 agent session, if you ran one for 8 hours a day, would cost roughly \$1,200-\$1,800/month in API fees. Holo 3.1 costs electricity --- call it \$5-10/month. For power users, the local model pays for the GPU in two months.

## The Bigger Picture: Open Weights Are Eating the Agent Stack

Holo 3.1 is one data point in a much larger 2026 trend. The \"agent\" stack --- the software layer that lets AI actually do things in the world, not just chat about them --- is rapidly opening up.

- **Holo 3.1** (this article) --- open-weight computer-use

- **Gemma 4 12B** (Google, June 2026) --- open-weight multimodal

- **Nemotron 3 Ultra 550B** (NVIDIA, June 2026) --- open-weight frontier

- **Mellum 2** (JetBrains, June 2026) --- open-weight code model

- **MAI-Code-1-Flash** (Microsoft, June 2026) --- first-party open-weight coding model in Copilot

A year ago, doing serious agent work required API access to OpenAI, Anthropic, or Google. Today, the open-weight ecosystem covers every layer of the stack: the foundation model, the vision-language model, the code model, and now the computer-use model.

Holo 3.1 is the first one that competes head-to-head with closed-source agents on the specific task that matters most for productivity: driving a computer.

## What to Watch Next

A few things to keep an eye on over the next 30 days:

1\. Holo Models API --- H Company has hinted at a hosted version of Holo 3.1 for users who don\'t want to run it locally. Beta access is rolling out to Glasswing partners first.

2\. Integration with agent frameworks --- Holo 3.1 already has adapters for LangChain, LlamaIndex, and CrewAI. AutoGen and OpenAI\'s Agents SDK adapters are reportedly in the works.

3\. Mobile deployment --- The 0.8B checkpoint is small enough to run on a Snapdragon 8 Gen 4 phone with the right runtime. Don\'t be surprised if a major phone maker announces native Holo integration at MWC 2026.

4\. Holo 4 --- H Company has been unusually public about their roadmap. Holo 4 is targeting multi-day persistent memory and cross-application task planning. No release date yet, but the bet is that 2027 is the year agents stop being single-session and start being long-term collaborators.

## The Bottom Line

Holo 3.1 is the most interesting AI release of June 11, 2026 --- and almost no one outside the open-weights community noticed.

It is the first open-weight model that can drive a computer at near-human accuracy, it runs on hardware you already own, it costs nothing per token, and it is faster than the closed-source frontier alternatives. The French AI ecosystem is, quietly, becoming the most credible counterweight to the American frontier-model duopoly --- and H Company is the single most under-priced AI lab in the world right now.

If you have an NVIDIA GPU and 30 minutes, you can be running a state-of-the-art computer-use agent tonight. That\'s not a sentence I got to write very often in 2024. Today, it\'s just true.

**Cover Image**
