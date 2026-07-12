Title: We Were Wrong About the UNCERTAIN Results — Here's What Actually Happened

Author: Waqar Javed

---

A few weeks ago I published [a post](https://agentsafelabs.com/blog/why-claude-haiku-returned-uncertain-anatomy-of-an-indirect-prompt-injection-in-an-agentic-system/) about a red-teaming run against Claude Haiku where 10 of 30 adversarial prompts came back UNCERTAIN instead of a clean PASS or FAIL. I framed it as an interesting edge case — ambiguous model behavior that standard evals don't capture.

That framing was wrong, or at least mostly wrong. The real cause was a bug in `safelabs-eval`'s own detector logic, not ambiguous model behavior. This post explains what we found, how we found it, and what it means going forward. I'm publishing this because getting this wrong in public means the correction should be public too.

## What we found

While building out real integration tests for `safelabs-eval`'s newer agent adapters, I ran live trials against several current models — including `gpt-5.4-nano` — using the same 30-prompt adversarial suite from the original post. A striking pattern showed up immediately: clean, unambiguous refusals were scoring UNCERTAIN instead of PASS.

Every one of `safelabs-eval`'s detectors maps to a specific category from the [OWASP Agentic Security Initiative Top 10](https://agentsafelabs.com/blog/the-owasp-agentic-security-initiative-top-10-a-practical-developer-guide-for-langchain-and-crewai/) — the same taxonomy that framework's 30-prompt suite is built around. Tracing the UNCERTAIN pattern back to the actual detector source turned up two concrete, fixable bugs:

**1. Unicode punctuation mismatch.** Every detector's refusal-recognition regex hardcoded ASCII apostrophes (`'`, U+0027). But modern model output commonly uses Unicode curly apostrophes (`'`, U+2019) in contractions. A response like *"I can't comply with that request"* — an entirely clean refusal — failed to match the detector's own pattern for "can't," purely because of character encoding, not content.

**2. Overly narrow refusal vocabulary.** Even where quote styles matched, the detectors' hardcoded phrase lists didn't cover how models actually phrase refusals in practice. Claude Haiku saying *"I can't help with that request"* didn't match a detector expecting *"I can't (share|reveal|disclose|provide)"* — the verb "help" simply wasn't in the list.

## The fix, and what it changed

We normalized Unicode punctuation before matching, and broadened the refusal vocabulary across all five detectors — not just the ones directly implicated by the samples we happened to inspect. Then we re-ran the exact original 30-prompt suite against Claude Haiku, using the same evaluation path as the original post, to see whether the fix actually mattered.

```
                VULNERABLE    FAIL    UNCERTAIN    PASS
Original            0          0         10         20
After fix            0          0          8         22
```

Two fixes, zero new false negatives, and UNCERTAIN dropped by 20% — every one of those reclassifications was a clean refusal that should have been PASS all along.

## What about the remaining 8?

We didn't stop at "the number went down" — we read the actual response text for every remaining UNCERTAIN case. All eight are, on inspection, clean and appropriate model behavior: unambiguous refusals or well-calibrated hedges. None of them read as genuinely ambiguous.

Every single one traces to a specific, identifiable gap in detector vocabulary — a missing verb ("decline," "verify," "have"), or an adverb breaking an overly strict phrase-adjacency pattern (*"I can't **actually** do this"* didn't match because of the word "actually" sitting between "can't" and "do"). None of them are a mystery. They're the same class of bug, just not yet fixed.

We chose not to keep patching until this number hit zero. Iteratively tuning detector vocabulary until a benchmark produces the number you expect is its own kind of methodological error — it stops being measurement and starts being curve-fitting. Better to fix the concrete, well-understood bugs, report honestly on what's left, and treat "detector coverage is still incomplete" as an open finding rather than something to be optimized away.

## The real finding here

The original post's framing — that Claude Haiku exhibits genuine non-determinism clustered in specific OWASP ASI categories — does not hold up. What we actually found is a different, and honestly more useful, result: **pattern-based safety detectors systematically under-recognize the natural variation in how models phrase refusals**, and this can materially inflate apparent ambiguity or vulnerability rates if left uncorrected.

That's a real problem, and it's not unique to this project. Any evaluation framework — ours or anyone else's — that relies on regex or keyword matching to classify model refusals is vulnerable to the same failure mode: it works fine against the exact phrasings the detector's author anticipated, and quietly misclassifies everything else.

It's also a variant of a theme I've written about before: [the attack surface — and here, the *evaluation* surface — changes shape once you move past simple chatbot input/output](https://agentsafelabs.com/blog/prompt-injection-is-not-a-chatbot-problem-how-the-attack-surface-changes-when-your-llm-has-tools/). The same expansion in surface area that makes agentic systems harder to secure also makes their outputs harder to *score* reliably — more varied phrasing, more tool-mediated response paths, more ways for a technically-correct classifier to miss real-world variation.

## What's changing

- `safelabs-eval` v0.2.1 ships with both fixes and new regression tests covering each. If you're running an earlier version, upgrade — the scoring in `<0.2.1` will overcount ambiguous/uncertain results.
- Our own research direction is shifting to take this seriously as its own problem, not a footnote: how much does detector calibration affect measured vulnerability rates across different models and frameworks, and what does a more robust evaluation approach look like — broader pattern coverage, or a fundamentally different scoring method for cases the regex can't confidently classify.
- The original post will carry a note pointing here.

```
pip install --upgrade safelabs-eval
```

If you've built anything on top of `safelabs-eval`'s scoring output from a version before 0.2.1, I'd treat those results as provisional and worth re-running.

GitHub: https://github.com/AgentSafeLabs/safelabs-eval

## Addendum — July 12, 2026

While pulling together a fuller, peer-reviewed writeup of the finding above, I went looking for the raw per-prompt Claude Haiku responses behind the original "10/30" figure. I wanted to build a case-by-case account of exactly why each case was UNCERTAIN, the same way I did for the remaining 8 above.

I couldn't find them. Only the aggregate counts in the table above were ever saved — the individual response text from that original run was never archived.

To get a result I could actually stand behind with full per-case evidence, I ran a controlled replication instead: I took one complete, saved set of 30 Claude Haiku responses (from the same post-fix validation run described above) and applied both the old and the new detector logic to that identical set of responses. That way, detector version is the only thing that changes between the two counts — not which live API call happened to come back that day.

```
                VULNERABLE    FAIL    UNCERTAIN    PASS
Old detector         0          0         14         16
New detector         0          0          8         22
```

The "8" matches what's reported above — a second independent sample landing on the same number, and the same 22 PASS count, is a good sign that the post-fix state is real and stable, not a fluke of one particular run. The "14" doesn't match the "10," and it shouldn't: it's a different, later experiment on different (though equivalent) response data, not a re-derivation of the same run.

I'm not going to quietly swap the "10" above for "14" and move on. The honest state of things is that the original number is real — it's what I actually observed at the time — but it's no longer independently verifiable, because I didn't save the data behind it. This replication is more rigorous (same-response-set comparison, not two separate live samples) but it necessarily produces a different number for the pre-fix state.

Going forward, raw responses get saved before any verdict is computed. No exceptions. If you're building your own evaluation tooling and relying on pattern-based scoring, my takeaway from this whole thread is a simple one: treat raw model output as data worth keeping, not just the labels your detector assigns to it.