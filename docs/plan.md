# Plan: Home Credit LAP Voice Agent

## Goal
Write a system prompt for a voice agent that calls existing Home Credit customers, presents a pre-approved Loan Against Property offer (up to Rs 75,00,000), runs a 7-point eligibility check, and hands qualified leads to a senior loan expert. Implement it on a voice platform (Retell AI / Bolna), test it with real calls, and submit the prompt plus public links to call recordings and transcripts.

## Deliverables
1. `prompt/system_prompt.md` - final system prompt text
2. Public links to call recordings + call logs (audio + transcript), covering multiple scenarios
3. (Optional) `submission.md` - one page listing the prompt and a table of scenario -> link

## Project structure
```
plan.md
prompt/
  system_prompt.md      <- final prompt (v-numbered drafts in prompt/drafts/)
  drafts/
tests/
  scenarios.md          <- test matrix and customer scripts
  results.md            <- pass/fail log + fixes made
submission/
  submission.md         <- prompt + links table
```

## Phases

### Phase 0 - Setup (~30 min)
- [x] Create project folder, open in VS Code
- [ ] Create `prompt/`, `tests/`, `submission/` folders (`prompt/`, `prompt/drafts/`, `tests/` and `platform/` created; `submission/` still to do)
- [ ] Create free-trial account on Retell AI or Bolna (pick one; check it supports dynamic variables, Hindi/English voices, and public call sharing)

### Phase 1 - Requirements breakdown (~45 min)
Output: `requirements.md`
- [x] Turn the brief into a checklist: 7 data points, each with eligibility rule
- [x] List every business rule: immediate disqualification, transfer switch, amount cap, busy callback, handoff gate, out-of-order answers
- [x] List the 9 prompt variables and where each is used
- [x] Decide language handling (English only vs English + Hindi via `language_to_speak`)
- [x] Decide persona: name/gender via `agent_name`, `agent_gender`; tone = professional, advisory
- [x] Persona/tone rules, checklist state model, and an open-decisions table (G1-G30) - see `requirements.md`
- [ ] Confirm the open decisions in `requirements.md` section G before starting Phase 2

### Phase 2 - Prompt design (~1 hr)
Output: `design.md`
- [x] Outline sections: role, variables, voice-style rules, call flow, checklist state tracker, rules (disqualify / transfer / cap / busy), handoff gate, fallbacks, language rules
- [x] Define a checklist state model: each of 7 fields = unasked / answered(value) / failed
- [x] Define exact scripts for: greeting, wrong person, busy, offer pitch, each question, each disqualification, transfer, over-limit, handoff, closing
- [x] Define fallbacks: silence, "who is this", "is this fraud", "call me later", "talk to a human", off-topic, abusive, unclear answers, number formats (lakh/crore, "ten years" vs "10")
- [x] Bilingual scripts (English + Hindi, with `agent_gender` placeholders), per-turn state procedure, rule priority ladder, voice-platform checklist (P1-P13)
- [ ] Confirm the open decisions in `design.md` section 7 (O1-O12) before starting Phase 3

### Phase 3 - Write prompt v1 (~2 hr)
Output: `prompt/system_prompt.md`, copy at `prompt/drafts/system_prompt_v1.md`, review notes in `design.md` section 9
- [x] Write full `prompt/system_prompt.md`
- [x] Keep voice-friendly: one question per turn, short sentences, no lists spoken, acknowledge fillers
- [x] Self-review against the checklist from Phase 1 (all 16 business rules traced; 4 gaps found and fixed)
- [x] Save `prompt/drafts/system_prompt_v1.md` as an exact copy
- [x] Prompt v1.1: slots 5a/5b, time-neutral greeting, recording disclosure, AI disclosure, no-promise incomplete close, gender rule in the recap (`design.md` section 10)
- [x] Prompt v1.2: occupation inference for slot 5A, slot 5B never inferred, incomplete close trimmed, pitch-refusal branch (`design.md` section 10)
- [x] Prompt v1.3: "company" removed from the salaried inference triggers (`design.md` section 10, test T22)
- [ ] Map `[END CALL]` to the platform end-call function (Phase 5, blocks any live call)

### Phase 4 - Test design (~45 min)
Output: `tests/scenarios.md` (T01-T20, caller cheat sheet, submission shortlist), `tests/results.md`
- [x] Write the 20 scenarios with turn-by-turn customer scripts, expected behaviour, end state and pass-fail checks
- [x] Caller cheat sheet: how to speak naturally, what to avoid
- [x] Variable presets per scenario (EN-F, EN-M, HI-F, HI-M)
- [x] Mark the 8 submission-recording candidates: T01, T02, T03, T05, T09, T11, T17, T15
- [x] Create `tests/results.md` from the template, with a regression set and a blockers list
- [x] Add T02b (inference limits) and T21 (declines the questions at the pitch) for the v1.2 branches; T02 checks made strict

Scenarios (in `tests/scenarios.md`):
1. Happy path, all 7 answered in order -> handoff
2. Out-of-order answers (customer volunteers loan amount + income early)
3. Customer is busy -> callback time
4. Wrong person / customer unavailable
5. Disqualify: agricultural property
6. Disqualify: no original documents
7. Disqualify: cash income
8. Disqualify: tenure < 3 or > 15 years
9. Loan > 75 lakh -> explain cap, accept max
10. Loan > 75 lakh -> customer declines
11. Existing loan / lower EMI mentioned -> transfer path
12. Skipped question after a diversion -> agent returns to it before handoff
13. Joint ownership; salaried-with-bank-income pass
14. Rambling, fillers, interruptions, "is this a scam?", "what is the interest rate?"
15. Hindi call (if bilingual)

### Phase 5 - Platform setup (~45 min)
Output: `platform/setup_notes.md` - checklist prepared, blanks to fill in on the platform
- [ ] Create agent, paste prompt v1, choose a professional voice
- [ ] Map variables with test values (customer_name, agent_name, agent_gender, language_to_speak, etc.)
- [ ] Confirm how `conversation_history` / `customer_utterance` are supplied (platform may inject automatically)
- [ ] Do a first web-call smoke test

### Phase 6 - Test and iterate (~2-3 hr)
- [ ] Run each scenario, log result in `tests/results.md`
- [ ] For each failure: find the cause in the transcript, patch the prompt, re-run
- [ ] Save versions in `prompt/drafts/` (v1, v2, ...) and copy the final to `prompt/system_prompt.md`
- [ ] Repeat until all scenarios pass cleanly

### Phase 7 - Final recordings and submission (~1 hr)
- [ ] Make clean final calls for each key scenario
- [ ] Generate public links for recording + transcript; open each in a logged-out/incognito browser to confirm it works
- [ ] Write `submission/submission.md`: final prompt + table (scenario | link | outcome)
- [ ] Final check against the brief: 7 points, all rules, all variables, handoff gate, natural conversation

## Time estimate
About 8-10 hours total, realistically 1-2 days. Phase 6 is the largest and least predictable.

## Risks
- Free-trial credit limits: write test scripts first and do not waste calls
- Agent asks a question twice or skips one: needs an explicit state tracker in the prompt
- Agent ends the call too early or never ends it: needs explicit end-call instructions
- Public link access: verify logged-out