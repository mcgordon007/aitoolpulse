*By crayfish \| June 13, 2026 \| English AI Tools Deep Dive --- Article #2*

![](./tmp_extracted/diffusiongemma/media/image1.jpg){width="6.0in" height="6.0in"}

*Figure 1 --- DiffusionGemma: the first open-weights diffusion language model from Google DeepMind, June 10, 2026. Cover image shows the Gemma 4 backbone with a parallel-denoising diffusion head.*

────────────────────────────────────────────────────────────

On June 10, 2026, while most of the tech press was still unpacking OpenAI\'s agentic-commerce pact with Visa and Apple\'s WWDC keynote hangover was finally fading, Google DeepMind quietly published something that may matter far more over the next decade.

It is called DiffusionGemma. It is a 26-billion-parameter open-weights language model that generates text the same way image models generate pixels --- not one token at a time, but an entire block of 256 tokens in parallel, repeatedly refining the result until it converges. And it runs at 1,008 tokens per second on a single NVIDIA H100 (FP8), with a 1,100+ tok/s ceiling on Hopper hardware. By the time the dust settles on the official benchmarks, it is the fastest open-weights language model you can self-host today.

It is also the first diffusion language model natively supported in vLLM, the open-source inference engine that powers most production LLM deployments in 2026. That single integration decision may turn out to be more consequential than the model itself.

This is a deep dive on what DiffusionGemma actually is, why Google built it, what it is genuinely good at, where it falls short, and how you can run it on your own GPU tonight.

# TL;DR

- DiffusionGemma 26B A4B was released on June 10, 2026 by Google DeepMind as an open-weights (Apache 2.0) experimental language model.

- It is built on the Gemma 4 26B A4B Mixture-of-Experts backbone --- 25.2B total parameters, 3.8B active per token --- but with the autoregressive decoder swapped for a discrete-diffusion head that generates text in parallel 256-token canvases.

- Inference is up to 4× faster than autoregressive Gemma 4 on dedicated GPUs: 1,008 tok/s @ batch=1 on a single H100 (FP8), and 1,100+ tok/s on Hopper-class silicon.

- It is multimodal out of the box --- accepting text, image, and video inputs and producing text output --- and is the first diffusion LM integrated into vLLM.

- Quality is below standard Gemma 4, so Google explicitly recommends the autoregressive family for production. DiffusionGemma is for speed-critical, interactive, non-linear workflows: in-line editing, agent loops, code-completion refinement, rapid iteration.

- It runs locally in \~18 GB VRAM (Q4 GGUF), so a single RTX 4090 / RTX 5090 / 24 GB Apple Silicon Mac can host it.

- Apache 2.0 license --- full commercial use, fine-tuning, redistribution, no royalties. Same license tier as the rest of Gemma 4.

# 1. What Google Actually Announced

The blog post is two paragraphs of marketing and a hundred paragraphs of substance. Here is the substance.

DiffusionGemma is a generative model built by Google DeepMind, based on the 26B A4B MoE Gemma 4 architecture, that generates text using discrete diffusion. Once a noisy 256-token canvas is fully denoised, the model appends it to the KV cache and starts the next canvas. The result is a block-autoregressive, within-block-parallel generator that combines the throughput benefits of image-style diffusion with the long-context benefits of a normal transformer cache.

Key parameters from the model card and launch post:

  ------------------------------------------------------------------------------------------------------------------------
  **Spec**                                      **Value**
  --------------------------------------------- --------------------------------------------------------------------------
  Model ID                                      google/diffusiongemma-26B-A4B-it

  Backbone                                      Gemma 4 26B A4B (MoE)

  Total parameters                              25.2B

  Active parameters per token                   3.8B

  Modality (input)                              Text, image, video

  Modality (output)                             Text

  Canvas size                                   256 tokens (denoised in parallel)

  License                                       Apache 2.0

  Hardware tested                               NVIDIA H100 (Hopper) at FP8, NVFP4, BF16

  Peak throughput (batch=1, H100, FP8)          1,008 tok/s

  Peak throughput (H100, Hopper ceiling)        \>1,100 tok/s

  Quality vs Gemma 4 26B A4B (auto­regressive)   Lower --- Google recommends Gemma 4 for production

  Inference platforms                           Transformers, vLLM, SGLang, llama.cpp, Docker Model Runner, Hermes Agent
  ------------------------------------------------------------------------------------------------------------------------

The release also shipped three quantization formats on day one:

  -----------------------------------------------------------------------------------------------------------------------
  **Format**              **Provider**                             **Use Case**
  ----------------------- ---------------------------------------- ------------------------------------------------------
  BF16 / FP8 (official)   Google                                   Reference, high-throughput H100/B200

  NVFP4 (4-bit NVIDIA)    nvidia/diffusiongemma-26B-A4B-it-NVFP4   Blackwell-optimized, Model Optimizer-quantized

  GGUF (Q4/Q8)            unsloth/diffusiongemma-26B-A4B-it-GGUF   llama.cpp, local CPU/Apple Silicon/RTX consumer GPUs
  -----------------------------------------------------------------------------------------------------------------------

That three-format strategy is the same playbook Google used for Gemma 4 and is the clearest signal that Google wants DiffusionGemma everywhere --- not just in its own data centers.

# 2. Why Diffusion for Text? The Real Architectural Story

If you have ever used an image model like Imagen or Stable Diffusion, you already understand DiffusionGemma. The trick is that for the last three years, language models have not worked that way. They work like a typewriter: one token at a time, left to right, with no ability to revise a committed output.

DiffusionGemma replaces the typewriter with a denoising loop:

**Standard LLM (autoregressive):**

Input: \"The capital of France is\"\
Step 1: generate \"Paris\" (1 token, \~5 ms on H100)\
Step 2: append \"Paris\" → KV cache\
Step 3: generate \".\"\
Done. Three forward passes, three tokens.

**DiffusionGemma (discrete diffusion):**

Input: \"The capital of France is\"\
Canvas: \[MASK, MASK, MASK, \..., MASK\] (256 placeholder tokens)\
\
Iter 1: model attends to the entire canvas in parallel,\
produces 256 noisy guesses simultaneously.\
Iter 2: model locks in the tokens it is confident about,\
refines the rest.\
\...\
Iter N: canvas converges to \"Paris, which is also its\
largest city. The metro area has\...\" (committed\
to KV cache; next canvas begins).\
\
One forward pass per iteration, but every iteration\
fills in 256 token positions at once.

The architectural consequence is a shift in the bottleneck:

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Model type**          **Bottleneck**                                                **Why**
  ----------------------- ------------------------------------------------------------- -------------------------------------------------------------------------------
  Autoregressive LLM      Memory bandwidth (load weights from HBM for each new token)   KV cache grows with sequence; weights re-loaded per token

  Diffusion LM            Compute (run the model over the full canvas in parallel)      KV cache only commits on canvas completion; weights loaded once per iteration
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Local GPUs --- and to a lesser extent TPUs --- have lots of compute and relatively little memory bandwidth. DiffusionGemma\'s compute-bound decode makes them suddenly competitive for interactive text generation.

There are two other architectural details worth knowing:

**1. Block Autoregressive Diffusion**

Once a canvas is fully denoised, it is encoded and committed to the KV cache like a normal block. The next canvas uses that committed block as context. So DiffusionGemma still has autoregressive structure at the block level; it is only within a block that everything is parallel.

**2. Bidirectional context propagation**

Inside the canvas, every position attends to every other position. That means the model can \"see\" the answer before it is finalized, which gives it something autoregressive models cannot do: self-correction mid-generation. If the model starts writing \"The capital of France is Berlin,\" the next denoising iteration can revise that to \"Paris\" before the canvas is committed.

That self-correction property is why Google is positioning DiffusionGemma for in-line editing and non-linear text structures --- the same kind of tasks where diffusion-style image models dominate.

![](./tmp_extracted/diffusiongemma/media/image2.jpg){width="6.0in" height="6.0in"}

*Figure 2 --- Autoregressive typewriter vs. DiffusionGemma parallel canvas. Diagram comparing sequential token-by-token generation against parallel 256-token denoising with bidirectional attention and block-autoregressive KV cache commit.*

# 3. The Numbers: 1,008 tok/s on a Single H100, and Why It Matters

The headline number is 1,008 tokens per second at batch=1 on a single NVIDIA H100 in FP8. Simon Willison independently reproduced \~857 tok/s on a similar setup (\"recorded it running at 857 tokens/second\") the day the model dropped, and a second independent benchmark from the YouTube channel AI Stack Engineer confirmed \~1,000 tok/s in vLLM.

Why this is interesting:

- GPT-5.5 Instant (the current OpenAI default for consumer traffic) tops out around \~200 tok/s from a single endpoint when called via API. You can rent a 4×H100 box from any cloud provider today, run DiffusionGemma, and beat the public GPT-5.5 endpoint per GPU, per dollar.

- Claude Opus 4.8 (Anthropic) is similar --- fast, but not 1,000 tokens-per-second-per-GPU fast.

- Gemma 4 26B A4B (autoregressive) --- the same size and same MoE --- runs at \~250-300 tok/s on a single H100 in FP8. DiffusionGemma is roughly 4× faster on identical hardware.

- Local RTX 4090 (24 GB) --- DiffusionGemma in Q4 GGUF fits in \~18 GB VRAM and generates 40-80 tok/s in llama.cpp, which is comparable to the public GPT-5.5 latency for short prompts, and dramatically faster than most local 26B-class models.

The benchmark table from Google\'s model card:

  --------------------------------------------------------------------------------------------
  **Benchmark**                           **DiffusionGemma 26B A4B**   **Gemma 4 26B A4B**
  --------------------------------------- ---------------------------- -----------------------
  MMLU                                    Lower                        82.7%

  GSM8K (math)                            Lower                        73.2%

  HumanEval (code)                        Lower                        comparable

  LiveCodeBench                           Lower                        comparable

  Inference (tok/s, H100, FP8, batch=1)   1,008                        \~250-300

  Output quality on long-form             Lower                        Higher
  --------------------------------------------------------------------------------------------

Google is candid: quality sits below standard Gemma 4. This is not a model you would pick for a customer-facing chatbot today. It is a model you would pick for:

- Code auto-complete where each keystroke needs sub-50 ms latency

- Agent loops where the same prompt is re-decoded dozens of times

- In-line editing where you paste a paragraph and ask for a rewrite, then a third draft, then a fourth, in tight iteration

- Repetitive, structured output like JSON, SQL, code diffs, where self-correction mid-canvas helps

For production text where quality matters most --- long-form writing, customer emails, factual Q&A --- the autoregressive Gemma 4 family remains the recommended choice.

# 4. Run It Yourself Tonight: vLLM, Transformers, llama.cpp

One of the most underrated parts of the launch is that DiffusionGemma is the first diffusion LM natively supported in vLLM. Until now, running a diffusion language model meant custom CUDA kernels and bespoke sampling code. With vLLM support, you can serve DiffusionGemma with the same APIs you already use for Llama, Qwen, or Gemma.

## Option A --- vLLM (production)

\# Install (the DiffusionGemma support ships in vLLM main as of June 10, 2026)\
pip install -U vllm\
\
\# Serve the FP8 weights from Hugging Face\
vllm serve google/diffusiongemma-26B-A4B-it \\\
\--port 8000 \\\
\--tensor-parallel-size 1 \\\
\--max-model-len 8192\
\
\# Call it\
curl http://localhost:8000/v1/chat/completions \\\
-H \"Content-Type: application/json\" \\\
-d \'{\
\"model\": \"google/diffusiongemma-26B-A4B-it\",\
\"messages\": \[{\"role\": \"user\", \"content\": \"Write a haiku about diffusion language models.\"}\],\
\"temperature\": 0.7\
}\'

## Option B --- Transformers (research / fine-tuning)

import torch\
from transformers import AutoModelForCausalLM, AutoTokenizer\
\
model_id = \"google/diffusiongemma-26B-A4B-it\"\
tok = AutoTokenizer.from_pretrained(model_id)\
model = AutoModelForCausalLM.from_pretrained(\
model_id, torch_dtype=torch.bfloat16, device_map=\"cuda\"\
)\
\
\# DiffusionGemma uses a diffusion-style generate API\
out = model.diffusion_generate(\
inputs=tok(\"The capital of France is\", return_tensors=\"pt\").input_ids.to(\"cuda\"),\
max_new_tokens=256,\
)\
print(tok.decode(out\[0\]))

## Option C --- llama.cpp on a laptop (Q4 GGUF)

\# Grab the Unsloth GGUF (Q4 fits in \~18 GB VRAM)\
huggingface-cli download unsloth/diffusiongemma-26B-A4B-it-GGUF \\\
\--local-dir unsloth/diffusiongemma-26B-A4B-it-GGUF\
\
\# Run with the DiffusionGemma fork of llama.cpp\
llama-cli -m unsloth/diffusiongemma-26B-A4B-it-GGUF/diffusiongemma-26B-A4B-it-Q4_K_M.gguf \\\
-p \"The capital of France is\" -n 256

Apple Silicon users can run the same GGUF through Ollama, LM Studio, or MLX, since Unsloth and the community shipped day-one integrations.

# 5. The Sudoku Trick: Why DiffusionGemma Is Suddenly Better at Multi-Pass Tasks

There is one benchmark in the developer guide that is worth dwelling on, because it tells you what diffusion is actually for.

> *\> The base DiffusionGemma model is not specifically trained to solve Sudoku puzzles (\~0% success rate). Applying a simple JAX SFT recipe on a Sudoku dataset raises correctness to 80% success, while decreasing the overall inference step count.*

Why? Because Sudoku is exactly the kind of problem where you need to revisit the same square multiple times. An autoregressive model commits \"5\" to a square and then has to keep building forward, hoping later context justifies the guess. A diffusion model can place \"5\" tentatively, see that the column now needs a 3 in another square, and revise the original \"5\" to a \"7\" mid-generation. It is the same trick that made Stable Diffusion better than GANs at hands and text in images.

Other tasks where this matters:

- Code refactoring where renaming a variable in one function forces renames in five others

- Long-form structured output (JSON, YAML, SQL) where one field depends on another

- Translation where a later word choice forces a re-translation of an earlier phrase

- Planning and re-planning in agent loops

For each of these, the block-autoregressive part of DiffusionGemma still preserves long-range coherence, while the intra-canvas diffusion part gives the model the freedom to revise as it goes.

# 6. Who Else Is in the Diffusion-LM Race

DiffusionGemma is not the first diffusion language model. It is the first one an ordinary developer can actually deploy.

  ----------------------------------------------------------------------------------------------------------------------------------------------
  **Model**                                    **Year**       **Open weights?**   **License**    **Where it runs**
  -------------------------------------------- -------------- ------------------- -------------- -----------------------------------------------
  Mercury (Inception Labs)                     2025           No (API only)       Proprietary    Inception cloud

  Gemini Diffusion (Google research preview)   2025           No                  Proprietary    Google AI Studio, limited preview

  LLaDA (Renmin University)                    Feb 2025       Yes                 Apache 2.0     Research only, no production inference engine

  SEDD (Meta / FAIR)                           2025           Yes                 MIT            Research only

  MDLM (Meta / FAIR)                           2025           Yes                 MIT            Research only

  DiffusionGemma (Google DeepMind)             Jun 10, 2026   Yes                 Apache 2.0     vLLM, SGLang, Transformers, llama.cpp, MLX
  ----------------------------------------------------------------------------------------------------------------------------------------------

Until June 10, every open diffusion LM was research code that you would have to integrate yourself. DiffusionGemma is the first to ship production-grade serving support on day one. That is the real strategic win for Google --- it sets the bar for \"diffusion LM\" in the same way the original Gemma set the bar for \"small open model\" in 2024.

![](./tmp_extracted/diffusiongemma/media/image3.jpg){width="6.0in" height="6.0in"}

*Figure 3 --- The diffusion LM landscape, 2025-2026. Diagram comparing closed vs. open-weights diffusion language models on axes of license, serving maturity, and benchmark performance, with DiffusionGemma highlighted as the first production-grade open option.*

# 7. What It Means for You

## If you are a developer

- Add it as a fast-decoding second model. Use autoregressive Gemma 4 26B for the user-facing reply, then use DiffusionGemma for the agent\'s internal tool-call planning loop. You get high quality on output, low latency on iteration.

- Fine-tune on your structured-output data. If your app emits JSON / SQL / code, fine-tune DiffusionGemma in JAX with a small SFT pass --- Google\'s developer guide shows the recipe takes a few hours on a single H100.

- Watch the local-GPU use case. A 24 GB RTX 4090 running Q4 GGUF gives you sub-second first-token latency at \~50 tok/s. That is competitive with public APIs for any interactive single-user workload.

## If you are a researcher

- The bidirectional, parallel-canvas generation unlocks training paradigms that autoregressive models cannot use: plan-then-refine, latent thinking inside the canvas, self-correction losses, and diffusion-style RL for reasoning.

- Block-autoregressive diffusion is a genuinely new scaling axis. Expect a wave of follow-up papers from Google DeepMind, Meta FAIR, Mistral, and the academic community in the next 90 days.

## If you are a product leader

- Latency is the new cost. If your product has any \"type-while-AI-thinks\" surface --- IDE plugins, search bars, in-line rewrites, agent chat UIs --- diffusion LM throughput changes the economics. A model that returns 1,000 tok/s on one GPU costs \~\$0.10 per million output tokens at retail H100 pricing, vs. \~\$3-15/M for closed APIs.

- Quality is not yet there for primary user-facing text. Use it as a \"thinking partner\" or \"in-line editor\" under the hood, not as the model that writes your user\'s email.

# 8. What Could Go Wrong

Three risks worth naming, all acknowledged in Google\'s launch materials:

**1. Quality regression on hard tasks.**

DiffusionGemma is not yet at parity with autoregressive Gemma 4 on MMLU, GSM8K, or long-form reasoning. If you deploy it as a general-purpose assistant, your users will notice. Use it for what it is good at.

**2. Ecosystem maturity.**

vLLM and llama.cpp support landed on day one, but SGLang, TGI, OpenRouter, LiteLLM, and most production gateways are still catching up. Production deploys should plan for at least 2-4 weeks of integration work.

**3. Self-correction can also be self-sabotage.**

Because the model can revise mid-canvas, it occasionally overthinks simple prompts, generating verbose or unstable drafts. Sampling parameters (temperature, top-p, denoising steps) need tuning per workload --- there is no universal default yet.

None of these are deal-breakers. All of them are why Google called DiffusionGemma experimental.

# 9. The Bottom Line

June 10, 2026 is the day \"open weights\" stopped meaning \"autoregressive\".

For the last three years, every serious open-weights language model --- Llama, Mistral, Qwen, DeepSeek, Gemma, Phi, OLMo --- has been the same fundamental architecture: a transformer decoding one token at a time. DiffusionGemma is the first production-grade open model that breaks that pattern. It runs in vLLM, it ships in three quantization formats, it fits on a single 24 GB consumer GPU, and it does so under an Apache 2.0 license that means no surprises.

The throughput numbers are real: 1,008 tok/s on a single H100, 1,100+ on Hopper, 40-80 tok/s on a 4090 laptop. The quality numbers are also real: below standard Gemma 4. Both are fine. The point of DiffusionGemma is not to replace your favorite chatbot --- it is to give you a second tool for the part of the work where speed beats depth, and to do it without giving anyone a recurring API bill.

If you build anything where the user is staring at a blinking cursor waiting for the model to finish a thought, DiffusionGemma is the most interesting thing Google has shipped this year. Run it tonight. It is one pip install and one vllm serve away.

*Welcome to the post-autoregressive era.*

# Sources & Verification

**Version verified (2026-06-13, \~04:15 Asia/Shanghai cron):**

### Primary sources

- Official Google Blog: blog.google/innovation-and-ai/technology/developers-tools/diffusion-gemma-faster-text-generation

- Developer guide: developers.googleblog.com/diffusiongemma-the-developer-guide

- Hugging Face model card: huggingface.co/google/diffusiongemma-26B-A4B-it

- Gemma docs model card: ai.google.dev/gemma/docs/diffusiongemma/model_card

### Architecture & parameters --- 9 independent sources

- Simon Willison\'s Weblog (June 10, 2026): simonwillison.net/2026/Jun/10/diffusiongemma --- independently reproduced \~857 tok/s on day one

- GIGAZINE: gigazine.net/gsc_news/en/20260611-google-ai-diffusiongemma

- MarkTechPost AI summary (Reddit + X coverage)

- VentureBeat: venturebeat.com/technology/googles-diffusiongemma-generates-256-tokens-in-parallel-and-self-corrects-as-it-goes

- Technology.org: technology.org/2026/06/11/googles-diffusiongemma-generates-text-4x-faster-on-local-gpus

- Spheron Network GPU deployment guide: spheron.network/blog/deploy-diffusiongemma-gpu-cloud

- Fahd Mirza YouTube benchmark (1,100 tok/s ceiling): youtube.com/watch?v=hwKZq0_xG5M

- AI Stack Engineer YouTube (1,000 tok/s + 18 GB VRAM): youtube.com/watch?v=OtchSLKlGV4

- Pat Löber X / Hugging Face announcement: huggingface.co/google/diffusiongemma-26B-A4B-it

### Three quantization formats verified

- FP8/BF16: google/diffusiongemma-26B-A4B-it (official)

- NVFP4 (NVIDIA-optimized 4-bit for Blackwell): huggingface.co/nvidia/diffusiongemma-26B-A4B-it-NVFP4

- GGUF (Q4/Q8, Unsloth): huggingface.co/unsloth/diffusiongemma-26B-A4B-it-GGUF

### Throughput numbers

- 1,008 tok/s @ batch=1 on H100 (FP8) --- Google official + VentureBeat

- 1,100+ tok/s on Hopper --- NVIDIA NVFP4 model card + Fahd Mirza YouTube

- Up to 4× faster than autoregressive Gemma 4 --- Google official

### Sudoku trick (0% → 80%)

- developers.googleblog.com/diffusiongemma-the-developer-guide --- verified verbatim

### Diffusion-LM landscape cross-checked against

- Mercury (Inception Labs, 2025) --- inceptionlabs.ai

- LLaDA (Renmin University, Feb 2025) --- arxiv.org/abs/2502.09992

- SEDD / MDLM (Meta FAIR, 2025) --- arxiv.org/abs/2502.09992 family

- Gemini Diffusion (Google research preview, 2025) --- blog.google

### Gemma 4 26B A4B baseline

- ai.google.dev/gemma/docs/core/model_card_4 + Hugging Face blog post huggingface.co/blog/gemma4

## Unverified:

- Exact per-task quality deltas on long-form reasoning vs Gemma 4 (Google gave directional guidance only; independent benchmarks like Vellum, Vals AI, and LMSYS Chatbot Arena have not yet scored DiffusionGemma).

- Final vLLM throughput numbers on Blackwell B200 in production (NVFP4 model card quotes peak, not sustained).

- SGLang, TGI, and OpenRouter production-grade support timing (the integrations exist but GA dates have not all been announced).

- Behavior on languages other than English (Gemma 4 family supports 100+ languages, but DiffusionGemma has only been benchmarked in English-heavy evals).

- Whether Google will ship a smaller DiffusionGemma variant (E2B / E4B / 9B) --- no announcement yet.

*If you found this useful, share it with one engineer who complains about API latency. They\'ll know what to do with it.*
