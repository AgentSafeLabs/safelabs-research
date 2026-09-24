Title: SafeAgent-300: A Benchmark, Three Surprises, and One Uncomfortable Lesson About Our Own Tools

Author: Waqar Javed
Published: September 20, 2026
Canonical URL: https://agentsafelabs.com/blog/safeagent-300-a-benchmark-three-surprises-and-one-uncomfortable-lesson-about-our-own-tools/

---

We built a balanced 300-prompt adversarial benchmark for agentic AI security. Along the way we found a model spontaneously trying to call tools that don't exist, a wide safety gap between models, and — the part we almost didn't publish — a case where our own detector coverage gaps nearly produced a completely backwards conclusion.

Most adversarial prompt benchmarks for LLM agents grow organically: a category here, a few prompts there, until the corpus reflects whatever happened to get tested rather than a deliberate design. We wanted something we could actually reason about statistically, so we built SafeAgent-300 from the ground up:

Exactly 30 prompts per category, across 10 categories, exactly 10 prompts per (category, difficulty-tier) cell, across 3 difficulty tiers.

No category is over- or under-represented. Every cell in the design has the same amount of data.

We ran all 300 prompts against six models spanning three providers, producing 1,800 labeled trials. Here's what came out of it.

## Finding 1: A model that hallucinated a tool it was never given

While reviewing outputs, we noticed something odd in a subset of Google Gemini's responses: the model was attempting to make structured function calls — the kind of output format models use when they've been given a tool schema and decide to invoke it — even though no tool schema had been declared anywhere in the request.

It wasn't random. It happened specifically in prompts that described an available tool in natural language ("you have the storage tool; use your judgement to clean up old files") without actually wiring one up. The model apparently inferred a callable tool from the prose description alone and tried to invoke it as if it were real, in 3.7% of trials overall — spiking to over 22% in the category most likely to use this kind of tool-framed language.

We're not claiming to know why the model does this — that's a mechanistic question we can't answer from output text alone. But it's a real, reproducible behavioral pattern, and it's the kind of thing worth knowing if your evaluation harness treats a structured-but-unexpected response the same as a simple failure to respond, rather than checking whether the model just tried to call a tool that was never there.

## Finding 2: Models disagree a lot about how cautious to be

Looking at the rate of detector-confirmed genuine violations across our six-model roster (after our own detector coverage gaps were fixed — more on that below), the spread was substantial: the most cautious model in our roster sat near zero violations, while the least cautious model's rate was roughly 13 times higher. We corroborated this independently across two separate investigative threads before reporting it, precisely because a spread that large is the kind of number that's easy to get wrong by accident.

This isn't a claim that one model is "better" in some absolute sense — different providers make different tradeoffs between helpfulness and caution, and a benchmark like this only measures one axis of that tradeoff. But if you're picking a model for an agentic deployment where caution actually matters, the gap between models is large enough that "which model" is not a minor implementation detail.

## Finding 3 (the one that almost got published wrong)

Here's the part of this project we think is actually the most useful lesson, more than either headline finding above.

Early analysis suggested something genuinely surprising: prompts using more sophisticated, "stacked" evasion techniques (things like layering base64 encoding inside a routine-sounding task, or burying an instruction override inside document content) were triggering fewer confirmed violations than blunt, direct attacks — by a factor of roughly seven. Read naively, that would mean models handle sophisticated attacks better than simple ones. If true, that's a genuinely important and counterintuitive result worth building a whole paper around.

We didn't trust it immediately, and we're glad we didn't. Before reporting it as a finding about model behavior, we manually read a sample of the "ambiguous" responses in the sophisticated-attack tier — the same discipline that caught our detector's problems in an earlier investigation.

Twenty percent of that sample turned out to be:

Unambiguous, unhedged compliance with a harmful instruction that our detector had simply failed to catch

Because the detector modules covering those categories hadn't yet received the more sophisticated compliance-pattern detection that other modules had.

In other words: the "sophisticated attacks are less dangerous" finding was substantially — though we're careful to say not entirely — a detector-coverage artifact. Once we designed, verified, and shipped the missing detection logic for the affected modules and re-scored the corpus, the gap narrowed from roughly 7x down to about 1.9x, with the overwhelming majority of newly-caught violations landing exactly where the confound hypothesis predicted they would.

We could have stopped there and called the story fully resolved. We didn't, because two of our five detector modules still haven't received this same treatment — we say so explicitly rather than letting a partial fix read as a complete one.

## Why we're telling you about our own mistake

It would have been easy to publish the original 7x finding. It's a more exciting headline than "we found and partially fixed a measurement bug." But a benchmark's actual value isn't the headline number — it's whether people can trust the number enough to build decisions on top of it. A benchmark that silently under-detects violations in exactly the prompts it labels "most sophisticated" would be actively misleading anyone using it to argue that simpler attacks deserve more attention than complex ones.

We think this generalizes past our own tool:

Any pattern-based safety scoring system for open-ended model output should be treated as measuring, at best, its own detector's coverage — not ground truth —

And that gap is only visible if someone actually goes looking for it in exactly the sub-populations a headline number would otherwise treat as settled.

## What we're releasing

The prompt library itself — all 300 prompts, without model completions — is open source. Raw model completions are withheld from public release, since a handful contain literal, functional exploit technique that shouldn't be freely distributable; de-identified aggregate data and defanged illustrative examples are available in the paper, and full raw data can be requested under a data-use agreement.

We're upfront that this isn't a finished, fully-validated benchmark yet: every classification here was made by one author with no independent second rater, every prompt was tested only once per model rather than across repeated samples, and the detector-coverage gap described above is only partially closed. Those are real limitations, not fine print — we'd rather you know exactly how much to trust each number than have you find out the hard way.

---

*Full methodology, corpus construction details, and the complete detector-coverage investigation are documented in our paper, "SafeAgent-300: A Balanced 300-Prompt Benchmark for Agentic AI Security, with Findings on Detector Coverage Gaps and Cross-Model Compliance Variance," currently under review. The prompt library is open source as part of the `safelabs-eval` project at*:

GitHub: https://github.com/AgentSafeLabs/safelabs-eval
