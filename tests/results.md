# Phase 6 - Test Results

**Scenarios:** `tests/scenarios.md` (T01 to T20)
**Prompt under test:** record the version in the notes column, for example "v1.1".

## How to use this log

- One row per **attempt**, not per scenario. When a scenario fails, patch the prompt, bump the version in `prompt/drafts/`, and add a new row rather than editing the old one. The history of a fix is what tells you whether the fix held.
- **Failed checks** takes the check numbers from that scenario, for example "T01 #2, #5".
- **Platform call link** is the public recording or transcript link. Open each one in a logged-out browser before relying on it.
- A scenario passes only when every one of its checks passes. A partial pass is a fail.

## Run log

| ID | Scenario | Date | Platform call link | Pass / Fail | Failed checks | Notes and fix |
|----|----------|------|--------------------|-------------|---------------|---------------|
| T01 | Happy path, in order | | | | | |
| T02 | Out-of-order answers | | | | | |
| T02b | Inference limits, 5B still asked | | | | | |
| T03 | Busy at start | | | | | |
| T04 | Wrong person | | | | | |
| T05 | Disqualify, agricultural | | | | | |
| T06 | Disqualify, no originals | | | | | |
| T07 | Disqualify, cash income | | | | | |
| T08a | Disqualify, tenure 2 years | | | | | |
| T08b | Disqualify, tenure 20 years | | | | | |
| T09 | One crore, accepts the cap | | | | | |
| T10 | One crore, declines the cap | | | | | |
| T11 | Existing loan mid-checklist | | | | | |
| T12 | "Reduce my EMI" at the offer | | | | | |
| T13 | Rate and fee diversion | | | | | |
| T14 | Changes an answer late | | | | | |
| T15 | Boundary values | | | | | |
| T16 | Rambling, scam, robot | | | | | |
| T17 | Hindi, female agent, happy path | | | | | |
| T18 | Hindi, male agent, short path | | | | | |
| T19 | Refuses the loan amount twice | | | | | |
| T20 | Transfer and disqualifier together | | | | | |
| T20b | Same, clauses reversed | | | | | |
| T21 | Declines the questions at the pitch | | | | | |
| T22 | "My company" must not infer salaried | | | | | |

## Prompt versions

| Version | Date | What changed | Scenarios re-run |
|---------|------|--------------|------------------|
| v1 | 2026-10-11 | First full prompt | not yet tested |
| v1.1 | 2026-10-11 | Slots 5a/5b, time-neutral greeting, recording line, AI disclosure, incomplete close promises nothing, gender rule in the recap | not yet tested |
| v1.2 | 2026-10-11 | Occupation inference for 5A with 5B never inferred, record-keeping claim dropped from the incomplete close, pitch-refusal branch | not yet tested |
| v1.3 | 2026-10-11 | "Company" removed from the salaried triggers; it now infers nothing | not yet tested |

## Regression set

After **any** prompt change, re-run these four before anything else:

- **T01** - the happy path still completes and hands off.
- **T15** - the boundary values still pass, so a fix has not tightened a limit by accident.
- **T02** - nothing is asked twice, so a fix has not broken the state tracker.
- **T02b** - slot five B is still asked after five A is inferred, so a fix has not over-applied the inference rule.

## Blockers

| # | Blocker | Status |
|---|---------|--------|
| 1 | `[END CALL]` must be mapped to the platform end-call function before any live call | open, Phase 5 |
| 2 | Confirm whether the platform injects the transcript or fills `{{conversation_history}}` and `{{customer_utterance}}` (P2) | open, Phase 5 |
| 3 | Confirm Hindi TTS accepts Devanagari and reads पचहत्तर लाख correctly (P7, P8) | open, Phase 5 |
