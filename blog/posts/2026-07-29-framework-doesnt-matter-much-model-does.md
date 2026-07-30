Title: Framework Doesn't Matter (Much). Model Does.

Author: Waqar Javed
Published: July 29, 2026
Canonical URL: https://agentsafelabs.com/blog/framework-doesnt-matter-much-model-does/

---

We ran the largest cross-framework agentic security evaluation we know of — 6 models, 6 execution conditions, 5 attack families, 7,020 trials — and the headline result surprised us: which orchestration framework you use barely moves the needle on security outcomes. Model choice and attack type dominate. Framework explains about 0.06% of the variance.

That's not what the existing literature said going in.

## The question we started with

Everyone building agentic AI right now is choosing between LangChain, CrewAI, AutoGen, LlamaIndex, the OpenAI Agents SDK, or just calling the model directly. A natural question follows: does that choice matter for security? If a model is safe under one framework, is it safe under another?

The most direct prior attempt to answer this — Nguyen & Husain's comparative penetration test across AutoGen and CrewAI — found a large gap: 52.3% attack refusal under AutoGen versus 30.8% under CrewAI, holding the model constant. That's a big number. If it's real, it means security teams need to re-test every framework independently, every time.

But there's a problem with taking that number at face value: frameworks don't just run your model, they rewrap what reaches it. System prompts get templated differently. Message roles get restructured. Tool schemas get reformatted. None of that is "the framework's security posture" in any meaningful sense — it's plumbing. And if two frameworks happen to phrase the same attack differently by the time it reaches the model, you can get a large measured gap that has nothing to do with the framework's actual defenses.

Nobody had checked.

## What we did differently

Before treating any cross-framework comparison as real, we verified — byte for byte — that the exact same payload was reaching the model in every condition. Not "should be the same." Actually diffed, per model, per framework, before scoring anything.

This wasn't a formality. It caught a real bug.

CrewAI's agent-construction code wasn't using our shared system prompt at all — it was building a persona from separate fields, and the resulting prompt included the word "safely" that appeared nowhere else in the test matrix, and was roughly three times longer than every other condition's prompt. Once we found it, we could measure exactly what it was doing to our numbers: PASS rate shifted from 66.4% to 61.3% after we fixed it and re-ran the affected trials (p = 0.0098 — not noise). If we hadn't checked, that confound would have looked exactly like a "CrewAI is less safe" finding. It wasn't. It was a plumbing bug.

We found one more asymmetry along the way: two of our six conditions weren't sending a system prompt at all for a stretch of the project, which mechanically made system-prompt-leakage attacks look impossible to succeed against them — not because those conditions were more secure, but because there was no prompt to leak. We fixed it, and excluded the tainted window from that specific analysis rather than quietly blending it in.

## What we found once the plumbing was clean

Once payload identity was actually verified, we ran a full variance decomposition — 6 models (three "cheap," three "frontier," spanning Anthropic, OpenAI, and Google), 6 execution conditions, 5 attack families, 7,020 trials.

- Attack family: 28.67% of outcome variance
- Model: 4.23%
- Framework: 0.06%
- Residual + interactions: the rest

A permutation test on the framework term came back at p = 0.70 — indistinguishable from random assignment. We checked this against an alternate way of scoring outcomes, in case the result was an artifact of how we weighted things: framework's share moved from 0.06% to 0.56%. Still nowhere close to attack family or model.

In plain terms: what you're attacking and which model you're attacking matter enormously. Which orchestration framework is sitting in between barely registers.

## The number that doesn't hold up

We also built a secondary, more granular metric — a per-model score for how consistently a model's security behavior holds across frameworks (we call it CFPS). The average across all six models was 0.1132, which reads as reasonably tight.

But here's the catch, and we think it's the more important part of this post: when we re-scored that same per-model ranking under the alternate weighting scheme, the ranking barely correlated with the original (Spearman r = 0.31). The framework-share finding survived the reweighting. The fine-grained per-model ranking didn't.

We're reporting both, not picking the one that looks better. The coarse finding — framework doesn't matter much — is solid. The fine per-model ranking is fragile. If you only read the CFPS number and not the caveat next to it, you'd walk away more confident than the data supports.

## What we'd tell someone building on this today

If you're deciding how to spend limited red-teaming time across a multi-framework agentic product, this data says: don't burn your budget re-testing the same model across five orchestration frameworks expecting to find framework-specific holes. Spend it on model selection and attack-surface coverage instead — that's where the actual variance lives.

That's a bounded claim, not a universal one. Six conditions, five attack families, one evaluation design. We're not claiming this generalizes to every framework or every deployment shape — just that, for what we tested, framework was the wrong place to look.

---

Full methodology, the confound writeups, limitations, and the complete results tables are in the paper: *Cross-Framework Portability of Agentic AI Security: A Controlled, Payload-Verified Evaluation* — DOI: 10.6084/m9.figshare.33110642 (https://doi.org/10.6084/m9.figshare.33110642)
