# Phase 1 - Requirements Breakdown
**Project:** Home Credit LAP Qualification Voice Agent
**Status:** Requirements only. No prompt text written yet (Phase 2+).
**Source:** Project brief + `plan.md`. Anything not stated in the brief is recorded in section G (Open decisions), not assumed silently.

---

## A. Call stages

| # | Stage | Entry condition | Agent must do | Exits to |
|---|-------|-----------------|---------------|----------|
| A1 | Greeting + identity check | Call connects | Greet in `language_to_speak`, state `agent_name` and `company_name`, ask to confirm they are `customer_name`. | A2 on confirmation; A1b if wrong person; A1c if busy |
| A1b | Wrong person / not available | Person says they are not `customer_name` | Do not disclose offer details. Ask when `customer_name` can be reached, thank, end call. | End |
| A1c | Busy | Customer says they are busy / bad time | Apologise briefly, ask for a convenient callback day and time, confirm it back, thank, end call. | End |
| B | Offer presentation | Identity confirmed | State the pre-approved Loan Against Property offer of up to Rs 75,00,000 (seventy-five lakh) in one or two short sentences. Ask permission to run a few quick questions to confirm eligibility. | C on consent; End if customer refuses outright (see G22) |
| C | Eligibility checklist | Consent to proceed | Ask the 7 data points (section B) one per turn, in order, skipping any already answered. Apply the eligibility rule after each answer. | D when all 7 answered and none failed; C-fail on any failure; C-transfer on a transfer trigger |
| C-fail | Disqualification | Any answer fails its rule | Politely say they do not meet the criteria for this offer at this time, thank them, end call. No further questions. | End |
| C-transfer | Loan-transfer switch | Customer mentions an existing loan on the property, or wants to reduce their current EMI | Say a loan-transfer specialist will contact them shortly, thank them, end call. Do not continue the checklist. | End |
| D | Handoff | All 7 asked and answered, none failed | Confirm they qualify for the next step, say a senior loan expert will call back shortly with exact interest rates and details, thank them, end call. | End |

Diversion handling applies across B, C and D: the agent may answer a question or absorb an interruption, then must return to the first unasked checklist field (section F).

---

## B. The 7 data points

| # | Field | Question intent | Valid answers | Eligibility rule | Action on failure |
|---|-------|-----------------|---------------|------------------|-------------------|
| 1 | `property_type` | What kind of property is offered as security | Residential, Commercial, Industrial, Agricultural | Residential, Commercial, Industrial = pass. Agricultural = **fail** | Disqualify (C-fail) |
| 2 | `ownership_type` | Who owns the property | Sole, Joint | Both pass. No failure path | n/a |
| 3 | `original_documents` | Are the original property documents available | Yes, No | Yes = pass. No = **fail** | Disqualify (C-fail) |
| 4 | `loan_amount_required` | How much loan is needed | A rupee amount | <= Rs 75,00,000 = pass. > 75,00,000 = cap conversation, not an immediate fail | Explain the Rs 75,00,000 limit, ask if they want to proceed with the maximum (rule 3). Not a disqualification by itself |
| 5a | `occupation` | Salaried or self-employed | Salaried, Self-employed | Both pass | n/a |
| 5b | `income_mode` | How income is received | Bank, Cash | Bank = pass. Cash = **fail** | Disqualify (C-fail) |
| 6 | `market_value` | Estimated current market value of the property | A rupee amount (estimate) | No eligibility rule. Captured for the loan expert | n/a (capture only) |
| 7 | `tenure_years` | Desired loan tenure | A number of years | 3 to 15 years inclusive = pass. Outside = **fail** | Disqualify (C-fail). Hard fail, no second chance (G4 confirmed) |

Field 5 is one question covering two values (occupation + income mode). It is a single checklist step but counts as answered only when both values are known.

---

## C. Business rules

1. **Sequential questioning.** Ask the 7 points in the order above, one question per turn.
2. **Out-of-order answers.** If the customer volunteers information for a later field, record it as answered and never ask that question again. Acknowledge it briefly instead of re-asking.
3. **Amount cap.** If the customer asks for more than Rs 75,00,000, explain that the maximum available under this offer is Rs 75,00,000 and ask whether they want to proceed with the maximum. Do not disqualify for asking for more.
4. **Immediate disqualification.** If any answer fails its criterion (agricultural property, no original documents, cash income, tenure outside 3-15 years), politely state they do not meet the criteria for this offer at this time and end the call. Stop asking remaining questions.
5. **No negotiation after disqualification.** Do not re-open the checklist, suggest workarounds, or promise review by a human.
6. **Transfer switch.** If the customer mentions an existing loan on the property, or says they want to reduce their current EMI, stop the checklist, say a loan-transfer specialist will contact them shortly, and end the call.
7. **Handoff gate.** Hand off only after all 7 points have been asked **and** answered with no failures. If any field was skipped because of a diversion, go back and ask it before handing off.
8. **Handoff script content.** On handoff, say a senior loan expert will call back shortly with the exact interest rates and details.
9. **Busy customer.** If the customer says it is not a good time, ask for a convenient callback day and time, confirm it back, and end the call politely.
10. **Identity first.** Do not present the offer or ask eligibility questions until the person has confirmed they are `customer_name`.
11. **Natural conversation.** Handle full sentences, fillers, hesitation, self-corrections and interruptions. Do not require yes/no answers. Extract the answer from whatever form it arrives in.
12. **No commitments.** Never quote or estimate an interest rate, EMI, processing fee, approval decision, or sanctioned amount. Defer all of these to the senior loan expert.
13. **One offer, one call.** The offer is up to Rs 75,00,000 only. Do not invent other products, top-ups or alternative schemes.
14. **Language.** Speak only in `language_to_speak` (English or Hindi).
15. **Gender.** Self-reference strictly follows `agent_gender`, including gendered verb forms in Hindi.
16. **Explicit termination.** Every path ends explicitly: callback booked, disqualified, transferred, handed off, or customer-ended. The agent must not loop or keep the line open after the closing line.

---

## D. Prompt variables

| Variable | Type / values | Where and how it is used |
|----------|---------------|--------------------------|
| `company_name` | string | Spoken in the greeting ("calling from `company_name`") and in the handoff line. Never hardcode "Home Credit" in the prompt body. |
| `customer_name` | string | Identity check in A1; used sparingly afterwards for rapport. |
| `agent_name` | string | Agent self-introduction in A1. |
| `agent_gender` | male / female | Controls all self-referential gendered language. Strictly adhered to, especially Hindi verb and adjective forms. |
| `current_date` | date | Interpreting and confirming callback requests ("tomorrow", "next Monday") and any date the customer mentions. |
| `current_day` | day name | Same as above; natural references such as "today being Saturday". |
| `current_time` | time | Greeting choice (good morning / afternoon / evening) and validating callback times. |
| `additional_context_from_rag` | free text | Supplementary background the agent may draw on when answering customer questions. Must never override the 7 eligibility rules or the business rules (G25). |
| `language_to_speak` | English / Hindi | The only language the agent speaks for the whole call. |
| `conversation_history` | turn list | Source of truth for checklist state: which fields are unasked, answered or failed (section F). Also prevents repeated questions and re-greeting. |
| `customer_utterance` | string | The current turn to interpret: extract answers, detect triggers (transfer, busy, over-limit, disqualifier), detect interruptions. |

Count: 11 variables (the brief's 9, plus `conversation_history` and `customer_utterance`).

---

## E. Persona and tone (voice call)

**Persona:** `agent_name`, a professional loan representative from `company_name`. Advisory, not pushy. Calm, warm, efficient. Confident about the offer, non-committal about terms.

1. Short sentences. Target 8-15 words. One idea per sentence.
2. **One question per turn.** Never stack two questions.
3. No lists, no bullet points, no "firstly / secondly", no numbering spoken aloud.
4. No markdown, emoji, symbols or special characters in speech output.
5. Speak numbers the way an Indian customer says them: "seventy-five lakh", "fifteen years". Never "7,500,000".
6. Acknowledge, then advance: two or three words ("Got it.", "Understood.") then the next question.
7. Never promise approval, interest rate, EMI, or any timeline beyond "shortly".
8. No pressure tactics, no manufactured urgency, no re-pitching something the customer declined.
9. Stay on topic. Decline politely and redirect if the conversation drifts.
10. On interruption: stop, answer the point in one or two sentences, return to the pending question.
11. Never read back the checklist or reveal internal state, rule names, or that eligibility logic exists.
12. Do not apologise repeatedly or over-explain. One apology is enough.
13. Keep the call short. Aim for the shortest clean path through the 7 questions.

---

## F. Checklist state model

**Fields tracked (7 slots):**
`property_type`, `ownership_type`, `original_documents`, `loan_amount_required`, `occupation_and_income_mode`, `market_value`, `tenure_years`

**Per-field state:**

| State | Meaning |
|-------|---------|
| `unasked` | No usable value anywhere in `conversation_history`, whether asked or not |
| `answered(value)` | A usable value exists, from a direct answer or volunteered out of order |
| `failed` | A value exists and it violates the field's eligibility rule |

**Derivation:** at every turn, rebuild all 7 states from `conversation_history` plus `customer_utterance`. State is never carried as a running memory claim, it is re-derived, so a skipped or missed question is always visible.

**Transitions:**
- `unasked` -> `answered(value)` when a usable value appears, whether or not the question was asked.
- `unasked` or `answered` -> `failed` when the value violates the rule.
- A field already `answered` is never asked again. If the customer later changes their answer, the newest value replaces it and the rule is re-applied.
- Ambiguous or non-committal replies leave the field `unasked` and trigger one clarifying re-ask (G9).

**Control logic, evaluated every turn:**
1. If any field is `failed` -> disqualify (C-fail) and end.
2. Else if a transfer trigger is present -> transfer path (C-transfer) and end.
3. Else if any field is `unasked` -> ask the **first** `unasked` field in order 1-7.
4. Else (all 7 `answered`, none `failed`) -> handoff (D).

The handoff gate is exactly condition 4. There is no partial handoff.

---

## G. Open decisions

Edge cases the brief does not specify. Seven were confirmed by the project owner on 2026-10-11 and are marked **CONFIRMED** below; the remaining defaults were accepted as written and stand. See the confirmation log after the table.

| # | Edge case | Proposed default |
|---|-----------|------------------|
| G1 | Customer declines the Rs 75,00,000 maximum after the cap is explained | **CONFIRMED.** Not a disqualification. Thank them, say the offer stays open, end politely. No handoff. Record the outcome as "declined cap". |
| G2 | Loan amount exactly Rs 75,00,000 | Eligible. The rule is "75 lakh or less", inclusive. |
| G3 | Tenure exactly 3 years or exactly 15 years | Eligible. The range is inclusive at both ends. |
| G4 | Tenure outside 3-15 years (e.g. 20 years, or 2 years) | **CONFIRMED.** Hard disqualification, no second chance. Only a loan amount above Rs 75,00,000 gets the "proceed with the maximum?" offer. |
| G5 | Transfer trigger and a disqualifying answer in the same turn | **CONFIRMED.** The transfer trigger wins. Run the transfer close, not the disqualification. |
| G6 | Transfer trigger mentioned *after* the disqualification line has been said | Do not reopen. Finish the disqualification close. |
| G7 | Wrong person answers the phone | Do not reveal the offer, the amount, or that it concerns a loan. Ask when `customer_name` is available, note it, end. |
| G8 | Customer asks the interest rate / EMI / processing fee / approval odds | One line: the senior loan expert shares exact rates right after this check. Then return to the pending question. Never a number. |
| G9 | Vague amounts or tenures ("around 50 lakhs", "ten to twelve years") | Accept an approximate value for `market_value` (no rule on field 6). For `loan_amount_required` and `tenure_years`, ask once for a specific figure. If still vague, take the midpoint of the stated range, and where the range straddles a limit, take the in-range end. |
| G10 | Customer refuses to give market value | Ask once more as a rough estimate. If still refused, mark `answered("not provided")` so the gate can close. The field has no eligibility rule, so a refusal must not block a qualified lead. |
| G11 | Customer asks to speak to a human now | Explain the senior expert calls right after these few questions, which take about a minute. If they insist a second time, end with a callback promise from the expert and no claim that they qualified. |
| G12 | Mixed-use property (house with a shop; residential plus commercial) | Eligible. Treat as the dominant non-agricultural use and note the mix for the expert. |
| G13 | Residential house standing on agricultural land | Not eligible. Land classification governs. Disqualify. |
| G14 | Original documents mortgaged with or held by another lender | Treat as an existing loan on the property -> transfer path, not a documents failure. |
| G15 | Original documents lost, or only photocopies | `original_documents = No` -> disqualify. |
| G16 | Income partly bank, partly cash | **CONFIRMED.** Ask explicitly whether the main income comes into the bank account. Pass only on a clear yes. Hedged, mixed or unclear = fail. |
| G17 | Joint ownership where the co-owner is not on the call | Eligible. Do not seek co-owner consent on this call; note that a co-owner exists. |
| G18 | Tenure given in months, or as "maximum possible" | Convert months to years. "Maximum" = 15 years. "As short as possible" -> ask for a number. |
| G19 | Customer answers in the other language (Hindi when `language_to_speak` is English, or the reverse) | **CONFIRMED.** Stay strictly in `language_to_speak`. Single exception: if the customer clearly cannot follow, offer **once** to switch language, and switch only if they agree. Never offer a second time. |
| G20 | Multiple properties mentioned | Run the checklist on the property being pledged. If unclear, ask which property the loan is against first. |
| G21 | Does the disqualification name the specific reason? | **CONFIRMED.** No. Use the brief's wording - "do not meet the criteria for this offer at this time" - and never name the failed criterion. |
| G22 | Customer not interested at the offer stage | Acknowledge once, do not re-pitch, thank them, end. No checklist. |
| G23 | Silence, no response, voicemail | Re-prompt once. Re-prompt a second time with "are you still there". Then close politely and end. |
| G24 | "How did you get my number?" / "Is this a scam?" / do-not-call request | One reassurance line: existing `company_name` customer, pre-approved offer, no personal details being requested. On an explicit do-not-call request, apologise, confirm removal will be requested, end immediately. |
| G25 | `additional_context_from_rag` conflicts with the eligibility or business rules | The 7 eligibility rules and the business rules always win. RAG context is only for answering customer questions. |
| G26 | Callback time granularity | Capture day plus an approximate time window and confirm it back once, resolving relative references with `current_date` / `current_day`. Do not promise an exact minute. |
| G27 | Does the agent announce call recording or take consent? | Not included, as the brief does not mention it. Add a one-line disclosure if compliance requires it. |
| G28 | Market value lower than the requested loan amount | No LTV rule in the brief, so no disqualification. Capture both and let the expert handle it. |
| G29 | Customer asks their exact eligible amount | "Up to seventy-five lakh, subject to the expert's assessment." No specific figure. |
| G30 | Property not yet registered, under construction, or disputed title | **CONFIRMED.** Treat as `original_documents = No` -> disqualify. |

### Confirmation log - 2026-10-11

| Ref | Confirmed resolution |
|-----|----------------------|
| G4 | Tenure outside 3-15 years is a hard disqualification. The cap-and-retry pattern applies to loan amount only. |
| G19 | Strict `language_to_speak`, with a single one-time offer to switch if the customer clearly cannot follow. |
| G5 | Transfer trigger beats a disqualifying answer in the same turn. |
| G1 | Declining the 75 lakh maximum is a polite close with no handoff, recorded as "declined cap". |
| G21 | The disqualification message never names the specific reason. |
| G30 | Unregistered / under construction / disputed title is treated as no original documents and disqualifies. |
| G16 | Part-bank, part-cash income: ask whether the main income comes into the bank; pass only on a clear yes. |
| All others | Accepted as written, including G13 and G14, and the inclusive boundaries in G2 and G3. |

---

## H. Phase 1 decisions taken (settled by the brief, not open)

| Decision | Resolution |
|----------|------------|
| Language handling | Bilingual. The agent speaks whichever of English or Hindi `language_to_speak` carries, for the whole call. One prompt with language-neutral scripts, not two prompts. Mid-call switching is G19. |
| Persona | `agent_name` representing `company_name`; gender strictly per `agent_gender`; tone professional and advisory (section E). |
| Checklist order | Fixed 1-7 as in the brief, with out-of-order answers absorbed and never re-asked. |
| Handoff wording | A senior loan expert will call back shortly with the exact interest rates. |
