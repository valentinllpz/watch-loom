## Loom and design analysis

Run only what applies. Read the ticket first (title, description, acceptance criteria, attachments); it decides
which of the two parts run.

**Skip rules (check before doing anything):**
- No `loom.com/share` link in the PR body → skip the Loom part entirely.
- No UI in the diff (`.erb`, components, JS/TS, CSS) **and** no design reference or design instruction in the ticket
  (Figma link, claude.ai artifact, screenshot or mockup attachment, "match the design", DESIGN.md) → skip the design
  part; write `"design": {"status": "skipped", "why": "<no UI change / no design reference>"}`.
- A ticket with no acceptance criteria → `no_criteria: "<why>"` instead of the coverage table.

**Loom part** (after watch-loom; when the author says "look here", "now you can see", "notice", re-extract those
seconds with `--timestamps`). For each Loom, write a `loom` entry:
1. **Claims:** each sentence where the author says something works, with its timestamp.
2. **Coverage:** one row per acceptance criterion: demoed (mm:ss or null), implemented (path:line or null).
3. **Evidence:** where it was recorded (local / review app / prod), role, flag state, viewport, happy path only or
   edges too, and `stale: true` when the Loom was recorded before the PR's last commit that touches what it shows
   (compare the Loom's date on its page with `git log`).
4. **Anomalies:** only things that are plainly wrong on screen: an error toast or page, a console error, a broken or
   overlapping layout, a number that contradicts another on screen. Never taste or polish.
5. **Contradictions:** said vs shown vs code, only when one grep or one read of the code settles it.

**Design part** (only when it isn't skipped): open each reference (Figma via its MCP `get_screenshot`, an artifact
with Artifact `read`, an attachment) and compare it with the same state in the Loom frames, or the review app when there
is no Loom. Report **major drift only**: an element missing or added, a different flow or step order, a different
component, wrong copy on a CTA, price or legal text, a layout that changes what the user sees first. Spacing,
pixel sizes, shades and icon details are not drift. Record the references checked and the drift found.

**Noise rules: what may reach the PR:**
- Nothing from this section is posted as it is. A claim not backed by the code, a criterion implemented nowhere, an
  anomaly, a contradiction or a major drift becomes a **candidate** (category `loom` or `design`) and goes through
  Verification like any other: no proof, no finding.
- At most **2** Loom/design candidates per PR, inside the budget of 8, highest severity first. The rest stay on the card.
- "Not demoed" alone is never a comment. It can only feed the heads-up, and only for a risky angle (money, auth,
  other users' data).
- A major drift is at most **medium**, and is posted only when the frame and the reference show the same state side by
  side (name both: `t=01:12` vs the Figma frame).
- Everything else (coverage table, evidence, minor drift) stays in your review notes, never posted.

