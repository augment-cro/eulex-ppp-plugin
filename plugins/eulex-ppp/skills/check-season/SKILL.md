---
name: check-season
description: Check a grower's season spray programme (DE or HR) against the mirrored plant-protection register with the treatment-api MCP. Turns the grower's words into a check_sequence request, runs the question loop, puts every question to the grower and carries the answers back in declared[]. Use when a grower describes treatments (products, dates, doses, water) and asks whether they comply.
---

# Check a season programme

You act for the grower. The service decides; the grower answers; you carry the answers.
**Never answer a question yourself, and never put a fact into a request that the grower
did not give.** When the grower does not know, the answer is `unknown`.

## 0. Before the first check

Call `list_scopes` once. Tell the grower in one line what its `notice` says: this is a test
service whose rules are not signed off, and results are for evaluation, not a basis for a
treatment decision.

## 1. Build the request from what the grower said

`check_sequence` takes the sequence itself plus `detail`. Always send `detail: "summary"`
(the full result of a season is megabytes).

| Field | From the grower's words |
|---|---|
| `country` | `DE` or `HR` |
| `crop` | the EPPO code: resolve the grower's fullest name ("highbush blueberry", the Latin name) with `resolve_term`, `kind: "crop"`; see below |
| `season_start`, `season_end` | `YYYY-MM-DD`; "the 2026 season" is 2026-01-01 to 2026-12-31 |
| `harvest_dates` | the picking dates the grower gives, `YYYY-MM-DD` |
| `harvest_kind` | `actual` for pickings that happened, `planned` for planned ones, else omit |
| `treatments[]` | one object per spray, in date order |

**The crop must resolve to a species code.** If `resolve_term` returns `genus_level` or
`ambiguous` (for example HR "borovnica"), put the candidates to the grower ("highbush
blueberry, VACCO, or bilberry, VACMY?") and send the code the grower picks. A genus-level
crop name matches almost no use, and the check then decides nothing and asks nothing.

Per treatment (required: `applied_at`, `product_auth_no`, `dose`, `dose_unit`):

- `applied_at`: `2026-04-08T09:00:00+02:00`; a date alone is accepted with a warning.
- `product_auth_no`: the authorisation number. If the grower gives only the product name,
  find it with `find_treatments` (see the `look-up-register` skill): `country`, `crop`,
  `detail: "summary"`, `limit` ≤ 10, filtered by `target` or `active_substance` where the
  grower named one, and match `authorisation.product_name`. If two numbers carry the name
  or none matches, ask the grower; do not pick one.
- `target`: the pest, disease or weed the grower names, as a code. Resolve the name with
  `resolve_term` (`kind: "pest"`, the country) and send the resolved `code`. If it comes
  back `ambiguous`, `genus_level` or `unresolved`, try the Latin name, then the code of a
  candidate the grower confirms; if none fits, send the grower's words and tell the grower
  the target may stay open. `check_sequence` does not report an unresolved target: the
  finding stays `indeterminate` with an open `target` and no reason code.
- `product_name`, `dose` (number), `dose_unit` (`l/ha`, `kg/ha`), `water_l_per_ha`,
  `area_ha`, `bbch` (integer 0–99): only when the grower gives them.
- `setting` (`field` for outdoor, `protected` for greenhouse/tunnel), `method`
  (`ground_spray`, `air_assisted`, …), `operator_category` (`professional` or
  `non_professional`): only when the grower says so.
- Leave out any field the grower did not give. A missing field is not an error; the check
  says when it needs one.

## 2. The loop (at most four `check_sequence` calls)

After each call read `data.crop` and `data.warnings[]` first. If `crop.eppo` is `null` or
`crop.basis` is `genus_level`, the crop did not resolve: go back to the crop step and ask
the grower; do not report verdicts from that check. Then read `data.questions[]` and the
`reason_codes` of each finding.

**Questions.** Each has `question_id`, `attribute`, `prompt` (`en`, `de`, `hr`), `allowed`,
`phrase` (the register's own words) and `treatment_index` (0-based position in
`treatments[]`; `null` when the question is about the whole sequence). Put every new
question to the grower in plain words, naming the spray by date and product:

> The Teldor spray of 22 May: *prompt.en*. The label's condition reads: "*phrase*".
> Possible answers: *allowed*.

Several questions with the same prompt and phrase may be put together, but the grower may
answer per spray. Do not show the `resolves` outcomes before the grower answers: an answer
must not be chosen by the verdict it gives.

**Inputs.** A finding with `input_missing` (or `dose_basis_differs`, `water_basis_differs`)
needs a figure only the grower has. Call `explain` with that `finding_id` and
`detail: "summary"`; its `next[]` lists `provide_input` with the field, for example
`treatments[6].in_basis or .leaf_wall_area_m2_per_ha` (leaf-wall area of the stand in m²/ha)
or `treatments[3].bbch`. Ask the grower for that figure. Add it to that treatment only if
the grower gives it.

**Carry the answers back.** Send the same sequence again with `declared[]`, one entry per
answered question:

```json
{"question_id": "q-7d3689772a05e1be", "attribute": "infestation", "value": "moderate", "declared_by": "grower:<name>"}
```

`value` is one of the question's `allowed` values, as the grower chose it; `unknown` when
the grower does not know. Keep every earlier declaration in the list. If you added an input
to a treatment, the sequence changed and every `question_id` changes with it: check again,
and re-declare the grower's earlier answers against the new ids (same attribute, spray and
phrase); do not ask the grower twice.

**Stop** when a check raises no new question and the grower has no further figure to give,
or after the fourth call.

## 3. Report

Use the `read-answers` skill: the verdict per rule group, what is decided, what stays open
and why, and the notice. Name the `check_id`.

**Verdicts on a genus entry get their own line**, a `compliant` one included. A finding
rests on a genus entry when it carries a `crop_basis` (the genus entries it rests on, with
the register's `label` and `label_lat`), or, where the service gives none, when its `use_id`
is a use whose `crop_match.via` is `genus` in the
`find_treatments` results you already have. For a `use_id` you have not seen, call
`find_treatments` once for the crop, filtered by the product's active substance
(`detail: "summary"`, `limit` ≤ 10), and read its `crop_match`; do not call `explain` on
every finding for this. Give that line the four parts of the `read-answers` skill, once per
use.

For each open finding whose `next` is `escalate`, offer escalation as the `read-answers`
skill describes (`pull_escalation`, your judgement, `submit_escalation`), only on the
grower's yes. Report each escalation's outcome with the re-run's `check_id`.

End the report with **Notes**: each genus reading the grower accepted and each settled
genus reading the grower disputed (for the operator's review), with product and `use_id`.
A note does not go into `declared[]` and does not change the check. Do not call `escalate`
or `get_job`.
