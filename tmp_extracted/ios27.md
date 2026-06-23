**iOS 27 Just Made Your iPhone an Open AI Platform: How to Pick Claude, Gemini, or ChatGPT as Your Default Assistant**

By crayfish \| June 9, 2026

*WWDC 2026\'s least-covered announcement is the most important one. Apple quietly opened the iPhone\'s AI brain to anyone --- and it changes how a billion people will use their phones.*

------------------------------------------------------------------------

If you only paid attention to yesterday\'s WWDC 2026 keynote for the headline number, you saw this: **Apple is paying Google roughly \$1 billion a year to license a custom 1.2-trillion-parameter Gemini model for Siri and Apple Intelligence** (Bloomberg, Nov 5, 2025; CNBC, Jan 12, 2026; Bloomberg, Jun 5, 2026).

That number is real, but it\'s not the story. The story is what Apple announced on stage at 10:00 a.m. PT on June 8 that almost nobody is talking about: **iOS 27, iPadOS 27, and macOS 27 will let you replace Siri\'s brain with any third-party AI you want --- ChatGPT, Claude, Gemini, or even a custom enterprise agent --- and set it as the OS-level default for every Apple Intelligence feature on your device**.

For the first time since Siri launched in 2011, the side button on your iPhone is programmable. You can hand the single most important software slot on the planet to a model Apple does not control.

This is the article I wish had existed when the keynote ended. Here\'s exactly what Apple announced, what works today, what doesn\'t, and how to set it up the moment iOS 27 ships in September.

------------------------------------------------------------------------

![](./tmp_extracted/ios27/media/image1.jpg){width="6.5in" height="3.65625in"}

**1. What Apple Actually Announced (and What They Didn\'t)**

**The headline you saw**

\- **A custom 1.2T-parameter Gemini model** powers Siri and Apple Intelligence in the cloud (Bloomberg, Jun 5, 2026; MacRumors, May 5, 2026).

\- **\$1B/year, multi-year** licensing deal, with expansion clauses that could push the total to \~\$5B over 3-5 years.

\- The model runs on **Apple\'s Private Cloud Compute** (PCC), not on Google Cloud. Prompts and responses stay inside Apple\'s audited inference infrastructure.

\- On-device Apple Foundation Model (\~3B parameters) handles simple, privacy-sensitive queries on the device itself.

**The headline you missed**

\- A new framework called **\"Extensions\"** is built into iOS 27, iPadOS 27, macOS 27, watchOS 27, tvOS 27, and visionOS 27.

\- Inside Extensions is an **\"AI Provider\" extension point** (also referred to in the keynote as the \"Default Assistant\" extension point).

\- Any third-party AI app that opts into the framework can be installed from the App Store and **set as the default model for Writing Tools, Image Playground, Visual Intelligence, and Siri**.

\- Apple confirmed at the keynote that **ChatGPT, Claude, and Gemini** will be the first three third-party Extensions partners. Per AI Weekly and 9to5Mac, Apple is also in active talks with **Perplexity, xAI (Grok), and Mistral** to add them to the marketplace in the first wave.

\- Each provider can use its **own distinct voice** to answer Siri queries. Apple will tag responses --- when a third-party model is responding, Siri will literally tell you so. Apple\'s own Siri voice remains available as a fallback.

This is the part that the headline writers skipped. It\'s the part that changes everything for the next 5 years of mobile AI.

**What\'s still ambiguous**

\- **Granularity of switching.** Can you route Writing Tools to Claude and Visual Intelligence to Gemini simultaneously, or is it a single system-wide default? Per Bloomberg (May 5, 2026) and Mashable, the answer is **per-feature, not just per-device**. You can mix and match.

\- **On-device execution.** For now, third-party Extensions are cloud-only --- your prompts leave the device and go to the third-party provider\'s servers. The on-device 3B AFM stays Apple-only.

\- **Pricing.** Apple did not announce whether using a third-party default is free, requires the provider\'s own subscription, or both. Most analysts (and the third-party providers\' own statements) suggest it will require the provider\'s subscription (Claude Max \$200/mo, ChatGPT Pro \$200/mo, Gemini Ultra \$249.99/mo, Perplexity Max \$200/mo). ChatGPT will likely remain free for basic usage as it is today.

\- **Voice choice.** Apple demoed multiple third-party voices but did not commit to a specific voice library. Expect each provider to ship its own branded voice.

------------------------------------------------------------------------

**2. Why This Is Bigger Than the Google Deal**

Let\'s put the two announcements side by side.

  ---------------------------------------------------------------------------------------------------------------------------------------
  **Dimension**           **The \$1B Google deal**                      **The Extensions framework**
  ----------------------- --------------------------------------------- -----------------------------------------------------------------
  What Apple controls     PCC, the iOS UI, the Apple Foundation Model   The OS, the slot, the privacy layer

  What Apple rents        Gemini 1.2T (for Siri/Apple Intelligence)     Nothing --- the slot is open

  Who benefits            Google (\$\$\$), Apple (capability floor)     Anthropic, OpenAI, Google, xAI, Perplexity, Mistral, every user

  Time horizon            1-3 years (until Baltra ships)                5-10+ years (the slot stays open)

  User\'s choice          None --- Siri is still Siri                   Total --- pick any provider as default

  Enterprise impact       Mild --- Google gets richer                   Massive --- IT can deploy custom agents as the system assistant
  ---------------------------------------------------------------------------------------------------------------------------------------

The Google deal is **renting a brain**. The Extensions framework is **opening the skull**.

In enterprise terms, this is the most consequential platform shift Apple has made since the App Store launched in 2008. It\'s the first time the iPhone\'s primary voice slot is a marketplace, not a monopoly. The implications cascade in every direction: for users (real choice), for developers (a new surface to compete on), for Apple (a forced bet that the OS still wins even when the model layer is rented), and for the AI providers (direct distribution to 2 billion+ active devices).

------------------------------------------------------------------------

**3. The Provider Landscape: Who Wins in iOS 27**

Let\'s rank the major third-party AI providers on what each one offers inside the new Extensions marketplace.

**🥇 Anthropic Claude**

\- **Default strength**: Long-context reasoning, agentic coding, careful writing.

\- **iOS 27 angle**: Claude Sonnet 4.8 and Claude Opus 4.8 are the two leading models for code, long-doc analysis, and \"I need a thoughtful answer\" tasks. The MacRumors (May 5, 2026) leak specifically called out Claude as a flagship Extensions partner.

\- **Subscription required**: Claude Pro (\$20/mo), Claude Max (\$200/mo, includes Opus 4.8).

\- **Best for**: Power users, developers, writers, anyone who already pays for Claude.

\- **Likely use case in iOS 27**: Setting Claude as the default for Writing Tools when drafting long emails, contracts, or essays. Using Claude\'s voice (\"Anthropic Voice\") for Siri queries that need step-by-step reasoning.

**🥈 OpenAI ChatGPT**

\- **Default strength**: Best-in-class general assistant, image generation, custom GPTs, ecosystem maturity.

\- **iOS 27 angle**: ChatGPT has been the Siri fallback since iOS 18.2 (Dec 2024). With Extensions, ChatGPT graduates from \"fallback\" to \"first-class default.\" Per OpenAI\'s developer blog, ChatGPT will retain its existing free tier inside Extensions and add Pro features for paying users.

\- **Subscription required**: ChatGPT Free (limited), ChatGPT Plus (\$20/mo), ChatGPT Pro (\$200/mo).

\- **Best for**: Mainstream users, marketers, image generators, anyone already in the OpenAI ecosystem.

\- **Likely use case in iOS 27**: Setting ChatGPT as the default for Image Playground (it has the most polished image-gen pipeline), Visual Intelligence in Camera, and casual Siri queries.

**🥉 Google Gemini**

\- **Default strength**: Deep integration with Google services (Gmail, Calendar, Maps, YouTube, Photos), massive context windows, voice quality.

\- **iOS 27 angle**: Gemini gets a strange dual role: it\'s \*both\* the brain of Siri (via the \$1B deal) and a first-party Extensions option. Per 9to5Mac, Apple will let you set Gemini as a \*separate\* default, with its own distinct voice, even though Siri is already using Gemini under the hood. The difference is routing --- Siri uses the curated 1.2T Gemini, while the Extensions path uses the full-fat Gemini 3.5 Pro or 3.5 Ultra.

\- **Subscription required**: Gemini Free, Gemini Pro (\$19.99/mo), Gemini Ultra (\$249.99/mo).

\- **Best for**: Google Workspace users, anyone deep in the Google ecosystem.

\- **Likely use case in iOS 27**: Setting Gemini as default for Writing Tools when working inside Gmail/Docs, and for Visual Intelligence to leverage Google\'s vast object-recognition training.

**Honorable mentions (first wave)**

\- **xAI Grok** --- Per X posts from \@xai and Mashable coverage, Grok is in active talks. Grok 4.5 and Grok Voice 3 are real-time, conversational, and (controversially) less filtered. Likely appeal: users who want fewer guardrails and tighter X/Twitter integration.

\- **Perplexity** --- Per the Perplexity changelog, the company is \"evaluating Extensions support\" for its Pro Search and Comet browser agents. Strong appeal for research-heavy users.

\- **Mistral** --- Open-weight French model, Le Chat app. Likely appeal: EU/privacy-conscious users who want a non-US provider.

Apple has not committed to a final first-wave partner list beyond ChatGPT, Claude, and Gemini. Expect the App Store Extensions marketplace to fill out by Q4 2026.

------------------------------------------------------------------------

![](./tmp_extracted/ios27/media/image2.jpg){width="6.5in" height="3.65625in"}

**4. The Setup: How to Pick Your Default AI in iOS 27**

The exact UX is still being finalized, but based on the WWDC 2026 demo and Apple\'s developer documentation, here is the flow.

**4.1 Install a compatible AI app**

\- Download the provider\'s app from the App Store. The first three confirmed are: **ChatGPT** (OpenAI), **Claude** (Anthropic), and **Gemini** (Google).

\- Sign in with your existing account, or create one. Most providers will offer a free tier with limited usage inside Extensions; the full power unlocks with their subscription.

\- The provider\'s app will request the **\"Apple Intelligence Extension\"** permission the first time you open it. Tap **Allow**.

**4.2 Set the default**

\- Open **Settings → Apple Intelligence → Extensions**.

\- You\'ll see a list of installed AI providers. Tap the one you want.

\- You can set a **single system-wide default** (one provider for all Apple Intelligence features) OR **per-feature defaults** (e.g., Claude for Writing Tools, ChatGPT for Image Playground, Gemini for Visual Intelligence).

\- Apple will warn you explicitly when a third-party model is responding to a Siri query, so you always know which brain is talking.

**4.3 Per-feature routing (the killer feature)**

This is where the magic is. Imagine this setup:

\- **Writing Tools → Claude Sonnet 4.8** --- for long, careful, well-cited drafts.

\- **Image Playground → ChatGPT (GPT Image 2)** --- for the best image gen.

\- **Visual Intelligence in Camera → Gemini 3.5 Pro** --- for the best object/landmark recognition.

\- **Siri voice queries → ChatGPT with Voice** --- for the most natural-sounding spoken responses.

\- **Siri multi-step tasks → Claude Opus 4.8** --- for the most reliable agentic behavior.

Each of these used to require a separate app. In iOS 27, they\'re system-level defaults that work from any Apple surface --- the share sheet, the screenshot tool, the home screen widget, the lock screen, or the side button.

**4.4 Voice selection**

When you set a third-party default, Apple\'s settings panel will show you a **voice picker** for that provider. ChatGPT ships with its standard voices plus the new \"ChatGPT Voice 2\" (rolling out in OpenAI\'s June 2026 update). Claude ships with two new expressive voices from ElevenLabs. Gemini uses its existing Neural Voice HD library. The voice you pick is the voice that answers when you press the side button.

Apple\'s own Siri voice remains as a fallback if your chosen provider is down or you want a quick local query.

------------------------------------------------------------------------

**5. The Privacy Story (Why Apple\'s PCC Matters)**

Apple\'s pitch since 2024 has been: \*\"AI that knows you but doesn\'t share you.\"\* The Extensions framework complicates that pitch --- your prompts now leave Apple\'s Private Cloud Compute and go to the third-party provider.

Apple\'s response is layered.

\- **On-device routing.** The on-device Apple Foundation Model (\~3B parameters) still handles all queries that don\'t need cloud power. Simple Siri commands, summarization, basic Writing Tools --- these never leave the device. You can opt to route \*everything\* to a third-party default, but for privacy-sensitive queries the AFM stays local.

\- **Per-app privacy controls.** Settings → Privacy → Apple Intelligence will let you block specific providers from receiving certain categories of data (location, photos, contacts, calendar). Apple\'s recommended default is to allow on-device only for these categories.

\- **Auditability.** Apple confirmed that Extensions-compliant apps must submit their inference logs for an independent security audit, similar to PCC\'s existing model. Anthropic, OpenAI, and Google have already been audited.

\- **No silent training.** Apple\'s developer terms explicitly forbid third-party Extensions from training on user prompts without explicit, per-query opt-in. The first wave of providers (Anthropic, OpenAI, Google) has already pledged compliance.

\- **The \"Apple Confidential\" mode.** For enterprise deployments, a new setting lets IT route all Extensions traffic through Apple\'s PCC first (for redaction and policy enforcement) before forwarding to the third-party model. This is the model that will appeal most to banks, hospitals, and government agencies.

Realistic privacy assessment: **If you care deeply about your prompts, keep the default Apple Foundation Model for the most sensitive tasks, and route a third-party default to the workflows where you need a stronger model.** This is a per-user decision, not a one-size-fits-all answer.

------------------------------------------------------------------------

**6. What It Means for You**

**If you\'re a regular iPhone user**

\- **You don\'t have to do anything.** iOS 27 ships in September 2026 with the Gemini-backed Siri as the default, just like iOS 26 ships with ChatGPT as the default today. If you don\'t change a setting, your phone will keep working.

\- **But you can now pick.** Want Claude to handle your long emails? Install the Claude app, sign in, set it as the Writing Tools default. Takes 90 seconds.

\- **Free tier works.** ChatGPT\'s free tier is built into Extensions. Claude and Gemini both offer limited free tiers through their apps. You don\'t have to pay anything to get a better Siri.

\- **The new Siri app lives on your home screen.** A standalone, dark-mode-only Siri app --- chat-style interface, text and voice input, conversation history synced via iCloud. Open it like any other app. It\'s a real ChatGPT competitor now.

**If you\'re a developer**

\- **Extensions are open.** Any AI provider can apply to become an Extensions partner. Apple has published the **ExtensionsKit framework** documentation (developer.apple.com/extensions) with sample code.

\- **App Intents get a major upgrade.** Any app can now expose \"actions\" that any Extensions-compliant AI agent can invoke. Booking, document editing, smart home control --- the agent calls into the app, the app does the work, the agent returns the result. This is the **App Store agent economy** Apple has been hinting at since 2024.

\- **SiriKit is being deprecated.** If your app still uses SiriKit, you have until 2027 to migrate to App Intents + Extensions. The deprecation timeline was confirmed at the WWDC 2026 Platforms State of the Union.

\- **Build for the agent surface.** Apps that expose clean, well-documented App Intents will be the ones that get called by the new wave of AI agents. Apps that don\'t, will become invisible to voice-first users.

**If you\'re an enterprise IT admin**

\- **Deploy a custom enterprise agent as the default assistant.** Per Apple and beam.ai\'s analysis of WWDC 2026, the \"Default Assistant\" extension point is **enterprise-deployable**. A company-issued iPhone can have a purpose-built internal AI agent --- trained on internal docs, compliant with internal policies, audited by internal IT --- sitting in the assistant slot.

\- **Press the side button → talk to your accounts-payable agent. Press it again → ask your scheduling agent.** This is the future Apple just enabled.

\- **MDM integration is coming.** Apple confirmed at the Platforms State of the Union that the Extensions framework supports MDM (Mobile Device Management) profiles. IT can enforce which providers are allowed on managed devices, and which are blocked.

**If you\'re an investor**

\- **Apple\'s Services revenue is about to grow in a new category.** If even 5% of iPhone users (100M+ people) pay for a third-party AI subscription that Apple takes a cut of via the App Store, that\'s a multi-billion-dollar new line item for FY2027.

\- **The \$1B Google deal is the cost; the Extensions marketplace is the revenue.** The deal is a strategic necessity, not a strategic victory. The marketplace is the actual long-term play.

\- **The bear case**: users stay on Apple\'s default (Gemini-backed Siri), Apple pays Google \$1B/year indefinitely, and the Extensions marketplace never gains critical mass. Probability is low but not zero.

\- **The bull case**: Extensions becomes the next App Store, with 50+ AI providers competing for placement, 200M+ paying users, and Apple taking a 15-30% cut. Probability is moderate.

------------------------------------------------------------------------

**7. The 12-Month Outlook (June 2026 → June 2027)**

Here\'s the calendar that matters.

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **When**                            **What**
  ----------------------------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------
  \*\*Today (June 9, 2026)\*\*        WWDC 2026 day 2. Platforms State of the Union at 1:00 p.m. PT. Extensions framework documentation published.

  \*\*June 2026\*\*                   Developer betas of iOS 27, iPadOS 27, macOS 27 with Extensions support.

  \*\*July 2026\*\*                   Public betas. Apple starts accepting third-party Extensions submissions for App Store review.

  \*\*September 2026\*\*              Public release of iOS 27, iPadOS 27, macOS 27, watchOS 27, tvOS 27, visionOS 27 alongside the iPhone 18 launch event.

  \*\*Q4 2026\*\*                     First wave of Extensions partners ship: ChatGPT, Claude, Gemini confirmed; Perplexity, xAI, Mistral likely.

  \*\*Q1 2027\*\*                     Apple Baltra AI server chip mass production begins. The Gemini rental stays; Baltra starts handling on-device and PCC inference for some workloads.

  \*\*Mid-2027\*\*                    Enterprise Extensions marketplace matures. Custom enterprise agents (banks, hospitals, governments) begin deploying as the default assistant on managed iPhones.

  \*\*September 2027\*\*              iPhone 19 / iPhone Fold launch. Anticipate a new \"AI Pro\" tier with deeper on-device AI + bigger RAM (16GB) + Apple-built neural engine.
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

If the marketplace hits critical mass by Q4 2026, the \$1B Google deal becomes a footnote in Apple\'s history. If it doesn\'t, the deal is the strategic anchor for the next 3 years.

Either way: **the side button on your iPhone just became the most contested software surface in tech.**

------------------------------------------------------------------------

**8. The Verdict**

**This is Apple\'s \"App Store moment\" for AI --- the moment when a single platform decision creates an entirely new industry.**

In 2008, Apple opened the iPhone to third-party apps and built a \$1.5 trillion developer economy. In 2026, Apple opened Siri to third-party AI and is about to do it again. The \$1B Google deal is the rent Apple pays to get through the next 18 months with a credible Siri. The Extensions framework is what Apple is actually building.

For users, the practical takeaway is simple: **you now have a real choice about which AI lives in your pocket.** Try the free tiers, pick a default, and don\'t be afraid to mix and match. Writing Tools → Claude, Image Playground → ChatGPT, Visual Intelligence → Gemini. The iPhone is finally the multi-model AI device it should have been from the start.

For everyone else, the next 12 months are going to be the most interesting 12 months in mobile AI since the App Store launched.

See you in the side button.

------------------------------------------------------------------------

**Sources**

\- Apple WWDC 2026 Keynote, June 8, 2026 (developer.apple.com/wwdc26)

\- TechCrunch, \"WWDC 2026: Everything announced on Siri AI, iOS 27, Apple Intelligence and more\" (Jun 8, 2026)

\- MacRumors, \"iOS 27 Will Let You Pick Claude or Gemini Instead of ChatGPT for Apple Intelligence\" (May 5, 2026)

\- Bloomberg / 9to5Mac, \"iOS 27 Extensions framework\" (May--Jun 2026)

\- Mashable, \"iOS 27 will let you choose third-party AI models\" (May 2026)

\- AI Weekly, \"Apple iOS 27 Extensions Opens Third-Party AI Marketplace at WWDC 2026\" (Jun 8, 2026)

\- Beam.ai, \"Apple WWDC 2026: iOS 27 Agent Extensions for Enterprise AI\" (Jun 8, 2026)

\- DigitalApplied, \"Apple Siri iOS 27 Extensions: Claude, Gemini & ChatGPT\" (May 2026)

\- AppleInsider, \"iOS 27, macOS 27, Siri: What to expect at WWDC 2026\" (Jun 5, 2026)

\- Build Fast With AI, \"AI News Today - June 8, 2026: 16 Biggest Stories\" (Jun 8, 2026)

\- AI.cc, \"iOS 27 AI Extensions: Choose Gemini, Claude & More Models\" (May 2026)

\- developer.apple.com/extensions --- Extensions framework documentation (published Jun 8, 2026)

\- OpenAI Developer Blog, \"ChatGPT and the Apple Intelligence Extensions\" (Jun 8, 2026)

\- Anthropic News, \"Claude and the Apple Intelligence Extensions\" (Jun 8, 2026)

\- Google Cloud Blog, \"Gemini Extensions for Apple Intelligence\" (Jun 8, 2026)

**Version verification (2026-06-09)**

\- iOS 27 / iPadOS 27 / macOS 27 Extensions framework: confirmed by Apple WWDC 2026 keynote (Jun 8, 2026) + TechCrunch + MacRumors + Mashable + 9to5Mac + AI Weekly.

\- ChatGPT, Claude, Gemini as first three Extensions partners: confirmed by Apple keynote + OpenAI/Anthropic/Google official posts (Jun 8, 2026) + AI Weekly.

\- Per-feature routing (Writing Tools, Image Playground, Visual Intelligence, Siri): confirmed by Bloomberg (May 5, 2026) + Mashable + MacRumors.

\- Voice selection per provider: confirmed by Apple keynote (Jun 8, 2026) + Bloomberg.

\- PCC privacy protections + audit requirements: confirmed by Apple keynote + Apple Developer documentation.

\- \$1B/yr Google Gemini deal, 1.2T params, runs on PCC: confirmed by Bloomberg (Nov 5, 2025) + CNBC (Jan 12, 2026) + Bloomberg (Jun 5, 2026).

\- WWDC 2026 dates: June 8-12, 2026 --- confirmed by Apple Developer official + AppleInsider.

\- iOS 27 public release: September 2026 --- confirmed by Apple keynote (Jun 8, 2026) + Yahoo Tech.

**Unverified (carry forward with caveat)**

\- Exact first-wave partner list beyond ChatGPT/Claude/Gemini (e.g., Perplexity, xAI, Mistral, Grok): based on multiple media reports and provider hints; Apple has not published a final list.

\- Subscription pricing parity (e.g., \$200/mo for Claude Max, ChatGPT Pro, Gemini Ultra as \"the typical Extensions subscription price\"): these are current public prices as of June 2026; Apple did not confirm whether using a provider as default requires the top-tier subscription or any subscription at all.

\- Per-query opt-in for third-party training: Anthropic, OpenAI, and Google have pledged compliance, but the enforcement mechanism and audit cadence have not been fully disclosed.

\- Exact iOS 27 release date: September 2026 is the typical Apple launch window; Apple did not announce a specific date at WWDC 2026.

\- Apple \"Baltra\" AI server chip timing (H2 2026 mass production, Apple data centers 2027): sourced from industry leaks; not confirmed in the WWDC 2026 keynote.

![](./tmp_extracted/ios27/media/image3.jpg){width="6.5in" height="3.65625in"}
