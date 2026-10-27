# AI Proposal Triage — Design

| | |
|---|---|
| **Name** | Sohaib Younas |
| **Reference** | MATACS-TH-2026-02 |
| **Time actually spent** | ~120 minutes |

## Summary

A design for a tool that ranks contractor proposals against admin-defined criteria.

- **AI** reads each proposal and pulls out facts, each with an exact quote as evidence.
- **Plain code** checks those facts, applies pass/fail rules, does all scoring maths and builds the ranking.
- **A person** reviews flags, can override any score with a reason, and makes the final award.

Expected result for the sample set: **A ranked 1st, E 2nd**, C and D flagged for a human, B ineligible.

## Files

| File | What it contains |
|---|---|
| [diagram/process-flow.png](diagram/process-flow.png) | Flow from upload to ranked output (AI steps in blue, rule steps in grey, human checkpoints in orange) |
| [WRITEUP.md](WRITEUP.md) | Answers to the five questions, scoring rules and assumptions |
| [PROMPTS.md](PROMPTS.md) | The extraction prompt (and optional reason prompt) exactly as sent |
| [ranking.json](ranking.json) | Expected ranking of proposals A–E |

## How I used AI tools

I used Claude to discuss the design, draft the write-up and prompts, and check the scoring arithmetic. I reviewed every number and decision myself. One place I disagreed: <!-- edit to match what actually happened --> the AI's first version had the model score "team & approach" out of 15 directly, which would make that score vary between runs and be hard to explain. I changed it so the AI only ticks a yes/no checklist and code converts it to points. I also corrected an early draft that counted all of Proposal C's experience without noticing that "since 2020" may fall outside the 5-year window, and added that as a reviewer note.
