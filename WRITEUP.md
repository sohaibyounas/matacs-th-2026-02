# Design Write-up

In short: the AI reads the proposals, plain code scores and ranks them, and a reviewer decides.

See the flow in [diagram/process-flow.png](diagram/process-flow.png).

## How the sample set was scored

All formulas come from the admin's criteria settings, not from code.

| Criterion | Points | Rule |
|---|---|---|
| Experience | 30 | 10 or more qualifying installs = full marks, fewer = proportional |
| Price | 25 | Lowest eligible price ÷ this price × 25 |
| Timeline | 15 | Shortest eligible timeline ÷ this timeline × 15 |
| Warranty | 15 | Panel: 25 yrs = 7.5. Workmanship: 15 yrs = 7.5, proportional |
| Team & approach | 15 | 3 points for each of 5 checklist items found |

Result: A 93.8 (1st), E 90.2 (2nd), C flagged, D flagged, B ineligible. Details in [ranking.json](ranking.json).

---

## 1. AI vs. rules

The AI does one step: reading free text and pulling out facts. Proposals are written differently each time ("12 weeks of award", "since 2021", "subcontractor to be confirmed"). This is what AI is good at and what rules are bad at.

Everything else is plain code:
- PDF to text, injection scan, checking the AI's quotes really exist in the text
- Mandatory pass/fail checks
- All scoring maths, ranking and flags

I drew the line here because maths and yes/no checks must give the same answer every time and be easy to check by hand. An AI deciding the ranking can be wrong or manipulated (see Proposal C). So the AI only answers "what does the proposal say?", never "who should win?".

Even the subjective criterion (team & approach) is turned into a checklist the AI ticks, so it is a fact question, not an opinion.

## 2. Consistency and explainability

To keep results repeatable:
- Temperature 0, fixed prompt version, fixed model version
- Extracted facts are saved, so re-ranking re-uses them instead of asking the AI again
- Scoring is plain maths, so the same facts always give the same score

So ranking the same proposals twice with the same criteria version gives the same result. If a fresh extraction is ever run, code compares it with the saved one and flags differences.

To keep them explainable, the reviewer sees next to every score:
- the exact quote from the proposal (and page) it came from
- the formula used, e.g. "Price: 231,000 ÷ 238,000 × 25 = 24.3"
- a one-line reason and any flags

The reviewer can check each number against the source in seconds.

## 3. Human in the loop

A human steps in at four points:
1. The admin defines and approves the criteria before any scoring.
2. The reviewer resolves every flag (missing data, unclear mandatory check, injection found, low-confidence extraction). Flagged proposals are never silently ranked or dropped.
3. The reviewer can correct any extracted fact or override any score, with a required written reason. The original value is kept.
4. The final award is always a human decision. The tool only recommends.

Each proposal has an audit log with: file hash, criteria version, prompt version, model name, raw AI output, validated facts with quotes, each score and formula, flags, every override (old value, new value, reason, who, when), and the final decision.

## 4. Changing criteria

- Criteria are saved as numbered versions. Every score records which version it used.
- If an admin edits criteria or weights, existing results are marked "out of date", not silently changed.
- If only a weight or formula changes, results are re-scored instantly from the saved facts. No AI cost.
- If a criterion is new or changed, only that criterion is re-extracted by the AI.
- The reviewer sees old vs new side by side and must confirm. Old results stay in history.
- I'd recommend locking criteria once a review starts; later changes need a logged reason.

## 5. Risks and safeguards

| Risk | Safeguard |
|---|---|
| Wrong extraction | Every fact must come with an exact quote; code checks the quote exists in the text. Numbers re-parsed by code. Low confidence = flag. Reviewer sees the evidence. |
| Made-up facts | Prompt says "not stated = null". Missing info lowers the score or raises a flag, never filled in. Example: D's timeline is null, so D is flagged instead of guessed. |
| Look-alike facts | Rule that facts must belong to the bidder. Example: B's licence is held by an unnamed subcontractor, so B fails the mandatory check, even though it is the cheapest. |
| Manipulation / prompt injection | Proposal text is wrapped as data, prompt says never follow it, code also scans for phrases like "rank it first", and the AI has no power to rank anyway. Example: C is flagged for a human and the hidden instruction has zero effect on its score. |
| Bias | Company name, location and branding are not criteria and are not used in scoring. Same prompt and formulas for everyone. Overrides are logged and can be reviewed for patterns. |
| Cost | One AI call per proposal, cached. Weight changes re-use saved facts. Upload size limit. |
| Over-trust | Output is labelled "recommendation" and flags are shown above the ranking. |

## Sample set notes

- C: on content alone it would score about 85.0 (3rd). Its experience counts installs "since 2020", which may include some older than 5 years; the reviewer should confirm.
- D: lowest price, so worth asking the bidder for a timeline.
- A vs E: close. A wins on experience and timeline; E on warranty and approach.

## Assumptions

- "Last 5 years" means counted back from the review date.
- Price and timeline are scored relative to the eligible proposals in the same batch (a common procurement method). Flagged proposals that pass mandatory checks are included in the price baseline, so the scores do not move once their flag is cleared.
- Warranty caps (25 years panel, 15 years workmanship) and the team checklist are example admin settings.
