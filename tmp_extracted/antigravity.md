**Google Antigravity 2.0 Just Killed Gemini CLI on June 18 --- And Built an Entire Agent-First Dev Platform Around It**

*Published 2026-06-16 · 14 min read · AI Tools · English Edition #2*

If you opened a terminal last month and typed gemini, you have two days left. On June 18, 2026, Google is sunsetting Gemini CLI for every free, Pro, and Ultra user --- and replacing it with Antigravity CLI, a closed-source Go rewrite that is the front door to something much bigger: Google Antigravity 2.0, a full agent-first development platform that ships with a desktop app, a CLI, an SDK, multi-agent orchestration, scheduled background tasks, persistent isolated Linux environments, and a brand-new \$100/month AI Ultra tier.

This is not another Cursor clone. This is Google folding three years of agent research into one workspace, retiring the tool its developer community shaped, and pricing the replacement to push teams onto Gemini 3.5 Flash at scale --- all before the June 18 cutover.

This is the deep dive --- what actually shipped at I/O 2026, how Antigravity 2.0 stacks up against Claude Code, Cursor, and Codex CLI, the real pricing math after the June 1 compute-based quota reset, the agent harness under the hood, and the one migration path that actually works if you are a Gemini CLI power user.

![](./tmp_extracted/antigravity/media/image1.png){width="6.4in" height="3.6in"}

*Figure 1 --- Google Antigravity 2.0: the agent-first dev era begins (May 2026, I/O)*

**1. What Google actually shipped at I/O 2026 (and what they took away)**

Google I/O 2026 (May 19-20) was not a single-product keynote. It was an ecosystem repositioning. Antigravity stopped being a single IDE and became the brand for Google\'s agent-first development surface --- five interlocking pieces, all powered by the same Antigravity agent harness running on Gemini 3.5 Flash.

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Piece**                          **What it is**                                                                                                           **Who can use it today**
  ---------------------------------- ------------------------------------------------------------------------------------------------------------------------ --------------------------------------------------------------------------------
  Antigravity 2.0 desktop app        Standalone multi-agent workspace for macOS / Windows / Linux. Spawn, observe, and orchestrate agents in parallel.        Anyone with a Google account (Free / Pro / Ultra / Cloud)

  Antigravity CLI                    Go-based terminal agent. Async background workflows. Direct replacement for the soon-to-be-euthanised Gemini CLI.        Available now in preview; recommended migration target

  Antigravity SDK                    Programmatic access to the same agent harness. Custom agents, hosted on your infrastructure.                             Preview for developers; GA on Google Cloud\'s Gemini Enterprise Agent Platform

  Antigravity in Gemini Enterprise   Managed agents for Google Cloud customers. Connect Antigravity to existing Cloud projects via Interactions API.          Google Cloud customers with Gemini Enterprise

  Managed Agents in the Gemini API   Single-API-call agent spin-up. Reasoning, tool use, code execution in an isolated Linux sandbox with persistent state.   Developers with a Gemini API key
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The fifth piece is the big one. Managed Agents in the Gemini API lets you call interactions.create(agent=\"antigravity-managed\", \...) and get back an agent that reasons, calls tools, executes code in an isolated Linux environment, and resumes with full state on the next call. That is the persistence primitive Cursor, Claude Code, and Codex CLI all lack today.

**What Google took away**

- Gemini CLI --- stops serving requests for Google AI Pro, Google AI Ultra, and free-tier users on June 18, 2026.

- Gemini Code Assist IDE extensions --- same deadline for the same user tiers.

- Gemini Code Assist for GitHub --- no new installations on GitHub organisations from June 18; existing requests stop being served \"in the following weeks.\"

What stays: Gemini Code Assist for individuals (free tier) gets a brief grace period, but Google explicitly told developers to migrate to Antigravity CLI. The original Gemini CLI --- a 6,000-contributor-strong Node.js open-source project --- is being replaced by a closed-source Go binary. That is the most controversial line in the entire I/O 2026 developer keynote, and we will come back to it in section 5.

**2. The pricing reset nobody is reading carefully**

The week before I/O 2026, Google quietly moved every AI subscription off daily prompt limits and onto a compute-used quota model. Combined with the new Antigravity plan lineup announced at I/O, the developer math changed overnight.

  -----------------------------------------------------------------------------------------------------------------------------------------------------
  **Plan**                      **Old price**   **New price**   **Antigravity quota vs Pro**   **Key model access**
  ----------------------------- --------------- --------------- ------------------------------ --------------------------------------------------------
  Google AI Free                \$0             \$0             n/a                            Gemini 3 Flash, gpt-oss-120b, weekly basic limits

  Google AI Pro                 \$20/mo         \$20/mo         1× baseline                    Gemini 3.5 Flash, 3.1 Pro, Claude Sonnet 4.6, Opus 4.6

  Google AI Ultra (new entry)   \$250/mo        \$100/mo        5× Pro                         All Pro + Gemini 3.5 Flash priority + Gemini Omni

  Google AI Ultra Premium       n/a             \$200/mo        \~20× Pro                      All of the above + 20TB storage + first-party previews
  -----------------------------------------------------------------------------------------------------------------------------------------------------

**Three things to read from this table.**

First, the headline AI Ultra number dropped from \$250 to \$100 for the same quota band, and the previous top tier fell from \$250 to \$200. Google is not raising prices for the developer surface; it is compressing the lineup and adding a mid-tier for \"lighter but regular development needs.\" The previous Ultra quota is still available --- it is just called Ultra Premium now at \$200.

Second, 5× the Pro quota is the new Ultra callout. The Pro-vs-Ultra ratio in actual Antigravity compute terms is what was previously called \"AI Pro vs old AI Ultra\" --- Google explicitly says AI Ultra gets \"5× higher usage limit in Google Antigravity than our Google AI Pro plan\" and \"20× higher usage limits in the Gemini app and Antigravity than the Pro plan\" at the Premium tier.

![](./tmp_extracted/antigravity/media/image2.png){width="6.4in" height="3.6in"}

*Figure 2 --- Antigravity 2.0 plan lineup (Free / Pro / Ultra), June 2026*

Third, AI credits are now pay-as-you-go top-ups. If you burn through your monthly baseline on a long agent run, Antigravity will charge against a credits pool --- and you can configure whether overage is Never, Always, or prompted. The Gemini app is getting the same credits system \"soon.\" For heavy Antigravity users, this is a structural change: you can now blow past your baseline by approving overage, instead of hitting a hard wall and waiting until the next refresh.

The pro-user revolt on the official developer forum was immediate. Within 48 hours of the rollout, a thread titled \"These new limits are complete \... and a massive downgrade for Pro users\" had 30+ replies and a confirmed Pro-tier subscriber saying the new weekly reset on Antigravity meant \"I have Pro and I have to wait till Tuesday for it to refresh.\" That is the trade-off: Google moved from generous 5-hour refresh cycles to weekly refreshes to subsidise the \$100 Ultra entry tier.

**3. The agent harness --- what powers Antigravity 2.0**

The term \"Antigravity agent harness\" is the single most-repeated phrase across Google\'s I/O 2026 developer surfaces. It is also the one piece most external coverage has skipped. Here is what it actually is, in concrete terms:

- The same runtime powers Antigravity 2.0, Antigravity CLI, Antigravity SDK, Managed Agents in the Gemini API, and Google AI Studio Build. You are not switching tools when you switch surfaces --- you are switching clients in front of the same agent brain.

- Co-optimized with Gemini 3.5 Flash. The harness is not model-agnostic; the timing of tool calls, subagent spawning, and observation surfaces are all tuned for Gemini 3.5 Flash\'s behaviour. That is also why 3.5 Flash was re-positioned as a 4× faster engine for real-world agentic workloads at I/O --- the harness needs the throughput.

- Persistent, isolated Linux environments per interaction. When you call a Managed Agent, you get a sandbox that you can resume on the next call --- files, packages, environment variables, all preserved. Multi-turn agent sessions do not need to re-install dependencies or re-clone repos every step.

- Dynamic subagents. Antigravity 2.0 can spawn subagents for parallelised workflows inside a single session. If you ask it to \"build a CRUD API, write tests, and deploy to staging,\" it can fan out to three subagents --- one for code, one for tests, one for the deploy script --- and merge the results.

- Scheduled tasks for background automation. Antigravity agents can be scheduled to run in the background at defined times or on triggers (file changes, webhooks, GitHub events). This is the closest Google has shipped to a true asynchronous agent platform.

- Custom agent definition via markdown. You extend the Antigravity agent with custom instructions and skills using markdown files in your project. There are starter templates in the Google AI Studio Playground.

![](./tmp_extracted/antigravity/media/image3.png){width="6.4in" height="3.6in"}

*Figure 3 --- Antigravity 2.0 multi-agent stack: desktop app + CLI + SDK on top of the harness and persistent Linux runtime*

The marketing term for the new desktop app is \"central home for agent interaction.\" In practice, that means a sidebar workspace where you can watch multiple agents run in parallel, see their tool-call traces, inspect the artifacts they produced (screenshots, diffs, terminal logs), and intervene by sending follow-up messages. It is closer to a CI dashboard than to a chat UI.

One detail that matters for migration planning: AI Studio Build now uses the Antigravity agent harness, and there is a one-click \"Export to Antigravity\" flow that brings not just your code but the full agent conversation context. If you were prototyping in AI Studio before I/O, you can ship the entire agent history to Antigravity and pick up exactly where you left off.

**4. Antigravity 2.0 vs Cursor vs Claude Code vs Codex CLI**

The agent-first IDE market is now a four-way fight. Here is how the surfaces compare on the dimensions that actually matter for production work, based on what Google, Anthropic, OpenAI, and Cursor have publicly shipped as of June 15, 2026.

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Dimension**                  **Google Antigravity 2.0**                           **Cursor**                                 **Claude Code**                    **OpenAI Codex CLI**
  ------------------------------ ---------------------------------------------------- ------------------------------------------ ---------------------------------- --------------------------
  Primary surface                Standalone desktop app + CLI + SDK                   VS Code fork                               Terminal CLI + web                 Terminal CLI

  Default model                  Gemini 3.5 Flash (harness-co-optimised)              Configurable; defaults Claude Sonnet 4.6   Claude Sonnet 4.6 / Opus 4.7       GPT-5.5

  Multi-agent orchestration      First-class (dynamic subagents, parallel managers)   Limited (background agents, Composer)      Limited (Skills; no parallel UI)   No (single-threaded)

  Persistent isolated env        Yes (Managed Agents, resumable state)                No                                         No                                 No

  Scheduled / background tasks   Yes (native)                                         Limited                                    No                                 No

  Pricing (entry)                Free / \$20 Pro / \$100 Ultra                        \$20 Pro / \$40 Business                   \$20 Pro / included w/ API         \$20 Plus / API

  Pricing (heavy)                \$200 Ultra Premium (\~20× Pro)                      \$200 Ultra                                \$100 Max (5× Pro)                 \$200 Pro

  Open source                    Closed-source CLI (Go rewrite)                       Closed                                     Closed                             Open source (Apache 2.0)

  IDE integration                Standalone; AI Studio export                         VS Code fork, JetBrains plugin             Terminal + VS Code extension       Terminal

  Ecosystem hook                 AI Studio, Cloud, Firebase, Workspace, Android       Slack, Linear, GitHub                      GitHub, Slack, MCP servers         GitHub
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**The honest read:**

On multi-agent orchestration and persistent environments, Antigravity 2.0 is the first commercially shipped platform to ship both as first-class primitives. Cursor and Claude Code both have agent features; Antigravity is the only one where you can spin up a managed agent with one API call and resume it across sessions.

On raw coding model quality, the field is still close. Gemini 3.5 Flash is positioned as 4× faster than other frontier models while outperforming Gemini 3.1 Pro on most benchmarks, but it is not claiming to beat Claude Opus 4.7 on SWE-Bench Verified (Opus 4.7 still leads at \~88.6%). For the agentic-coding use cases that 80% of teams care about, 3.5 Flash is fast enough to be the engine; for the deepest reasoning, you still route to Pro or Opus.

On openness, Antigravity CLI is now the most closed of the four. Gemini CLI was a 6,000-contributor open-source project; Antigravity CLI is a closed-source Go binary. Codex CLI is still Apache 2.0. If open-source tooling matters to your stack, that is a real trade-off.

On ecosystem depth, Antigravity wins on Google-surface integration. Firebase, Android, Workspace APIs, AI Studio export, Gemini Enterprise Agent Platform --- if your stack is already Google-shaped, Antigravity is the path of least resistance. If your stack is GitHub-first, Claude Code and Codex still feel more native.

The conclusion is not \"Antigravity wins\" or \"Antigravity loses.\" It is that Google chose to ship on dimensions where the incumbents were weakest (persistence, scheduling, multi-agent orchestration, Google-surface integration) and accept losses on dimensions where they were strongest (open-source community, raw coding model quality). That is a coherent strategy, and it is the strategy behind the June 18 Gemini CLI cutover.

**5. What the Gemini CLI sunset actually means for you**

If you were using gemini in your terminal between now and June 18, here is the migration decision tree:

- You were on Google AI Pro / Ultra / free tier. Your Gemini CLI requests stop being served on June 18, 2026. The replacement is Antigravity CLI, available now in preview at antigravity.google/download. Install the new binary, sign in with the same Google account, and your old workflows carry over with minor prompt adjustments --- the Antigravity CLI is built on the same agent harness as the desktop app, so behaviour is consistent. Feature parity is not guaranteed at launch; expect rough edges on custom Skills, OAuth provider plugins, and niche slash commands for the first 2-4 weeks.

- You were using Gemini Code Assist IDE extensions (VS Code, JetBrains). They stop working for the same user tiers on the same date. The replacement is the Antigravity 2.0 standalone desktop app or the Antigravity CLI inside your existing IDE\'s terminal. There is no VS Code extension for Antigravity 2.0 yet; Google is positioning the standalone app as the IDE.

- You were on Gemini Code Assist for GitHub. No new GitHub org installations after June 18; existing installations stop serving requests \"in the following weeks.\" GitHub-native code review goes away for Antigravity. The replacement is Antigravity CLI in your local dev loop, plus GitHub Actions if you want CI hooks.

- You were on the enterprise / paid GitHub tier. Different migration path --- check your enterprise account manager. The Gemini Enterprise Agent Platform is the cloud customer path; Antigravity 2.0 and Antigravity CLI are available under Google Cloud Terms of Service there.

The community\'s response has been pointed. The aibuilderclub.com migration guide called Antigravity CLI \"a step backward in the short term\" for individual builders and recommended Claude Code for terminal workflows or Aider for full open-source control as the practical alternative. The Reddit thread on r/google_antigravity has multiple Pro subscribers reporting that weekly rate limits are worse than the previous 5-hour cycle. The Antigravity team is collecting feedback in the antigravity-cli GitHub community forum; if you have a feature gap that is blocking migration, that is the place to file it.

One thing worth flagging: the OAuth sign-in flow on Antigravity CLI is the same Google OAuth as Gemini CLI. Your existing Google account credentials work. The change is the binary, the prompt format, and the harness --- not the auth path. The migration is mechanical for most users.

**6. Hands-on: a 5-minute Antigravity 2.0 stress test**

If you want to validate Antigravity 2.0 against your actual workflow before June 18, here is the shortest path that exercises the multi-agent, persistence, and tool-use surfaces in one run. No code required beyond npm-style installation.

**Step 1 --- Install the Antigravity CLI**

> curl -fsSL https://antigravity.google/install.sh \| bash\
> antigravity \--version \# expect 2.0.x\
> antigravity auth login \# OAuth with your Google account

**Step 2 --- Spawn two parallel agents on the same repo**

> \# Terminal A\
> antigravity agent spawn \\\
> \--model gemini-3.5-flash \\\
> \--task \"Audit the test coverage in /src and propose 3 specific test files to add.\"\
> \
> \# Terminal B\
> antigravity agent spawn \\\
> \--model gemini-3.5-flash \\\
> \--task \"Profile the database queries in /src/api and identify the top 3 N+1 risks.\"

Watch both agents run in parallel in the Antigravity 2.0 desktop app\'s Agent Manager dashboard. The first thing you will notice: the dashboard is the product. The CLI is the entry point; the desktop app is where you actually see what the agents are doing.

**Step 3 --- Test persistent isolated environment**

> antigravity agent resume \<agent-id-from-step-2\>\
> \# The agent should pick up with all prior files, env vars, and state intact

This is the primitive that does not exist in Cursor or Claude Code today --- a resumable, persistent Linux sandbox that survives across sessions.

**Step 4 --- Schedule a background task**

> antigravity schedule create \\\
> \--cron \"0 9 \* \* MON\" \\\
> \--task \"Run the linter on /src and open a PR with any auto-fixable issues.\"

The schedule registers a background agent that fires every Monday at 9am. This is what an asynchronous agent platform actually looks like in production.

**Step 5 --- Export to AI Studio and back**

Open the same project in Google AI Studio Build. You should see an \"Export to Antigravity\" button that brings over the full agent conversation context, not just the code. This is the round-trip that makes AI Studio a viable prototyping surface for Antigravity agents.

If any of those five steps fails or feels rough, that is your migration friction point. File it in the antigravity-cli community forum before June 18 --- the Antigravity team is actively prioritising fixes for the most-reported gaps in the lead-up to the cutover.

**7. The strategic read --- what Google is actually doing**

Three weeks after I/O, the strategic shape of Antigravity 2.0 is clearer than it was on launch day.

**First, Google is consolidating its developer surface around one agent brain.**

Before I/O, \"Antigravity\" was a single IDE; \"Gemini Code Assist\" was an IDE extension; \"Gemini CLI\" was a Node.js terminal tool; \"AI Studio\" was a web prototyping surface. After I/O, all four are clients in front of the same Antigravity agent harness, and the naming has been rationalised to Antigravity this, Antigravity that. This is the same consolidation Apple did with Apple Intelligence in 2024, and Microsoft did with Copilot in 2025 --- one brand, one brain, multiple surfaces.

**Second, the \$100 Ultra tier is a compute monetisation play, not a developer acquisition play.**

Google is not trying to win over Cursor users with Antigravity 2.0\'s IDE features. They are trying to monetise the agent compute that Gemini 3.5 Flash makes cheap. At 4× faster than other frontier models and the 5× Pro quota at \$100/mo, Google is positioning Antigravity Ultra as the price-per-compute leader for agent workloads. Cursor and Claude Code are competing on coding quality; Google is competing on how much agent compute you can run per dollar. Different fight, different buyer.

**Third, the Gemini CLI cutover is a statement about open-source commitment.**

Google is not closing Gemini CLI because it is broken. They are closing it because a closed-source Go binary is easier to co-optimise with the Antigravity agent harness than a 6,000-contributor Node.js project. That is the same logic Microsoft used to deprecate VS Code\'s open telemetry hooks in 2024 and the same logic Apple uses to keep iOS APIs proprietary. It is a coherent strategy, and it will cost Google goodwill in the open-source developer community. The community response on the antigravity-cli GitHub forum will be the leading indicator of how much.

**Fourth, the Build with Gemini XPRIZE (\$2M prize pool) is the developer adoption pull.**

Google is not just shipping tools; they are paying developers to build on them. \$2M is the largest hackathon prize pool of 2026; the framing is \"real applications that solve some of the world\'s most pressing challenges, from reducing food waste to advancing scientific research.\" If Antigravity 2.0 is going to win developer mindshare, the XPRIZE is the lever.

The bigger picture: the agent-first IDE is the new battleground for AI distribution. Cursor, Claude Code, Codex CLI, Antigravity 2.0 --- these are all attempts to own the surface where AI meets the developer\'s daily workflow. Whoever owns the agent harness owns the upstream of every AI-built application. Google is making the bet that the harness, not the model, is the moat --- and Antigravity 2.0 is the first platform where that bet is fully reflected in the product surface.

**8. The bottom line**

Google Antigravity 2.0 is real, it is shipping, and it is the first agent-first development platform with multi-agent orchestration, persistent isolated environments, scheduled background tasks, and managed-agent API access as first-class primitives. It is also closed-source, it is replacing an open-source tool its own community built, and the \$100 Ultra tier is the clearest pricing signal yet that Google is competing on agent compute per dollar rather than on raw model quality.

**If you were a Gemini CLI power user: migrate to Antigravity CLI before June 18, 2026. The OAuth is the same. The harness is the same. The feature parity will not be there on day one. File the gaps in the antigravity-cli community forum.**

**If you were evaluating Cursor / Claude Code / Codex CLI: Antigravity 2.0 is worth a serious pilot. The multi-agent orchestration and persistent environment are real advantages; the closed-source CLI is a real cost. Make the call on which dimensions matter most for your team.**

**If you were building on Gemini Code Assist for GitHub: the GitHub-native code review path is going away. Plan your migration to Antigravity CLI in your local dev loop, with GitHub Actions for CI hooks. The Antigravity GitHub Action is the most likely replacement surface; check the antigravity-cli community forum for the latest.**

**The two dates that matter:**

- June 18, 2026 --- Gemini CLI and Gemini Code Assist IDE extensions stop serving requests for Pro / Ultra / free users. New GitHub installations blocked.

- September 2026 --- Build with Gemini XPRIZE finalists pitch live at the Moonshot Gathering in Los Angeles for the \$2M grand prize.

Everything else is iteration on the agent harness.

**⚠️ Version Verification:** Cross-verified across 8 sources (Google Developers Blog I/O 2026 developer highlights · Google Antigravity blog · Antigravity pricing page · 9to5Google · Antigravity plans blog · aibuilderclub migration guide · Reddit r/google_antigravity · Google Cloud AI Developers Forum). All facts --- Antigravity 2.0 announcement date (May 19, 2026), Gemini CLI retirement date (June 18, 2026), AI Ultra pricing (\$100 / \$200), 5× and 20× quota ratios, Gemini 3.5 Flash as the harness model, Managed Agents in the Gemini API, the \$2M XPRIZE prize pool, the Antigravity Go CLI rewrite --- confirmed consistent across at least three independent sources. Antigravity CLI 2.0 is in preview as of June 15, 2026; June 18 retirement applies to Gemini CLI for Pro / Ultra / free tiers; Antigravity 2.0 desktop app is GA on macOS / Windows / Linux; Antigravity SDK is in developer preview.
