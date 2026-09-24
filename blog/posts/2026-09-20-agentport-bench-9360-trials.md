Title: Does Your Agent Framework Choice Actually Matter for Security? We Ran 9,360 Trials to Find Out.

Author: Waqar Javed
Published: September 20, 2026
Canonical URL: https://agentsafelabs.com/blog/does-your-agent-framework-choice-actually-matter-for-security-we-ran-9360-trials-to-find-out/

---

A controlled, payload-verified evaluation of seven agentic frameworks says: mostly no — with one small, honest exception.

If you're building a tool-using LLM agent today, you're choosing among a genuinely crowded field: LangChain, CrewAI, AutoGen, LlamaIndex, the OpenAI Agents SDK, Google's Agent Development Kit, Semantic Kernel, or just talking to the model's API directly and rolling your own orchestration. That's a real decision with real engineering tradeoffs — but does it also change how safe your agent is against adversarial input?

We built AgentPort-Bench to answer that question properly, and the honest answer is more interesting than a simple yes or no.

## The confound nobody controls for

Here's the problem with most cross-framework security comparisons: when you run "the same attack" through two different frameworks, you're not actually holding the attack constant. Every framework templates prompts differently, structures message roles differently, and serializes tool schemas differently. If Framework A shows a different attack-success rate than Framework B, you genuinely don't know whether that's because the framework changed the model's behavior, or because the adapter quietly rephrased the attack on its way to the model.

We found out how real this risk is the hard way. During our own data collection, we discovered CrewAI's default agent construction didn't use our shared baseline system prompt at all — it built one from separate role/goal/backstory fields, and the resulting text included the word "safely" that appeared nowhere in any other condition's prompt. That's not a subtle difference. Fixing it and re-collecting the affected trials measurably shifted the results (a statistically significant swing in the pass rate). If we hadn't verified payload identity byte-for-byte across every single condition, we would have reported a "framework effect" that was actually an adapter bug.

That's the whole reason this evaluation exists: you can't ask whether frameworks differ in safety impact until you've made sure they're actually receiving the same attack.

## The actual experiment

Once payload identity was verified, we ran a controlled comparison across:

- **Eight execution conditions**: a direct-API baseline plus seven agentic frameworks
- **Six models** spanning three providers and two capability tiers
- **Five attack families** mapped onto OWASP's Agentic Security Initiative threat taxonomy
- **9,360 total trials**, combining an original six-condition study with a later two-framework extension into one unified dataset — analyzed jointly here for the first time

## What actually explains the outcome

Attack category dominates. Which model you're using matters, but less. Which framework you're using barely registers at all — about two orders of magnitude smaller an effect than attack category, and about one order of magnitude smaller than model choice.

That much, a classical statistical test could already tell us. But a non-significant result only tells you "we couldn't detect a difference" — not "the difference is small enough not to matter in practice." Those are different claims, and conflating them is a common way papers overstate a null result.

So we didn't stop at the classical test. We also ran a formal equivalence test — the kind of statistical tool designed specifically to answer "is this effect small enough to call practically negligible," with the acceptable margin decided before looking at the results, not after. Every single one of the 28 possible pairwise comparisons between our eight conditions came back equivalent under that pre-specified margin.

That's a genuinely stronger claim than "we found no significant difference," and we think it's the more useful one for anyone actually deciding which framework to build on.

## The one honest exception

We could have stopped there and called it a clean story: framework choice doesn't matter, full stop. We didn't, because the data doesn't quite support that.

CrewAI — even after fixing the system-prompt bug described above — retains a small, statistically real residual effect. It's not large: it clears the equivalence bar we set, and it's the single closest pairwise comparison to that bar, not the only one that failed it. It shows up uniformly against every other condition, old and new alike, not specifically against the two frameworks we added later. Our best guess — genuinely a guess, not a tested finding — is that CrewAI's role/goal/backstory prompt structure can't be delivered as a single message the way every other condition can, even once you match its content. We flag that as the natural next question, not as something we've confirmed.

We think reporting this precisely — real, small, specific, not inflated into "CrewAI is unsafe" and not smoothed away into "no effect anywhere" — is more useful than either a falsely tidy or a falsely alarming headline.

## Two things you'd only find if you went looking

Along the way we ran into two practical gotchas worth knowing if you're building on these frameworks yourself:

***A fully deterministic reasoning-token-exhaustion failure***

One frontier model, on one specific prompt, silently consumed its entire token budget on invisible internal reasoning and returned nothing — 6 out of 6 times, across two different frameworks. If your evaluation harness treats an empty response as a simple failure-to-respond rather than checking whether the model just burned its whole budget thinking, you'll misdiagnose this.

#### A token-accounting trap

Google ADK and Semantic Kernel report reasoning-token usage under genuinely different conventions for the same underlying model — one folds reasoning tokens into the total output count, the other reports them separately. Compare the two frameworks' own usage numbers directly, without correcting for this, and you'll manufacture a cost difference that doesn't exist.

## The bottom line

If you're choosing an agent framework based on security concerns alone, this data says: don't. Put your effort into attack-surface coverage and model selection instead — that's where the real variance lives. Framework choice is, for the eight conditions and five attack families we tested, close enough to a non-issue that we can say so with a formal statistical test behind it, not just a shrug.

The one exception is small enough that it shouldn't change your framework decision on its own — but it's real enough that if you're running CrewAI in production, it's worth knowing about.

---

Full methodology, the complete 9,360-trial dataset provenance, and the statistical models are documented in our paper, "AgentPort-Bench: A Controlled Seven-Framework Evaluation of Agentic AI Security Portability," currently under review.

*The evaluation framework is open source at github.com/AgentSafeLabs/safelabs-eval, and the AgentPort-Bench contributor submission repository is live at github.com/AgentSafeLabs/agentport-bench-results if you'd like to submit your own framework's results.*
