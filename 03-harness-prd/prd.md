# AI PRD · Juno
 
_Module 3. Draft v3, updated with PM interview answers and the prototype's sample output (juno-feedback-pulse.lovable.app). Items marked **[CONFIRM]** are still open._
 
## Problem & user
 
**User:** The RocketShip Product Manager, preparing for the weekly backlog refinement session.
 
**Problem:** Each week the PM manually reviews a ~350-row BI export of product signals (usage, support, Jira and feedback) to decide what deserves attention. This takes about 2 hours, and important issues can be buried or overweighted because a few comments are loud.
 
**What Juno does:** Juno turns the uploaded weekly signal report into a short, ranked list of top concerns and recurring themes. Every concern cites exact Signal IDs with the original quote, so the PM can verify any claim quickly. Every recommendation is labelled "Proposed, requires PM review." Juno proposes; the PM decides.
 
**Success metrics**
- **Hours saved:** from about 2 hours to under 15 minutes per week.
- **Adoption:** how many of Juno's top 3 concerns make it into refinement (target: 2 of 3 or better).
- **Trust (guardrail):** 100% of cited Signal IDs exist in the source file and the quoted text matches it.
## The harness
 
| Surface | Specification |
|---|---|
| 01 Context | Weekly BI export (CSV, ~350 rows) with `id, source, timestamp, text, account, severity, feedback, comment`. PM sets a start date (currently Sept 1, 2026) and the weekly window is the previous Monday to Sunday. HR-source signals are excluded. No names or account identifiers in output. |
| 02 Tools | Draft top concerns and themes with cited Signal IDs; draft top 3 user stories; send high-priority alerts to the PM in MS Teams. |
| 03 Loop | One pass per uploaded batch. One automatic retry on failed verification, then stop and hand off to the PM. |
| 04 Memory | PM-only. Last 4 weeks of approved outputs, used to label concerns new / recurring / fading and to remember which were actioned. Saved after PM approval; PM can edit or delete. |
| 05 Permissions | read auto · draft auto · write confirm · send auto to the PM only, blocked for anyone else |
| 06 Verification | Every cited Signal ID and quote must match the source file. Single-signal concerns are labelled as such. Exclusion rules are checked before output. |
 
## 01 Context · Data Requirements
 
**Required sources**
- One BI export per run, uploaded by the PM. The export contains all data since before launch, so Juno trims it before analysis.
- Fields: `id, source, timestamp, text, account, severity, feedback, comment`. `severity` is set manually by a human reviewer (values seen so far include P0, P1 and "escalation").
- **Start date** (set by the PM; Sept 1, 2026 for now). It bounds the **themes** view, which looks at everything since that date.
- **Weekly window:** the previous Monday to Sunday. It bounds the **top concerns** view. Any partial week after that window is ignored for weekly concerns and reported in limitations.
**Definition of "negative"**
A signal counts as negative or escalated when its feedback is marked negative, or its severity is P0 or escalation. Blank feedback is not treated as negative.
 
**Exclusions (deliberate)**
- HR-source signals are excluded entirely, and Juno reports how many were excluded.
- Signals dated before the start date are excluded and counted.
- Signals with blank text are flagged, not analysed.
- Juno may read the `account` field to count distinct accounts per concern. Names and account identifiers never appear in output, Teams messages or memory; only the counts do. No customer PII or confidential financial information appears in drafts, Teams messages or memory.
**Freshness and failure**
- Juno runs when the file is uploaded and asks for the file in Teams on Monday at 8:00 AM.
- If the file is missing, unreadable or lacks required columns, Juno stops, names the problem and asks for a corrected file. It does not guess.
## 02 Tools · System Capabilities
 
| Verb | Input | Output | Side-effect class |
|---|---|---|---|
| Draft top 3 concerns and major themes | Trimmed CSV, start date, memory | Ranked concerns (negative/escalated count, total, distinct accounts, Signal IDs, quotes) and themes (occurrences, evidence threshold) | draft |
| Draft top 3 user stories | The top concerns | Three user stories, each linked to its source concern and Signal IDs (to be added to the prototype) | draft |
| Send Teams alert | A concern meeting the high-priority rule | Short message to the PM with the issue, why it matters and Signal IDs | send |
| Save run to memory | Reviewed output | A 4-week record | write |
 
**Evidence rules (from the prototype)**
- A theme needs at least 2 distinct Signal IDs to appear.
- A concern based on a single signal is labelled "based on a single signal."
**High-priority rule (confirmed by the PM; revisit after a few weeks of real use)**
A concern triggers a Teams alert if it has either (a) any P0 signal, or (b) 2 or more negative/escalated signals in the weekly window. In the sample week, "Outages & data failures" would trigger an alert on rule (b); the other two concerns would not.
 
**Deliberate omissions**
- Juno does not message anyone but the PM, and does not assign user stories to anyone.
- Juno does not create backlog items or decide what ships.
## 03 Loop · AI Costs & Latency
 
**Turn ceiling and escalation**
- One analysis pass per uploaded batch.
- If verification fails, Juno retries once. If it fails again, it stops and gives the PM a partial result with failures marked.
- Juno does not re-run without a new upload or PM request.
**Latency and cost target**
- About 1 minute per run for a ~350-row file.
- Cost capped at $200 per week across all runs and re-runs. Juno reports the actual cost of each run, and stops and tells the PM if a re-run would exceed the weekly cap.
## 04 Memory · Data Requirements
 
**What persists, at what scope**
- The last 4 weeks of Juno's outputs: ranked concerns, themes and Signal IDs only.
- Scope: the PM's own memory only. It is not shared with other users.
- Also stored: which concerns the PM pulled into refinement ("actioned"), as a status on that concern.
**What memory is used for**
- **Trend labels:** each concern is labelled "new" (not in the last 4 weeks), "recurring" (appeared in one or more prior weeks) or "fading" (appeared in the previous week and has fewer signals this week than last week). Example: pipeline failures appearing three weeks running would be labelled recurring.
- **Actioned items:** a concern the PM marked as actioned is not re-recommended as a fresh priority. If new negative signals arrive for it, Juno shows it as "actioned, still receiving signals" so the PM can see whether the action worked.
**Expiry and write rules**
- Each entry expires after 4 weeks.
- A run is saved to memory only after the PM approves it (write = confirm). Rejected runs are not stored.
- Marking a concern as actioned is a PM action, done with a button on each concern in the results screen. Juno does not infer it.
- The PM can view, edit and delete any week's entry, for example if Juno mislabelled a concern.
- No names, accounts or customer financials are stored.
## 05 Permissions · AI Risks & Mitigations
 
| Side-effect class | Concrete actions | Tier |
|---|---|---|
| read | Read the uploaded CSV and memory | auto |
| draft | Draft concerns, themes and user stories | auto |
| write | Save a run to memory | confirm |
| send | Teams message to the PM | auto |
| send | Message to anyone else | blocked |
 
**Justification:** Reading and drafting are reversible and stay in the session. Saving to memory changes what Juno will rely on later, so the PM approves it. Messages to the PM alone carry no outside exposure, so they are automatic; anything that could reach another person is blocked outright.
 
**Key risks and mitigations**
- **Loud-voice bias:** a few vocal accounts dominate the ranking. Mitigation: show distinct-account counts alongside totals, and flag single-signal concerns.
- **Over-trust:** the PM accepts the list unchecked. Mitigation: every item cites Signal IDs and quotes, and all actions say "requires PM review."
- **Generic actions:** proposed actions such as "audit the access flows" are not specific. Mitigation: treat them as prompts for discussion, and consider improving specificity later.
- **Manual severity inconsistency:** severity is set by a human, so labelling may vary. Mitigation: show the severity as recorded and never reinterpret it.
- **Data leakage:** HR exclusion, no names or accounts in outputs, nothing sensitive in Teams.
## 06 Verification · AI Testing & Measurement
 
**The check before output ships**
1. Every cited Signal ID exists in the uploaded file.
2. Every quote matches the source text for that ID.
3. Every count (negative, total, distinct accounts, occurrences) can be recomputed from the file.
4. Every theme meets the 2-distinct-ID threshold.
5. No HR-source signal, name or account identifier appears in the output.
6. Date logic is correct: concerns fall in the weekly window, themes fall after the start date.
**Failure behaviour**
- A concern with a single signal stays in the list, labelled as such.
- A failed check on citations or counts triggers one retry, then a handoff to the PM with the failure marked.
- A failed exclusion check (5) blocks the whole output.
- If fewer than 3 concerns have enough evidence, Juno shows what it has rather than padding the list.
**Data quality section (always shown)**
Juno reports how many signals were excluded for date, HR source, blank text or partial week, so the PM knows what was left out.
 
## Eval plan
 
_Draft, to be refined in Module 6. Golden-set cases:_
1. Clean week with clear top concerns.
2. Thin week where concerns rest on a single signal.
3. Corrupted file or missing columns (should stop and ask).
4. File with HR rows, pre-start-date rows and blank-text rows (should exclude and report counts).
5. Partial week at the end of the file (should be ignored and reported).
6. A concern recurring across weeks (should use memory to label it recurring).
7. A quote altered or an ID invented (verification should catch it).
**Golden set:** 4 to 6 past weeks of real (sanitised) exports, plus the synthetic cases above. For the real weeks, the PM records their own top 3 concerns in advance as the reference answer.
 
**Pass thresholds** (values are first proposals)
 
| Check | Threshold |
|---|---|
| Cited Signal IDs exist in the file | 100% |
| Quotes match source text | 100% |
| Counts recomputable from the file | 100% |
| Exclusions respected (no HR rows, names or accounts in output) | 100%, zero tolerance |
| Themes meet the 2-distinct-ID rule; single-signal concerns labelled | 100% |
| Teams alerts match the high-priority rule (none missed, none extra) | 100% |
| Juno's top 3 overlaps the PM's reference top 3 by at least 2 | at least 80% of golden weeks |
| Trend labels (new / recurring / fading) correct against memory | 100% on synthetic cases |
| Latency | under 60 seconds |
| Cost | within the $200 weekly cap |
 
**Regression cadence**
- Run the full golden set whenever the prompt, model, skill or verification rules change.
- Run it monthly even if nothing changed, to catch drift.
- Log the six verification checks on every real weekly run, and review any failure the same week.
## Open questions
- None at this time. Revisit the high-priority rule and the success targets after a few weeks of real use.
## Prototype follow-ups
- Add the drafted top 3 user stories to the results screen, each linked to its concern and Signal IDs.
- Add the Teams alert (a mock message to the PM) for concerns that meet the high-priority rule.
- Merge the "Recommended Priorities" section into the top concerns table, since it repeats it, and make proposed actions more specific.
- Add an "Approve and save this run" button, trend labels (new / recurring / fading) and a "Mark as actioned" button on each concern.
- Show the verification checks (pass/fail) so the PM can see what Juno verified.
## Out of scope
 
Any capability not in the verb list above, and any action tiered `blocked`. Both are decisions on the record, not omissions.
 
