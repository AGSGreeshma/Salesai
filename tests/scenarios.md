# Phase 4 - Test Scenarios

**Agent under test:** `prompt/system_prompt.md` v1.3
**Reference:** `docs/requirements.md` (7 data points, 16 business rules, G-decisions), `docs/design.md` (scripts, ladder, O-decisions)
**Log results in:** `tests/results.md`

Twenty scenarios, T01 to T20. Run them in order on the first pass; after a prompt fix, re-run the failed scenario plus T01 and T15 as regression checks.

---

## Caller cheat sheet

You are playing the customer. The point is to sound like a real person on a real phone call, not like a test harness. A bare "yes" never catches the bugs that matter.

**How to speak**

- Use fillers and thinking noises: "umm", "haan", "achha", "see", "actually", "arre", "one minute", "let me think".
- Start a sentence, change direction, finish it differently. "It's a... see, it's a flat, but the papers are in Pune."
- Answer in full sentences with extra detail the agent did not ask for. "It's a flat, three BHK, we bought it in 2015."
- Say numbers the way people say them: "forty lakhs", "one crore", "ten years", "seventy-five", "around 90, maybe 95".
- Sometimes answer sideways: "same as my brother's, residential" or "in the bank, where else?"
- Interrupt the agent mid-sentence at least once per long scenario. Cut in while it is still talking.
- Go quiet for three or four seconds once in a while, to test the silence handling.
- Be a bit impatient in one or two scenarios. Ask "how long will this take?"
- In Hindi scenarios, speak natural Hindi with the usual English words mixed in: "property", "loan", "bank", "documents", "tenure".

**What to avoid**

- Do not answer only "yes" or "no" unless the scenario script says so.
- Do not volunteer an answer the scenario has not reached, unless it is an out-of-order scenario.
- Do not read the script in a flat voice. Vary pace and volume.
- Do not help the agent recover. If it asks something it already asked, say so the way a real person would: "I just told you that."
- Do not correct the agent's facts for it. If it misstates something, let it stand and note it as a failure.
- Do not stay on the line after the agent closes. Let the call end so you can check whether it actually hangs up.

**Before every call, confirm the variables are set.** A call with an unfilled variable is a wasted call. Listen to the first sentence: if you hear a placeholder name or a bracket read aloud, hang up and fix the mapping.

---

## Variable presets

| Preset | `agent_name` | `agent_gender` | `language_to_speak` | `customer_name` | Used by |
|---|---|---|---|---|---|
| EN-F | Priya | female | English | varies | T01, T02, T03, T05, T07, T09, T11, T13, T15, T16, T19 |
| EN-M | Rohit | male | English | varies | T04, T06, T08, T10, T12, T14, T20 |
| HI-F | Priya | female | Hindi | Sunita Devi | T17 |
| HI-M | Rohit | male | Hindi | Mahesh Kumar | T18 |

`company_name` = Home Credit for every scenario. `current_date`, `current_day`, `current_time` are set to the real time of the call. `additional_context_from_rag` is left empty except where a scenario says otherwise.

---

## Scenario index

| ID | Name | Language | Expected end state | Submission candidate |
|---|---|---|---|---|
| T01 | Happy path, in order | EN | qualified | **yes** |
| T02 | Out-of-order answers | EN | qualified | **yes** |
| T02b | Inference limits, 5A inferred, 5B still asked | EN | disqualified | no |
| T03 | Busy at start | EN | callback | **yes** |
| T04 | Wrong person | EN | callback | no |
| T05 | Disqualify, agricultural | EN | disqualified | **yes** |
| T06 | Disqualify, no originals | EN | disqualified | no |
| T07 | Disqualify, cash income | EN | disqualified | no |
| T08 | Disqualify, tenure 2 years (+20-year variant) | EN | disqualified | no |
| T09 | One crore, accepts the cap | EN | qualified | **yes** |
| T10 | One crore, declines the cap | EN | declined cap | no |
| T11 | Existing loan mid-checklist | EN | transfer | **yes** |
| T12 | "Reduce my EMI" at the offer | EN | transfer | no |
| T13 | Rate and fee diversion | EN | qualified | no |
| T14 | Changes an answer late | EN | disqualified | no |
| T15 | Boundary values | EN | qualified | **yes** |
| T16 | Rambling, scam, robot | EN | qualified | no |
| T17 | Hindi, female agent, happy path | HI | qualified | **yes** |
| T18 | Hindi, male agent, short path | HI | disqualified | no |
| T19 | Refuses the loan amount twice | EN | incomplete | no |
| T20 | Transfer and disqualifier together | EN | transfer | no |
| T21 | Declines the questions at the pitch | EN | not interested | no |
| T22 | "My company" must not infer salaried | EN | qualified | no |

---

## T01 - Happy path, answers in order

**Goal:** the clean reference call. All eight slots filled in order, ending in handoff.
**Language / preset:** EN-F. **Customer:** Rakesh Sharma.
**Persona:** 42, salaried IT employee in Pune, mildly curious, cooperative, slightly slow to answer.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Haan, Rakesh speaking. Who is this?" | Confirm identity, thank, give the recording line once, then the offer pitch ending in one question |
| C2 | "Achha, okay... haan, go ahead, ask." | Ask question one, the property type |
| C3 | "It's a flat. Residential, three BHK, in Pune." | Acknowledge, ask question two |
| C4 | "See, it's in my name only. Just mine." | Acknowledge, ask question three |
| C5 | "Haan haan, originals are with me, in the locker." | Acknowledge, ask question four |
| C6 | "Umm, I need around forty lakhs." | Acknowledge, ask slot five A |
| C7 | "I'm salaried. Working in an IT company, eight years now." | Acknowledge, ask slot five B **in a separate turn** |
| C8 | "Salary comes in the bank, HDFC account." | Acknowledge, ask question six |
| C9 | "Market value... umm, maybe ninety lakhs? Could be ninety-five." | Accept the estimate, ask question seven |
| C10 | "Ten years should be fine for me." | Handoff close, then end the call |

**Expected end state:** qualified.
**Pass checks:**
1. Exactly one question per agent turn, all the way through.
2. Slots five A and five B arrive in two separate turns.
3. The recording line is said once, after the identity is confirmed, before the pitch.
4. No interest rate, EMI or fee is mentioned at any point.
5. The handoff close says a senior loan expert will call back shortly with the exact interest rate.
6. The call actually ends. No trailing question, no silence waiting for a reply.
7. Acknowledgements vary; the same one is not used twice in a row.

---

## T02 - Out-of-order answers

**Goal:** the agent must absorb volunteered answers and never re-ask them.
**Language / preset:** EN-F. **Customer:** Rakesh Sharma.
**Persona:** businesslike, wants to get to the point, has done this before.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yes, Rakesh here." | Identity, recording line, pitch |
| C2 | "Actually see, I know about this already. I need about fifty lakhs, and my salary comes into the bank, ICICI. Just tell me the process." | Acknowledge both facts in one short line, then ask question one. **Must not** ask the loan amount or the income mode again |
| C3 | "Residential. It's a house, ground floor plus one." | Ask question two |
| C4 | "Joint, with my wife." | Ask question three |
| C5 | "Haan, originals are there with me." | Skip questions four, five A and five B. Ask **question six**, the market value |
| C6 | "Umm, about one crore twenty lakhs." | Ask question seven |
| C7 | "Fifteen years." | Handoff close, then end the call |

**Expected end state:** qualified.
**Pass checks:**
1. The loan amount question is never asked. Fifty lakhs is carried from C2.
2. The income mode question is never asked. Bank is carried from "comes into the bank" in C2.
3. The occupation question is never asked. The word "salary" in C2 fills slot five A as salaried.
4. The question asked after C5 is the market value. Asking about the amount, the occupation or the income mode at that point is a fail.
5. The agent acknowledges what it picked up from C2, so the customer hears that it registered.
6. All eight slots are filled before the handoff.
7. No re-pitch after C2, even though the customer cut the pitch short.

---

## T02b - Inference limits: five A inferred, five B still asked

**Goal:** slot five B must never be inferred from slot five A. Being salaried says nothing about how the money arrives.
**Language / preset:** EN-F. **Customer:** Rakesh Sharma.
**Persona:** salaried, but paid in cash by a small employer. Mentions his job early, without being asked.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yes, Rakesh." | Identity, recording line, pitch |
| C2 | "Haan okay. See, I'm a salaried employee, my company is in Hinjewadi." | Fill slot five A as salaried from his own words, then ask question one. **Must not** ask the occupation |
| C3 | "Residential flat." | Question two |
| C4 | "Mine only." | Question three |
| C5 | "Yes, originals with me." | Question four |
| C6 | "Thirty lakhs." | Ask **slot five B**, the income mode. Five A is already filled, so this is the next unanswered slot |
| C7 | "Actually my company pays me in cash. Every month, cash in hand." | Disqualification close, then end the call |

**Expected end state:** disqualified.
**Pass checks:**
1. The occupation question is never asked. It was filled from "salaried employee" at C2.
2. Slot five B **is** asked at C6. Being salaried is not treated as proof of bank income.
3. The cash answer fails, even though the customer is salaried. Occupation and income mode are judged separately.
4. The close names no reason.
5. The agent does not jump from question four to the market value, which would mean it had wrongly inferred five B.

---

## T03 - Busy at the start, gives a callback time

**Goal:** the callback path, including relative-date resolution.
**Language / preset:** EN-F. **Customer:** Rakesh Sharma. Note the real `current_date` and `current_day` before the call.
**Persona:** driving, distracted, polite but short.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yeah, speaking. But see, I'm driving right now. Can you call later?" | Not pitch. Apologise briefly and ask which day and time suits |
| C2 | "Umm... tomorrow evening? After six sometime." | Resolve "tomorrow" against `current_date`, propose or confirm one specific time, read back day, weekday and time once, then end the call |

**Expected end state:** callback.
**Pass checks:**
1. The offer, the amount and the eligibility questions are never mentioned.
2. The confirmation names the actual weekday, not just "tomorrow".
3. One specific time is confirmed, not a vague "evening".
4. The agent does not ask for an alternative phone number.
5. The call ends after the confirmation. It does not try to sneak in a question.

---

## T04 - Wrong person answers, customer unavailable

**Goal:** no information leaks to a third party.
**Language / preset:** EN-M. **Customer:** the call is for Rakesh Sharma; his brother answers.
**Persona:** brother, helpful, curious about who is calling.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Hello? No, this is his brother. Rakesh isn't at home." | Apologise for the trouble, ask for a good time to reach Rakesh. **No loan, no amount, no offer** |
| C2 | "Why, what is it regarding? Some loan thing?" | Decline to share details with a third party, politely, and repeat the question about a good time |
| C3 | "Achha, he'll be back by evening. Call after seven." | Confirm, thank, end the call |

**Expected end state:** callback.
**Pass checks:**
1. The words loan, property loan, offer, lakh and pre-approved are never spoken to the brother.
2. No eligibility question is asked.
3. The agent does not confirm or deny what the call is about.
4. The callback time is acknowledged and the call ends.

---

## T05 - Disqualify, agricultural property

**Goal:** immediate disqualification with no reason named, and no reopening under pressure.
**Language / preset:** EN-F. **Customer:** Rakesh Sharma.
**Persona:** owns farmland near Nashik, keen on the loan, pushes back when refused.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yes, Rakesh." | Identity, recording line, pitch |
| C2 | "Haan, tell me." | Ask question one |
| C3 | "See, it's agricultural land. About two acres, near Nashik." | Disqualification close, then end the call |
| C4 | *(if the line is still open)* "Arre, why? What's the problem? Agricultural is also property, no?" | One line declining to go into details, then end the call |

**Expected end state:** disqualified.
**Pass checks:**
1. The close does not say agricultural, land type, property type, or name any criterion.
2. The wording is "do not meet the criteria for this offer at this time" or very close to it.
3. No further eligibility questions are asked.
4. No workaround, no alternative product, no "let me check with my senior", no callback promise.
5. The call ends. If the customer presses, the agent repeats one short line and ends again.

---

## T06 - Disqualify, no original documents

**Goal:** photocopies and lost papers are a failure, not a transfer.
**Language / preset:** EN-M. **Customer:** Suresh Patel.
**Persona:** vague about paperwork, embarrassed.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yes, Suresh here." | Identity, recording line, pitch |
| C2 | "Okay, ask." | Question one |
| C3 | "Commercial. A shop, in the market area." | Question two |
| C4 | "Mine only." | Question three |
| C5 | "Originals? Umm... actually no. See, they got lost in the flood. I only have photocopies." | Disqualification close, then end the call |

**Expected end state:** disqualified.
**Pass checks:**
1. Photocopies are treated as a failure, not as a yes.
2. The close names no reason.
3. The agent does not route this to the transfer team, because no other lender was mentioned.
4. No remaining questions are asked.

---

## T07 - Disqualify, cash income, via the mixed-income probe

**Goal:** the hedged-income answer must trigger the main-income probe once, and a hedged reply must fail.
**Language / preset:** EN-F. **Customer:** Imran Qureshi.
**Persona:** shop owner, cash business, chatty.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Haan, Imran speaking." | Identity, recording line, pitch |
| C2 | "Achha, go on." | Question one |
| C3 | "Residential, a house. Two floors." | Question two |
| C4 | "Joint with my brother." | Question three |
| C5 | "Yes, originals are with me." | Question four |
| C6 | "Thirty lakhs, around that." | Slot five A |
| C7 | "I have my own business. A cloth shop." | Slot five B, in a separate turn |
| C8 | "See, some comes in the bank, but mostly it's cash. You know how shops are." | Ask the main-income probe once: does the main income come into the bank |
| C9 | "Mostly cash, haan. Bank mein thoda hi aata hai." | Disqualification close, then end the call |

**Expected end state:** disqualified.
**Pass checks:**
1. The mixed answer at C8 does not immediately disqualify. The probe is asked once.
2. The hedged reply at C9 fails. It is not treated as a pass.
3. Self-employed by itself never fails. Only the income mode does.
4. The close names no reason.
5. The probe is asked once, not twice.

---

## T08 - Disqualify, tenure outside the range

**Goal:** tenure gets no second chance, unlike the loan amount.
**Language / preset:** EN-M. **Customer:** Suresh Patel.
**Persona:** confident, wants to repay fast.

Run both variants as separate calls.

**T08a, two years.** Reach question seven through clean answers: residential, sole, originals yes, twenty lakhs, salaried, bank, market value sixty lakhs.

| # | Customer says | Agent must |
|---|---|---|
| C8 | "Just two years. I'll repay it fast, I don't want a long loan." | Disqualification close, then end the call |

**T08b, twenty years.** Same path.

| # | Customer says | Agent must |
|---|---|---|
| C8 | "Twenty years. Make it as long as possible, I want a small EMI." | Disqualification close, then end the call |

**Expected end state:** disqualified, both variants.
**Pass checks:**
1. The agent does **not** say the range is three to fifteen years and ask whether they want to proceed within it. That pattern belongs to the loan amount only.
2. The close names no reason, so the customer is not told the tenure was the problem.
3. No EMI figure is mentioned in response to C8 in T08b.
4. In T08b, the phrase "as long as possible" is not silently read as fifteen years. Twenty years was stated and it fails.

---

## T09 - One crore, cap explained, customer accepts

**Goal:** the amount cap, crore normalisation, and the checklist continuing afterwards.
**Language / preset:** EN-F. **Customer:** Rakesh Sharma.
**Persona:** ambitious, negotiates a little, then accepts.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yes, Rakesh." | Identity, recording line, pitch |
| C2 | "Haan, ask." | Question one |
| C3 | "Residential, a bungalow." | Question two |
| C4 | "My name only." | Question three |
| C5 | "Haan, originals are there." | Question four |
| C6 | "I need one crore." | Explain the maximum is seventy-five lakh, ask whether to go ahead with seventy-five lakh |
| C7 | "Can't you do more? It's a big property." | Repeat the maximum in one short sentence, without negotiating |
| C8 | "Achha, theek hai. Seventy-five is fine." | Acknowledge, then continue with slot five A |
| C9 | "Salaried." | Slot five B |
| C10 | "Bank, yes." | Question six |
| C11 | "Around one crore eighty." | Question seven |
| C12 | "Twelve years." | Handoff close, then end the call |

**Expected end state:** qualified.
**Pass checks:**
1. One crore is recognised as above the limit. It is not misheard as one lakh or accepted as valid.
2. The cap is explained once at C6. At C7 the maximum is restated briefly, with no new offer and no negotiation.
3. The checklist resumes at the next unanswered slot, not from the beginning.
4. Asking for more than the limit does not disqualify anyone.
5. The handoff happens, with all eight slots filled.

---

## T10 - One crore, customer declines the cap

**Goal:** declining the maximum is a polite close, not a rejection.
**Language / preset:** EN-M. **Customer:** Suresh Patel.
**Persona:** firm, not interested in less.

| # | Customer says | Agent must |
|---|---|---|
| C1 to C5 | Same as T09 through question three | As T09 |
| C6 | "I need one crore, minimum." | Explain the maximum, ask whether to proceed with seventy-five lakh |
| C7 | "No no. Seventy-five is too less for me. Then leave it." | Polite close: that is fine, the offer stays open, thank you. Then end the call |

**Expected end state:** declined cap.
**Pass checks:**
1. The agent does **not** say they fail to meet the criteria. This is not a disqualification.
2. No handoff and no mention of the senior expert.
3. The offer is left open in the wording.
4. No second attempt to sell seventy-five lakh.
5. The call ends.

---

## T11 - Existing loan mentioned mid-checklist

**Goal:** the transfer path, triggered part-way through.
**Language / preset:** EN-F. **Customer:** Rakesh Sharma.
**Persona:** already has a mortgage, mentions it casually.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yes, Rakesh." | Identity, recording line, pitch |
| C2 | "Haan, go ahead." | Question one |
| C3 | "Residential flat." | Question two |
| C4 | "Mine only." | Question three |
| C5 | "Actually, the originals are with SBI. I already have a loan on this flat." | Transfer close, then end the call |

**Expected end state:** transfer.
**Pass checks:**
1. Papers held by another lender is treated as a transfer trigger, not as a documents failure.
2. The close says a loan transfer specialist will contact them shortly.
3. The agent asks nothing further: not the lender, not the outstanding amount, not the EMI.
4. The remaining checklist questions are abandoned.
5. No rate, no eligibility promise, no handoff claim.

---

## T12 - "I want to reduce my EMI", at the offer stage

**Goal:** the transfer trigger before the checklist even starts.
**Language / preset:** EN-M. **Customer:** Suresh Patel.
**Persona:** stressed about his current EMI.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yes, Suresh." | Identity, recording line, pitch |
| C2 | "Actually what I really want is to reduce my current EMI. It's too much, fourteen thousand a month. Can you do something?" | Transfer close, then end the call |

**Expected end state:** transfer.
**Pass checks:**
1. The transfer trigger is caught at the offer stage. No checklist question is asked.
2. The agent does not quote or discuss his current EMI, or compare it with anything.
3. The close is the transfer close, not the not-interested close.
4. The call ends.

---

## T13 - Rate and fee diversion mid-flow

**Goal:** answer within the rules, then return to the exact pending question.
**Language / preset:** EN-F. **Customer:** Rakesh Sharma.
**Persona:** price-sensitive, asks twice, then cooperates.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yes, speaking." | Identity, recording line, pitch |
| C2 | "Before anything, what's the interest rate?" | Defer to the senior expert in one line, no number, then ask question one |
| C3 | "Residential, a flat." | Question two |
| C4 | "Joint, with my father." | Question three |
| C5 | "Haan, originals are with me. And what about processing fee? Is there any hidden charge?" | Defer the fee in one line, no number, then return to question four |
| C6 | "Forty-five lakhs." | Slot five A |
| C7 | "Salaried. But tell me at least an approximate rate, around what percent?" | Refuse the number again, politely, once. Then ask slot five B |
| C8 | "Bank." | Question six |
| C9 | "Eighty lakhs or so." | Question seven |
| C10 | "Ten years." | Handoff close, then end the call |

**Expected end state:** qualified.
**Pass checks:**
1. No percentage, no rupee figure, no range, and no "around" is given for rate or fee, across all three attempts.
2. After each diversion the agent returns to the question that was pending, not to an earlier one and not skipping ahead.
3. Question three is answered at C5 and never re-asked, even though the turn also contained a question.
4. All eight slots are filled before the handoff.
5. The repeated rate question does not derail the flow or cause a repeat of the pitch.

---

## T14 - Customer changes an earlier answer

**Goal:** a late correction must re-run the rules and disqualify.
**Language / preset:** EN-M. **Customer:** Suresh Patel.
**Persona:** realises half-way that he described the wrong property.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yes, Suresh." | Identity, recording line, pitch |
| C2 | "Okay." | Question one |
| C3 | "Residential." | Question two |
| C4 | "Mine." | Question three |
| C5 | "Yes, with me." | Question four |
| C6 | "Fifty lakhs." | Slot five A |
| C7 | "Salaried." | Slot five B |
| C8 | "Bank account." | Question six |
| C9 | "Actually, one minute. I should tell you, the property I want to give is the farm land, agricultural. Not the house. Is that a problem?" | Re-apply the rule to the new value and run the disqualification close, then end the call |

**Expected end state:** disqualified.
**Pass checks:**
1. The correction overrides the earlier "residential" answer, even though it came six turns later.
2. The agent does not keep going on the basis of the old answer and hand off.
3. The close names no reason, so it does not say the land type was the problem.
4. The agent does not argue, re-ask, or suggest using the house instead.
5. No handoff and no expert callback promise.

---

## T15 - Boundary values

**Goal:** the inclusive limits pass cleanly. Exactly seventy-five lakh and exactly fifteen years.
**Language / preset:** EN-F. **Customer:** Meena Iyer.
**Persona:** precise, gives exact figures, joint owner with her husband.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yes, this is Meena." | Identity, recording line, pitch |
| C2 | "Haan, please ask." | Question one |
| C3 | "It's residential. A flat in Chennai." | Question two |
| C4 | "Joint. Me and my husband, both names." | Question three |
| C5 | "Yes, the originals are with us." | Question four |
| C6 | "Exactly seventy-five lakhs is what we need." | Acknowledge and go to slot five A. **No cap conversation** |
| C7 | "I'm salaried, government service." | Slot five B |
| C8 | "Salary comes to my SBI account." | Question six |
| C9 | "The flat would be around one crore ten." | Question seven |
| C10 | "Fifteen years. The maximum." | Handoff close, then end the call |

**Expected end state:** qualified.
**Pass checks:**
1. Exactly seventy-five lakh passes with no mention of a maximum or a cap.
2. Exactly fifteen years passes. It is not treated as above the range.
3. Joint ownership passes with no request for the husband's consent or details.
4. The handoff happens.
5. Use this as a regression test after every prompt change.

---

## T16 - Rambling customer, fillers, interruption, scam and robot questions

**Goal:** conversational robustness. The hardest scenario for a voice agent.
**Language / preset:** EN-F. **Customer:** Rakesh Sharma.
**Persona:** talkative, suspicious at first, warms up. Interrupt the agent at least twice, and go silent once for four seconds.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Hello? Haan who is this, which company?" | Identify in one line, confirm identity, recording line, pitch |
| C2 | *(interrupt mid-pitch)* "Wait wait. Is this a scam? Because I keep getting these calls asking for OTP." | Reassure once, state it will not ask for bank details, OTP or password, then continue to the pending question |
| C3 | "Achha... and are you a robot? You sound like a recording." | Say plainly it is an AI voice assistant from Home Credit handling the first steps, with a senior expert taking over. Then ask question one |
| C4 | "See, I have a flat in Kothrud, we bought it in 2015, my wife wanted a bigger place but anyway, it's residential, three BHK." | Extract residential, acknowledge, ask question two |
| C5 | *(four seconds of silence)* | Re-prompt once, briefly |
| C6 | "Haan sorry, I'm here. What was the question?" | Repeat question two in shorter words, no new information |
| C7 | "Joint. With my wife." | Question three |
| C8 | "Haan the papers are there. How long will this take? I have a meeting." | Answer the time question in a few words, then ask question four |
| C9 | "Umm, around sixty lakhs. Maybe fifty-five, sixty." | Take sixty or confirm the figure once, then slot five A |
| C10 | "Salaried." | Slot five B |
| C11 | "Bank." | Question six |
| C12 | "One crore forty, around." | Question seven |
| C13 | "Twelve years." | Handoff close, then end the call |

**Expected end state:** qualified.
**Pass checks:**
1. Never more than one question per turn, even in the busy turns C2, C3 and C8.
2. The scam reassurance is given once and not repeated.
3. The AI question is answered honestly. The agent never claims to be human and never dodges.
4. No request for OTP, account number, PAN or Aadhaar at any point.
5. After the silence, the agent re-prompts once rather than repeating the whole pitch.
6. The rambling answer at C4 is parsed correctly and the property type is not asked again.
7. The interruptions do not cause a lost question or a duplicate question.

---

## T17 - Hindi call, female agent, full happy path

**Goal:** the complete Hindi flow with correct female self-reference.
**Language / preset:** HI-F. **Customer:** Sunita Devi.
**Persona:** 48, self-employed, tailoring business, bank account for business income, speaks Hindi with the usual English loanwords.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "हाँ जी, सुनीता बोल रही हूँ। कौन?" | पहचान, रिकॉर्डिंग लाइन, ऑफ़र पिच, सब हिंदी में |
| C2 | "अच्छा... हाँ जी, पूछिए।" | सवाल एक, प्रॉपर्टी का प्रकार |
| C3 | "देखिए, घर है। रेसिडेंशियल, दो मंज़िल का।" | सवाल दो |
| C4 | "मेरे और मेरे पति, दोनों के नाम पर है।" | सवाल तीन |
| C5 | "हाँ जी, ओरिजिनल कागज़ हमारे पास ही हैं।" | सवाल चार |
| C6 | "उम्म, पैंतीस लाख चाहिए।" | स्लॉट पाँच ए |
| C7 | "अपना काम है, सिलाई का। खुद का बिज़नेस।" | स्लॉट पाँच बी, अलग टर्न में |
| C8 | "बैंक में ही आता है, करंट अकाउंट में।" | सवाल छह |
| C9 | "अंदाज़ा... अस्सी लाख के आसपास होगी।" | सवाल सात |
| C10 | "दस साल।" | हैंडऑफ़ क्लोज़, फिर कॉल ख़त्म |

**Expected end state:** qualified.
**Pass checks:**
1. Every agent turn is in Hindi. No full English sentences. English finance loanwords are fine.
2. Female self-reference throughout: बोल रही हूँ, समझ गयी, करूँगी. No male form anywhere.
3. The braces are never spoken. No "रहा स्लैश री" and no reading of both forms.
4. Self-employed with bank income passes, and the probe is not needed because C8 is clear.
5. Amounts are spoken as पैंतीस लाख and पचहत्तर लाख, never as digits.
6. The handoff close mentions the senior expert and the exact interest rate, in Hindi.

---

## T18 - Hindi call, male agent, short path

**Goal:** male Hindi forms, on the shortest possible path.
**Language / preset:** HI-M. **Customer:** Mahesh Kumar.
**Persona:** 55, owns farmland, direct.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "हाँ, महेश बोल रहा हूँ।" | पहचान, रिकॉर्डिंग लाइन, ऑफ़र पिच |
| C2 | "हाँ बताइए।" | सवाल एक |
| C3 | "खेती की ज़मीन है। चार एकड़।" | अयोग्यता वाला क्लोज़, कारण बताए बिना, फिर कॉल ख़त्म |
| C4 | *(if still connected)* "क्यों? क्या दिक्कत है?" | एक छोटी लाइन, जानकारी नहीं दे सकता, फिर कॉल ख़त्म |

**Expected end state:** disqualified.
**Pass checks:**
1. Male self-reference throughout: बोल रहा हूँ, समझ गया, सकता हूँ. No female form anywhere.
2. The disqualification close is in Hindi and names no reason.
3. खेती की ज़मीन is recognised as agricultural.
4. The agent does not switch to English at any point.
5. The call ends, and ends again if the customer presses.

---

## T19 - Customer refuses the loan amount twice

**Goal:** the incomplete-information close, with nothing promised.
**Language / preset:** EN-F. **Customer:** Rakesh Sharma.
**Persona:** wants the agent to name a figure first, will not commit.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yes, Rakesh." | Identity, recording line, pitch |
| C2 | "Haan, ask." | Question one |
| C3 | "Residential flat." | Question two |
| C4 | "Mine only." | Question three |
| C5 | "Yes, with me." | Question four |
| C6 | "See, that depends on what you people can offer. You tell me." | First rephrasing of question four. May say "up to seventy-five lakh", nothing more specific |
| C7 | "Arre, how much can I get? You tell me the figure first." | Second rephrasing. Still no specific figure offered |
| C8 | "I'll decide later. I can't say right now." | Incomplete-information close, then end the call |

**Expected end state:** incomplete.
**Pass checks:**
1. The close does **not** promise a callback from the loan expert or anyone else.
2. The close does **not** say they do not meet the criteria. This is not a disqualification.
3. The close does **not** say they qualify or mention a handoff.
4. The wording notes what was covered and invites the customer to reach out when they have the details.
5. The agent never names an eligible figure beyond "up to seventy-five lakh".
6. Exactly two rephrasings are used, not three or four.

---

## T20 - Transfer trigger and a disqualifier in the same sentence

**Goal:** the G5 collision. Transfer must win.
**Language / preset:** EN-M. **Customer:** Suresh Patel.
**Persona:** gives both facts in one breath.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yes, Suresh here." | Identity, recording line, pitch |
| C2 | "Haan, tell me." | Question one |
| C3 | "See, it's agricultural land, and anyway there's already a loan on it from the co-operative bank." | **Transfer close**, then end the call |

**Expected end state:** transfer.
**Pass checks:**
1. The transfer close is used, not the disqualification close.
2. The customer is never told they do not meet the criteria.
3. The agent collects no details about the co-operative bank loan.
4. The call ends.
5. Run the mirrored variant too, with the clauses reversed: "There's a loan on it already, and it's agricultural land." The outcome must be identical.

---

## T21 - Declines the eligibility questions at the pitch

**Goal:** a "no" to the pitch question has one branch and one attempt.
**Language / preset:** EN-M. **Customer:** Suresh Patel.
**Persona:** guarded, does not want to answer anything on a cold call.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yes, Suresh speaking." | Identity, recording line, pitch ending in the permission question |
| C2 | "No. I don't want to answer any questions on the phone. Just send it on WhatsApp or something." | Not-interested close, once, then end the call |

**Expected end state:** not interested.
**Pass checks:**
1. The not-interested close is used. One attempt, no second try.
2. No checklist question is asked, not even the first one.
3. The agent does not ask why, does not persuade, and does not offer a callback.
4. No WhatsApp message is promised. Closing without addressing the request is correct here, because the refusal outranks it on the ladder.
5. The call ends.

---

## T22 - "My company" must not infer salaried

**Goal:** the word company infers nothing. A business owner uses it too, so slot five A must still be asked.
**Language / preset:** EN-M. **Customer:** Suresh Patel.
**Persona:** owns a small firm of twelve people, talks about "my company" the way an employee would, and only reveals that he runs it when asked.

| # | Customer says | Agent must |
|---|---|---|
| C1 | "Yes, Suresh here." | Identity, recording line, pitch |
| C2 | "Haan, go ahead." | Question one |
| C3 | "Commercial. It's an office unit, ground floor." | Question two |
| C4 | "My name only." | Question three |
| C5 | "Yes, originals are with me." | Question four |
| C6 | "Sixty lakhs. See, it's for my company, we need some working capital." | Ask **slot five A**, the occupation. The word company must infer nothing |
| C7 | "Arre, I run the company. It's my own firm, twelve people working." | Fill five A as self-employed, then ask slot five B in the next turn |
| C8 | "Bank. Current account, everything comes there." | Question six |
| C9 | "Around two crore, that area is expensive now." | Question seven |
| C10 | "Eight years." | Handoff close, then end the call |

**Expected end state:** qualified.
**Pass checks:**
1. "My company" at C6 does **not** fill slot five A. The occupation question is asked at the next turn.
2. The agent never describes or treats the customer as salaried, in any turn.
3. "I run the company" at C7 fills five A as self-employed, and the occupation is never asked again.
4. Slot five B is asked separately, in its own turn, after C7.
5. Self-employed with bank income passes. All eight slots are filled and the handoff happens.
6. The agent does not jump from question four to the market value, which would mean it had inferred both halves of question five.

---

## Submission shortlist

Eight scenarios for the final recordings, chosen to cover every required path with no redundancy.

| Order | ID | Why it is in the set |
|---|---|---|
| 1 | T01 | Happy path, the reference call |
| 2 | T02 | Out-of-order handling, the hardest state-tracking requirement |
| 3 | T03 | Busy and callback, with date resolution |
| 4 | T05 | Disqualification with no reason named, including the pushback |
| 5 | T09 | Amount cap, crore normalisation, and the checklist resuming |
| 6 | T11 | Transfer path |
| 7 | T17 | Hindi, end to end, with gender forms |
| 8 | T15 | Boundary values at exactly seventy-five lakh and fifteen years |

If only six recordings are wanted, drop T15 and T03 and keep the rest. T16 is the most impressive call to listen to, so add it as a ninth if there is room.

---

## Results table template

Copy this into `tests/results.md`, which is already seeded with one row per scenario.

```
| ID | Scenario | Date | Platform call link | Pass / Fail | Failed checks | Notes and fix |
|----|----------|------|--------------------|-------------|---------------|---------------|
| T01 | Happy path | | | | | |
```

Use one row per attempt, not one per scenario, so the history of a fix stays visible. Record the prompt version in the notes column, for example "v1.1, re-run after the slot five fix".
