# Workflow: Project Notes → Fact-Checked LinkedIn Post

**Pipeline type:** draft → critique → revise (writing pipeline from the audit)
**Tool:** n8n (self-hosted, Oracle Cloud) + Anthropic API
**File:** `linkedin-post-pipeline.json` (import directly into n8n)

## Why this pipeline

Every CodeCelix/FlyRank project ends with the same manual task: write a LinkedIn post or progress report, then re-read it myself to make sure I haven't stated an unverified metric as fact or misattributed team work as solo. This automates the drafting and makes the fact-check step a hard gate instead of a mental note.

---

## 1. Step diagram

```
[Manual Trigger]
      |
      v
[STEP 1: GATHER]  <- I fill 8 structured fields (no free text blob)
      |  project_name, what_built, tech_stack, timeframe,
      |  team_or_solo, verified_metrics, unverified_claims, post_type
      v
[STEP 2: SYNTHESIZE]  (Claude call 1)
      |  -> JSON: hook_angle, key_facts[{fact, status: verified|unverified}], tone_notes
      v
[STEP 3: DRAFT]  (Claude call 2)
      |  -> plain text LinkedIn post, built only from tagged facts
      |     voice card enforced: direct, decision-driven, honest about limits, no hype
      v
[STEP 4: CRITIQUE]  (Claude call 3)
      |  -> JSON: {pass: bool, issues: [string]}
      |     checks: unverified-as-fact, hype language, attribution mismatch, voice drift
      v
   [Pass?]
    /    \
  yes     no
   |       |
   v       v
[5b: FORMAT]  [5a: REVISE]  (Claude call 4, fixes listed issues)
   |               |
   +-------+-------+
           v
   final_post (ready for my read, not auto-posted)
```

Handoff contract between every step is JSON with named fields — nothing free-floats as an unstructured blob, which is what makes step 4 able to gate step 3's output automatically.

---

## 2. Every prompt/configuration used

**Step 2 — Synthesize (system prompt):**
> You extract structured narrative facts from raw project notes for a LinkedIn post. Output ONLY valid JSON, no markdown fences, no preamble. Schema: `{hook_angle, key_facts: [{fact, status: "verified"|"unverified"}], tone_notes}`. Mark a fact unverified if it came from the unverified_claims field or if a metric has no clear source. Never invent facts not present in the input.

**Step 3 — Draft (system prompt):**
> You write LinkedIn posts in this exact voice: direct, decision-driven, honest about limits, no hype. No emoji-stacking, no "thrilled to announce," no fake humility, no exclamation-point stacking. Short sentences. Concrete technical detail over vague enthusiasm. Use ONLY facts tagged 'verified' as claims of fact. Unverified facts may only appear with explicit hedge language or be omitted. Team work must be attributed as team work.

**Step 4 — Critique (system prompt):**
> You are a strict editor checking a LinkedIn draft against a fact sheet. Flag: (1) any claim not in verified key_facts, (2) unverified fact stated as settled, (3) hype language, (4) solo/team attribution mismatches, (5) voice drift. Output ONLY `{pass: bool, issues: [string]}`.

**Step 5a — Revise (system prompt, only fires if pass=false):**
> Revise the LinkedIn draft to fix every issue listed. Keep the voice: direct, decision-driven, honest about limits, no hype. Output only the revised post text.

Model used in all four calls: `claude-sonnet-5` (verify against current Anthropic docs before running — model IDs change).

---

## 3. Five runs

Run the workflow on 5 real projects. Log actual outputs here — do not backfill with invented numbers.

| # | Project | Input summary | Step 2 flagged unverified? | Step 4 pass on first try? | Issues caught | Final post used? |
|---|---------|---------------|------------------------------|------------------------------|----------------|-------------------|
| 1 | | | | | | |
| 2 | | | | | | |
| 3 | | | | | | |
| 4 | | | | | | |
| 5 | | | | | | |

Good candidates from current work: Clinic Lead Gen (n8n system), FashionHub, INFILOFT deployment milestone, Body Reshaping fingerprint fix, FlyRank Week 3-8 capstone progress.

For each run, paste: (a) the 8 input fields, (b) Step 4's raw JSON output, (c) final post text.

---

## 4. Time accounting

| | Manual (no pipeline) | With pipeline |
|---|---|---|
| Setup cost (one-time) | — | *log actual n8n build + prompt tuning time* |
| Per-post time | *time yourself writing + self-editing one post normally* | *time from filling Step 1 fields to final_post* |
| Break-even point | — | setup cost ÷ (manual time − pipeline time per post) |

Fill with real stopwatch numbers from the 5 runs, not estimates. If revise (5a) fires often, per-post time goes up — note whether that's still faster than manual self-editing.

---

## 5. Known failure points / required human review

- **Step 1 is the weak link.** Garbage or vague input (e.g. "made it faster") produces a synthesis with nothing concrete to hook on. The pipeline can't invent specificity you didn't give it.
- **Critique (Step 4) is a second AI judging a first AI's output on the same underlying facts** — it will catch hype language and obvious unverified-as-fact slips reliably, but it can miss subtler over-claiming if the fact sheet itself was already loosely worded in Step 2. Read the final post yourself regardless of `pass: true`.
- **No fact-checking against reality** — the pipeline only checks internal consistency (does the draft match what I told it), not whether what I told it is actually true. If I mis-tag an unverified metric as verified in Step 1, nothing downstream catches that.
- **Team attribution depends on me stating it correctly in team_or_solo.** The pipeline enforces consistency, not ground truth.
- **Voice drift over many runs** — worth spot-checking every 5-10 posts against the actual voice card, since small drift can compound and the critique step is checking against its own understanding of the voice description, not a moving benchmark of my real writing.
- **Human checkpoint before publish, always.** This pipeline is not wired to auto-post to LinkedIn — final_post lands in n8n's execution log for manual copy-paste, by design.

---

## 6. Pass/revise self-check

- [ ] Ran end to end on a new input not used while building
- [ ] 3+ distinct steps with defined handoffs — yes, 5 steps (or 6 counting the revise branch)
- [ ] 5 real runs logged with actual outputs above
- [ ] Time accounting includes setup cost, not just per-run time
- [ ] Failure points and human-review points named above
