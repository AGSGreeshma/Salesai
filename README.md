# Home Credit LAP Voice Agent

A system prompt for an AI voice agent that phones existing Home Credit customers, presents a pre-approved Loan Against Property (LAP) offer of up to ₹75,00,000, runs a seven-point eligibility check, and hands qualified leads to a senior loan expert.

The prompt was built and tested on the Sarvam voice-agent platform with real voice calls.

## What is in this repo

| Path | What it is |
|---|---|
| `prompt/system_prompt.md` | The final system prompt (v1.3). This is the submission prompt and uses all 11 variables. |
| `prompt/drafts/` | Earlier versions (v1, v1.1, v1.2). |
| `tests/scenarios.md` | Test matrix and customer scripts. |
| `tests/results.md` | Pass/fail log and the fixes made. |
| `platform/setup_notes.md` | How the agent was configured on Sarvam. |
| `transcripts/` | Transcripts of the recorded calls (see below). |
| `requirements.md`, `design.md`, `plan.md` | Requirement breakdown, prompt design and the work plan. |

Call recordings (audio) and call logs are in the shared Drive folder: **<paste Drive folder link here>**

## What the agent does

1. **Greeting and identity.** Confirms it is speaking to the right customer. If the customer is busy, it asks for a callback time and ends the call. A wrong person gets no loan details.
2. **Offer.** Gives a single line that the call may be recorded, then explains the pre-approved LAP offer of up to 75 lakh.
3. **Eligibility checklist.** Collects seven data points, one question at a time:

| # | Data point | Rule |
|---|---|---|
| 1 | Property type | Residential, commercial or industrial are eligible. Agricultural is not. |
| 2 | Ownership | Sole or joint, both eligible. |
| 3 | Original documents | Must be available. |
| 4 | Loan amount | 75 lakh or less. Above that, the agent explains the cap and asks whether to proceed with the maximum. |
| 5 | Occupation and income mode | Salaried or self-employed, and income must arrive in the bank. Cash income is not eligible. |
| 6 | Market value | Recorded, no rule. |
| 7 | Tenure | 3 to 15 years inclusive. |

4. **Handoff.** Only after all seven are answered and eligible, it says a senior loan expert will call back shortly with the exact interest rates, then ends the call.

## Business rules covered in the prompt

- **Immediate disqualification:** a polite close that says the customer does not meet the criteria for this offer at this time. The agent never names the reason.
- **Transfer switch:** an existing loan on the property or a wish to reduce the current EMI ends the call with a loan-transfer specialist handoff. This wins over a disqualifying answer given in the same turn.
- **Amount cap:** above 75 lakh, the agent explains the limit and offers the maximum. If the customer declines, the call closes politely.
- **Out-of-order answers:** anything the customer volunteers early is noted and never asked again.
- **Handoff gate:** no handoff until all seven are answered. A skipped question is asked after any diversion.
- **Natural conversation:** handles full sentences, fillers, interruptions, "is this a scam?" and "what is the interest rate?" without promising rates, approval or disbursement.
- **Gender and language:** `agent_gender` is followed strictly, including Hindi verb forms. The agent speaks only `language_to_speak` (English or Hindi).

## Prompt variables

`company_name`, `customer_name`, `agent_name`, `agent_gender`, `current_date`, `current_day`, `current_time`, `additional_context_from_rag`, `language_to_speak`, `conversation_history`, `customer_utterance`.

## How the prompt is structured

Sixteen sections, including a priority ladder for resolving conflicts between rules, a never-do list, and an internal state tracker for the seven checklist items that is rebuilt from the conversation every turn. That rebuild is what stops the agent from repeating a question or skipping one.

## Platform copy vs. submitted prompt

`prompt/system_prompt.md` is the canonical prompt with all 11 `{{variables}}`. On Sarvam, the agent runs a resolved copy: agent name (Priya), gender (female), language (English) and company are written in directly, the customer name is inserted in the greeting only, and conversation history and the latest utterance are supplied by the platform itself. The date, day and time are added as platform variables. The rules and flow are identical.

## Recorded test calls

All calls were live voice calls against the English, female agent. Transcripts are in `transcripts/`.

| # | Scenario | Result |
|---|---|---|
| 01 | Happy path, all seven answered in order | Qualified, handoff line, call ended by agent |
| 02 | Out-of-order answers (occupation, income and amount volunteered early) | Not re-asked, remaining questions only, handoff |
| 03 | Customer is busy | Callback time confirmed ("tomorrow, Monday, around ten"), no offer details shared |
| 04 | Cash income | Polite disqualification, no reason named |
| 05 | Loan of 1 crore, customer accepts 75 lakh | Cap explained, checklist completed, 15-year tenure accepted, handoff |
| 06 | Loan of 1.2 crore, customer declines 75 lakh | Polite close, no further questions |
| 07 | Existing home loan and wish to lower EMI, with documents "with the bank" | Transfer specialist handoff, not disqualified |

## Not recorded

Covered by the prompt but not recorded as calls: Hindi calls, the male-agent variant, wrong person answering, agricultural property, tenure outside 3 to 15 years, and a mid-checklist diversion. Their scripts are in `tests/scenarios.md`.

## Key decisions

- Tenure outside 3 to 15 years is a hard disqualification. Exactly 3 or 15 is eligible.
- Only a loan amount above 75 lakh triggers the "proceed with maximum" offer.
- Unregistered, under-construction or disputed property is treated as having no original documents.
- A part-bank, part-cash income needs a clear yes that the main income is in the bank.
- The incomplete-information close promises no callback.
- Asked directly, the agent says it is an AI voice assistant.
- "Company" alone does not imply salaried. Words such as salary, job or employee do. Business, shop or "I run" imply self-employed. Income mode (bank or cash) is never inferred.

## Limitations

- Audio quality and pacing were judged by ear on live calls. The transcripts are copied word for word from the platform's call logs.
- Test calls showed no usage cost on the platform.
