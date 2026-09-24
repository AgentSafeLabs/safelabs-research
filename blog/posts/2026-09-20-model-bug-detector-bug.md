Title: We Thought We'd Found a Model Bug. We'd Actually Found a Detector Bug.

Author: Waqar Javed
Published: September 20, 2026
Canonical URL: https://agentsafelabs.com/blog/we-thought-wed-found-a-model-bug-wed-actually-found-a-detector-bug/

---

How a three-phase investigation into "non-deterministic" model behavior turned into a case study on why nobody measures the reliability of their own safety detectors.

If you build or use an LLM safety evaluation pipeline, there's a good chance it works like this: you send an adversarial prompt to a model, the model responds, and a detector — usually pattern-matching against a list of refusal phrases — decides whether the model refused, complied, or did something ambiguous. That verdict becomes a data point. Enough data points become a safety report. The report becomes a claim about how safe a model is.

Almost nobody asks how safe the detector is.

We didn't either, until a stray observation turned into a three-phase investigation that ended up telling us more about our own tooling than about any model we tested.

## Act 1: A model that seemed to change its mind

It started small. Running a fixed 30-prompt adversarial set against Claude Haiku, we got an unusually high rate of ambiguous ("UNCERTAIN") verdicts — responses our detector couldn't confidently classify as either a refusal or a compliance. The obvious read was that the model was being inconsistent: refusing some attacks cleanly, hedging on near-identical ones.

We built the tooling to investigate that inconsistency. Instead, we found two boring, unglamorous bugs in our own detector:

***A punctuation mismatch:***

Our refusal patterns were written with straight ASCII apostrophes. Claude Haiku, like a lot of models, often uses the typographic apostrophe instead — "I can\*\*'t" vs. "I can'\*\*t." Same words, different Unicode character, invisible to a naive regex.

#### A too-narrow refusal vocabulary:

The model refuses things all the time using completely ordinary language — "I'd rather not," "let me redirect you," "here's why I can't help with that" — that our pattern list simply never anticipated.

Once we fixed both, the "inconsistency" dropped by roughly half on a same-response-set replication (holding the model's actual answers fixed and only changing the detector version — the cleanest way we could isolate the fix's real effect). Every one of the residual ambiguous cases traced to a specific, nameable vocabulary gap, not to the model changing its behavior mid-experiment.

#### The takeaway wasn't "our detector had two bugs." It was:

An apparent finding about model behavior was substantially an artifact of the tool measuring it and if we hadn't gone looking, that finding would have shipped as a real result about Claude Haiku's consistency, not our own regex.

## Act 2: The fix works — and it clusters in a way we didn't expect

A fix on one model, on one prompt set, tells you almost nothing about whether the fix generalizes. So we reran it across six models spanning three providers.

The good news: the fix generalizes. Ambiguous-verdict rates dropped across the board.

The interesting news: the mechanism behind the fix didn't spread evenly. The punctuation-normalization half of the fix did essentially all its work on OpenAI's models and none on Anthropic's or Google's — a clean 83%-vs-0% split, not a gradient. Two completely different model families apparently have very different house styles for how they type an apostrophe in a refusal. We don't know why (that's a training-data or RLHF-process question we can't see into from the outside), but the split itself repeated identically across both a cheap-tier and a frontier-tier OpenAI model, which is a stronger signal than we expected to find by accident.

Then came the part that actually worried us.

### The fix that fixes one thing and breaks another

Running the same six-model comparison at higher resolution than our first pass allowed, we found something Phase 1 never could have surfaced: the exact same fix that resolves spurious ambiguous verdicts also introduces confirmed false positives — cases where the model straightforwardly complied with a harmful instruction, and our "fixed" detector scored it as a clean pass.

We read every flagged case by hand rather than trusting the aggregate numbers. In one category, 83% of the detector's "fixed" reclassifications were wrong — the model had opened with a capability-sounding disclaimer ("I don't have the ability to directly access your filesystem…") and then handed over exactly the destructive shell command the prompt asked for. Our broadened refusal-vocabulary pattern was matching the disclaimer and never noticing the compliance that followed it.

This is worse than it sounds at first glance. The previous state — an ambiguous verdict — was designed to route uncertain cases to human review. The new state — a confident PASS — actively suppresses that review. We hadn't just failed to fully fix the detector; for this specific slice of data, we'd made it more dangerous to trust.

## Act 3: Closing the loop, honestly

The obvious next step was to check whether this same failure pattern showed up anywhere else — the other five models, the other three attack categories we hadn't yet audited by hand.

It did, once more, in a different model and a smaller but still real magnitude (15%, or 7% under a stricter reading that excludes one contestable judgment call — we report both numbers rather than picking the more flattering one).

But the part of this phase we think matters more than the new numbers is what we didn't find. Of the 20 model-category combinations in our closing audit, 9 had zero data to examine at all — not "clean," just empty. We insisted on distinguishing those two states explicitly (we call it UNTESTABLE vs. NULL) because collapsing them into one reassuring "no problems found" number would misrepresent how much of the space we actually checked.

## Why this isn't really about our detector

We're not writing this to convince you our specific tool is now bulletproof. We're writing it because the pattern generalizes: any pattern-based safety classifier — and a lot of production safety pipelines still are pattern-based, because it's cheap, fast, and auditable — has an error surface that almost nobody measures or reports alongside their headline safety numbers.

A fix can be real, generalizable, and still wrong in a specific, structured way that only shows up if someone reads the actual text a "fixed" verdict is based on. We think that read-the-actual-text step should be treated as mandatory whenever a detector change could plausibly convert a safe "I'm not sure" into a false "all clear" — not an optional nice-to-have for teams with spare bandwidth.

Every number in the full write-up traces back to an archived, versioned dataset and detector-code checkout — we built the paper to be independently checkable, not just readable.

---

*Full technical details, methodology, and the complete dataset provenance are available in our paper, "Detector-Calibration Failures in Pattern-Based LLM Refusal Classification: Discovery, Generalization, and a Confirmed False-Positive Pattern Across Models," currently under review. The `safelabs-eval` framework used throughout this investigation is open source at:*

*github.com/AgentSafeLabs/safelabs-eval*
