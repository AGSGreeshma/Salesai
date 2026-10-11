# Phase 5 - Platform Setup Notes

**Prompt to deploy:** `prompt/system_prompt.md` v1.3
**Scenarios to run after setup:** `tests/scenarios.md`, log in `tests/results.md`

Blanks marked `______` are for you to fill in from the platform itself. Nothing here should be guessed — every blank is something the docs or the UI will tell you, and guessing wrong costs a call from the free trial.

**Platform chosen:** `______` (Retell AI / Bolna / other)
**Date set up:** `______`
**Account email:** `______`
**Trial credit or minutes available:** `______`

---

## 1. Account creation

- [ ] Create the free-trial account
- [ ] Confirm the trial allows **outbound calls to Indian numbers**, or that web calls are enough for testing: `______`
- [ ] Confirm a phone number is included, or note what one costs: `______`
- [ ] Note the trial limits that will bite: minutes `______`, concurrent calls `______`, retention of recordings `______`
- [ ] Confirm **Hindi voices** exist on the plan: `______`
- [ ] Confirm **public sharing** of recordings and transcripts exists on the plan: `______`

If either of the last two is missing on the free tier, stop and reconsider the platform before building the agent. They are both submission requirements.

---

## 2. Create the agent

- [ ] Agent name: `______`
- [ ] Model selected: `______` (note the exact model; a weaker model will break the state tracker before the prompt does)
- [ ] Temperature or creativity setting: `______` (lower is better here; the prompt wants obedience, not invention)
- [ ] Where the system prompt goes: `______` (field name)
- [ ] Is there a separate **first message / greeting** field? `______`
  - If yes, decide: put the Stage A greeting there, or leave it empty so the prompt drives it. **Chosen:** `______`
  - If the platform forces a first message, the greeting is duplicated. Remove it from one place or the other.
- [ ] Prompt pasted and saved: `______`
- [ ] **Check nothing was truncated.** Scroll to the bottom of the saved prompt in the UI and confirm the critical rules recap and the section-order note are both there. This is P13. Result: `______`

---

## 3. Voice choice

- [ ] Voice for **male** agent (`agent_gender` = male): `______`
- [ ] Voice for **female** agent (`agent_gender` = female): `______`
- [ ] Is the voice fixed per agent, or settable per call? `______`
  - If fixed per agent, you need **two agents**, one male and one female, both with the same prompt. Note the two agent IDs: `______`
- [ ] Hindi quality check: have each voice read one line and judge it.
  - Test line: "नमस्ते, मैं रोहित बोल रहा हूँ, होम क्रेडिट से।"
  - Male voice verdict: `______`
  - Female voice verdict: `______`
- [ ] Hindi number check (P8). Test line: "आप पचहत्तर लाख रुपये तक का लोन ले सकते हैं।"
  - Does it say "पचहत्तर लाख" correctly? `______`
- [ ] English number check. Test line: "You can get up to seventy-five lakh rupees against your property."
  - Does it say "lakh" correctly, not "lack" or "lakes"? `______`
- [ ] Devanagari vs romanised script (P7): which does the TTS handle better? `______`
  - If romanised is clearly better, the Hindi lines in the prompt need transliterating. That is a prompt change, so log it as v1.3.

---

## 4. Variable mapping

**P1 - variable syntax.** The prompt uses `{{variable_name}}` throughout.

- Syntax the platform actually uses: `______`
- [ ] If it is not `{{var}}`, do a find-and-replace on a copy of the prompt. Do not edit `prompt/system_prompt.md`; save the platform version as `platform/system_prompt_<platform>.md` so the canonical file stays platform-neutral.
- What happens to an **unfilled** variable? `______` (reads the placeholder aloud / renders empty / errors)
- [ ] Test one deliberately empty variable on a web call and listen. Result: `______`

### Test values per preset

| Variable | EN-F | EN-M | HI-F | HI-M |
|---|---|---|---|---|
| `company_name` | Home Credit | Home Credit | होम क्रेडिट | होम क्रेडिट |
| `customer_name` | Rakesh Sharma | Suresh Patel | सुनीता देवी | महेश कुमार |
| `agent_name` | Priya | Rohit | प्रिया | रोहित |
| `agent_gender` | female | male | female | male |
| `current_date` | set per call | set per call | set per call | set per call |
| `current_day` | set per call | set per call | set per call | set per call |
| `current_time` | set per call | set per call | set per call | set per call |
| `additional_context_from_rag` | *(empty)* | *(empty)* | *(empty)* | *(empty)* |
| `language_to_speak` | English | English | Hindi | Hindi |
| `conversation_history` | see P2 below | see P2 below | see P2 below | see P2 below |
| `customer_utterance` | see P2 below | see P2 below | see P2 below | see P2 below |

Scenario-specific names override the preset: T06/T08/T10/T12/T14/T20/T21 use Suresh Patel, T07 uses Imran Qureshi, T15 uses Meena Iyer. See the preset table in `tests/scenarios.md`.

- [ ] Whether `company_name` and `customer_name` should be passed in Devanagari for Hindi calls, or in Latin script and left to the TTS: `______`
- [ ] Preset saved as a reusable test payload or template? `______`

---

## 5. P2 - conversation history and the current utterance

This is the most important thing to settle, because Sections 7 and 8 of the prompt depend on it.

- Does the platform inject the running transcript automatically? `______`
- Does it expose a variable for the history? `______` (name: `______`)
- Does it expose a variable for the latest customer turn? `______` (name: `______`)

Then pick one:

- [ ] **Auto-injected.** Leave `{{conversation_history}}` and `{{customer_utterance}}` unmapped or empty. The prompt already says to use the conversation so far when the variables are not supplied, so no prompt change is needed. Confirm on a live call that the agent does not re-ask an answered question: `______`
- [ ] **Variables must be filled.** Map them to the platform's own names and confirm the format (plain text, role-labelled turns, JSON): `______`
- [ ] **Neither exists.** Flag immediately. The state model cannot work without history and the platform is the wrong choice.

---

## 6. Ending the call - the `[END CALL]` mapping

**This blocks every live test.** Until it is done, the agent will either read the bracket aloud or never hang up.

- Mechanism the platform uses: `______` (end-call tool/function, transfer function, a trigger phrase, or a silence timeout)
- Exact name or phrase required: `______`

Then one of:

- [ ] **Function or tool.** Register it, note its exact name, and replace every `[END CALL]` in the platform copy of the prompt with the instruction to call that function. Count of occurrences to replace: **42** in v1.2 (41 in v1.1; the pitch-refusal branch added one)
- [ ] **Trigger phrase.** Configure the phrase and make sure it is something the agent would never say in normal speech. Phrase used: `______`
- [ ] **No mechanism.** Note the fallback: `______` (hang up manually, or rely on a silence timeout)

- [ ] Verify on a web call that the agent actually disconnects after a close, and does not read the token aloud. Result: `______`
- [ ] Verify it ends on a **disqualification** too, not only on the handoff. Result: `______`

---

## 7. Interruption and responsiveness (P5)

Settings to find and record. Names differ per platform.

| Setting | Platform name | Default | Set to | Why |
|---|---|---|---|---|
| Barge-in / interruption enabled | `______` | `______` | `______` | T16 requires the agent to stop when interrupted |
| Interruption sensitivity | `______` | `______` | `______` | Too sensitive and "haan", "achha" cut the agent off mid-question |
| Endpointing / silence before replying | `______` | `______` | `______` | Too short and it talks over a customer who is still thinking |
| Responsiveness / reply latency | `______` | `______` | `______` | |
| Silence timeout before re-prompt | `______` | `______` | `______` | T16 has a four-second silence; the prompt expects one re-prompt, then a second, then a close |
| Max call duration (P11) | `______` | `______` | `______` | Overrides the ~25-turn budget if lower |
| Max turns, if any (P11) | `______` | `______` | `______` | |
| Filler words / backchannel feature | `______` | `______` | `______` | If the platform adds its own "mm-hmm", it may collide with the prompt's acknowledgements |

Tuning note: set these once, run T16, then adjust. Do not tune them scenario by scenario, or the results stop being comparable.

---

## 8. Setting `current_date`, `current_day`, `current_time` per call

T03 cannot be verified unless these are right, because the check is whether the agent resolves "tomorrow" to the correct weekday.

- Does the platform offer built-in date/time variables? `______` (names: `______`)
- Can dynamic variables be set per call, from the dashboard or only via API? `______`
- Timezone the platform uses, and whether it can be set to IST: `______`

- [ ] **If built-ins exist**, map them and confirm the timezone is IST. Result: `______`
- [ ] **If values must be passed per call**, record the format expected (`2026-10-11`, `11 October 2026`, epoch): `______`
  - Decide the spoken-friendly format. The agent reads these aloud only indirectly, when confirming a callback, so a plain format like "11 October 2026" and "Saturday" and "4:30 pm" is safest. **Chosen:** `______`
- [ ] Before each test call, set the three values to the real current time. A stale date silently breaks T03.

---

## 9. Public call links

- Where recordings appear: `______`
- Where transcripts appear: `______`
- Is there a share or public-link option? `______`
- Does the link include audio, transcript, or both? `______`
- Link retention on the free tier: `______`

- [ ] Generate one public link from a smoke-test call: `______`
- [ ] **Open it in a logged-out or incognito browser.** It must load with no account. Result: `______`
- [ ] If public links are not available, note the fallback now, before making the final calls: `______` (download the audio and transcript and host them elsewhere)

---

## 10. Smoke test before the real runs

- [ ] One web call, EN-F preset, three turns only. Confirm:
  - The greeting says the real agent name and company, with no placeholder read aloud: `______`
  - "Hello" is used, with no time-of-day word: `______`
  - The recording line is said once, after the identity is confirmed: `______`
  - The pitch is followed by exactly one question: `______`
  - The call ends cleanly when you refuse the questions, which is T21: `______`
- [ ] One web call, HI-M preset, three turns. Confirm:
  - Everything is in Hindi, no English sentences: `______`
  - Male forms only: बोल रहा हूँ, not रही: `______`
  - No braces or slashes are spoken: `______`

Only after both smoke tests pass should you start burning trial credit on T01 to T21.

---

## 11. Open items to carry into Phase 6

| # | Item | Status |
|---|---|---|
| 1 | `[END CALL]` mapped and verified on both a handoff and a disqualification | `______` |
| 2 | P2 settled: history auto-injected or variables mapped | `______` |
| 3 | P13 checked: prompt not truncated in the platform UI | `______` |
| 4 | P11 recorded: max duration and max turns, and whether they undercut the ~25-turn budget | `______` |
| 5 | Hindi TTS script decision (Devanagari or romanised), and whether it forces a v1.3 | `______` |
| 6 | Two agents needed for the two genders, or one agent with a per-call voice | `______` |
| 7 | Public link verified logged out | `______` |
