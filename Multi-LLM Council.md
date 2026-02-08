# The Multi-LLM Council: How It Works

Most AI councils have a hidden flaw: every persona shares the same brain. Five "experts" debating, but all trained on the same data, with the same blind spots.

The fix: route each persona to a **different LLM provider**.

```
         ┌─────────────────────────────────┐
         │         OPUS (Claude)           │
         │   Crafts persona prompts        │
         │   Selects provider per persona  │
         └───────────┬─────────────────────┘
                     │
       ┌─────────────┼─────────────────┐
       ▼             ▼                 ▼
  ┌─────────┐  ┌──────────┐  ┌──────────────┐
  │ Haiku 1 │  │ Haiku 2  │  │   Haiku 3    │
  │ calls   │  │ calls    │  │   calls      │
  │ Kimi K2 │  │ GPT-5.2  │  │ Gemini Flash │
  └────┬────┘  └────┬─────┘  └──────┬───────┘
       ▼            ▼               ▼
    [east]     [pragmatic]     [futurist]
       └────────────┼───────────────┘
                    ▼
               ┌─────────┐
               │ Haiku 4 │
               │ merge   │
               └────┬────┘
                    ▼
         ┌─────────────────────┐
         │    OPUS (Claude)    │
         │    Synthesises      │
         └─────────────────────┘
```

A Kimi model trained on Chinese philosophical texts reasons differently about ethics than GPT trained primarily on English web data. Gemini, built on Google's knowledge graph, spots different patterns than DeepSeek's reasoning-specialist architecture. These aren't cosmetic differences — they're structural divergences in how each model weighs evidence, frames uncertainty, and fills gaps.

The orchestration uses **cost-tiered delegation**. Opus handles two things only: designing the persona prompts (judgment) and weaving the final synthesis (depth). Everything else — executing the API calls, waiting for responses, merging files — goes to Haiku agents at ~10x cheaper token rates. The external calls run in parallel, so five providers return in roughly the time of the slowest one.

The result is a council where disagreement is genuine, not performative. When a Kimi-powered philosopher and a GPT-powered economist converge on the same conclusion from different training distributions, that convergence carries real epistemic weight. When they diverge, the tension is worth investigating rather than dismissing.

One utility script (`llm_call.py`), five API keys, zero dependencies beyond Python's standard library. The infrastructure is deliberately minimal — because the value is in the reasoning diversity, not the plumbing.

---

*2026-02-08*
Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
