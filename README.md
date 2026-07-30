# safelabs-research

Safe Labs AI research — attack taxonomy, advisories, and blog posts on
AI agent security. This repo mirrors what's published live on
[agentsafelabs.com/blog](https://agentsafelabs.com/blog/), giving each
post a version-controlled, timestamped provenance record independent
of the CMS.

**Live site is the canonical published version.** Files here are
archival copies + drafts, kept in sync manually — if a live post is
edited on WordPress, the corresponding file here should be updated in
the same commit cycle to avoid drift.

---

## Posts

| Post | Published | Internal copy | Live |
|---|---|---|---|
| Why Claude Haiku Returned UNCERTAIN: Anatomy of an Indirect Prompt Injection in an Agentic System | Jun 5, 2026 | [blog/posts/2026-06-05-why-claude-haiku-returned-uncertain.md](blog/posts/2026-06-05-why-claude-haiku-returned-uncertain.md) | [live ↗](https://agentsafelabs.com/blog/why-claude-haiku-returned-uncertain-anatomy-of-an-indirect-prompt-injection-in-an-agentic-system/) |
| The OWASP Agentic Security Initiative Top 10: A Practical Developer Guide for LangChain and CrewAI | Jun 5, 2026 | [blog/posts/2026-06-05-owasp-asi-top-10-guide.md](blog/posts/2026-06-05-owasp-asi-top-10-guide.md) | [live ↗](https://agentsafelabs.com/blog/the-owasp-agentic-security-initiative-top-10-a-practical-developer-guide-for-langchain-and-crewai/) |
| Prompt Injection Is Not a Chatbot Problem: How the Attack Surface Changes When Your LLM Has Tools | Jun 7, 2026 | [blog/posts/2026-06-07-prompt-injection-not-a-chatbot-problem.md](blog/posts/2026-06-07-prompt-injection-not-a-chatbot-problem.md) | [live ↗](https://agentsafelabs.com/blog/prompt-injection-is-not-a-chatbot-problem-how-the-attack-surface-changes-when-your-llm-has-tools/) |
| We Were Wrong About the UNCERTAIN Results — Here's What Actually Happened | Jul 11, 2026 | [blog/posts/2026-07-11-correcting-uncertain-results.md](blog/posts/2026-07-11-correcting-uncertain-results.md) | [live ↗](https://agentsafelabs.com/blog/we-were-wrong-about-the-uncertain-results-heres-what-actually-happened/) |
| Framework Doesn't Matter (Much). Model Does. | Jul 29, 2026 | [blog/posts/2026-07-29-framework-doesnt-matter-much-model-does.md](blog/posts/2026-07-29-framework-doesnt-matter-much-model-does.md) | [live ↗](https://agentsafelabs.com/blog/framework-doesnt-matter-much-model-does/) |

### Note on the UNCERTAIN post correction

The original "Why Claude Haiku Returned UNCERTAIN" post's central
finding — 10 of 30 prompts returning UNCERTAIN, framed as genuine model
non-determinism clustered in ASI06/09/10 — was substantially explained
by two bugs in `safelabs-eval`'s detectors (Unicode punctuation
mismatch, overly narrow refusal vocabulary), not genuine ambiguity.

Both bugs are fixed in `safelabs-eval` v0.2.1. See the correction post
above for the full writeup, and the archived UNCERTAIN post itself for
an inline correction notice rather than silent edits to the original
claim.

## Repo structure

```
blog/
  posts/          -- one file per post, filename = YYYY-MM-DD-slug.md
README.md
```

## License

Apache 2.0 — see [LICENSE](LICENSE) if present, otherwise inherits the
org-wide license used across Safe Labs AI Inc. public repos.