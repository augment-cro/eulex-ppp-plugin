---
name: read-answers
description: Read and explain the answers of the treatment-api MCP - what compliant, non_compliant and indeterminate mean, the reason codes, the notice and rules_not_signed_off, uses that rest on a genus entry (caveat, discrepancy, recommendation, accept or escalate), when to call explain, when to stop, and how to escalate an open finding with pull_escalation and submit_escalation. Use after check_sequence, or when a grower asks why a spray is open or not compliant.
---

# Read the answers

## The notice comes first

Every answer carries a `notice`. Repeat its sense to the grower once per conversation:

- **Test service.** This is a test service whose rules are not signed off
  (`rules_not_signed_off`). Results are for evaluation, not a basis for a treatment decision.
- **Mirrored register only.** Use rules are judged against the register as mirrored;
  emergency (Art. 53) authorisations are not covered.
- **2026 treatments are judged against the current snapshot.** A status or use change
  earlier in 2026 is not seen.

Say it once, then answer. A value the service returns is the service's answer: do not
narrate how it was produced (no review status, no model or model name, no `how: "model"`,
`strong`, `note: "row not reviewed"` or vocabulary-pass trace), and do not repeat
`rules_not_signed_off` per finding.

## Verdicts

- `compliant`: the register as mirrored, with what the grower stated, settles the rule
  in favour.
- `non_compliant`: it settles the rule against. Say which value breaks which limit
  (`expected`, `actual`), with the `use_id`.
- `indeterminate`: not settled either way. **This is not a pass**, and not a breach.

**Report only the verdicts the check gives.** Do not turn a register lookup into a verdict:
a product that `find_treatments` did not list under a crop name is not "unregistered", and
an `indeterminate` finding is not `non_compliant`. If you think the check missed something,
say so as a question for the operator, apart from the verdicts.

`overall` and `groups` (`registry`, `use`, `records`) fold the findings. The rules:
`R-AUTH-DATE` (authorised on the date), `R-USE` (a registered use admits the spray: crop,
target, dose, water, stage, PHI, conditions), `R-COUNT-MAX` (number of applications),
`R-INTERVAL-MIN` (days between applications), `R-REC-COMPLETE` (record fields).

## Reason codes of an `indeterminate` finding

Who can move it:

| Code | Meaning | Who acts |
|---|---|---|
| `depends_on` | a grower question is open (a `questions[]` entry) | grower answers |
| `input_missing`, `where_relevant_missing` | a field of the spray is missing (`bbch`, leaf-wall area, …) | grower gives the figure |
| `dose_basis_differs`, `water_basis_differs` | the label states the rate in another basis (leaf-wall area, m³, …) | grower gives the figure in that basis or the stand's leaf-wall area |
| `use_rule_undecided` | count or interval waits on an open `R-USE` finding | follows that finding |
| `no_common_use` | no single use admits every spray of the product, so count and interval cannot be judged | nobody; say so |
| `no_statement_applies` | no limit statement of the use applies to the spray as described | read with `explain` |
| `unknown_attribute`, `typed_missing`, `text_only`, `attribute_not_testable`, `ambiguous_statements` | the register text is not typed well enough for the ruler | you can pull it, with the grower's consent (see Escalation); not the grower |
| `genus_level` | the use names the crop only at genus level and the check did not settle it for the grower's crop | give the four parts below; you can pull it, with the grower's consent (see Escalation) |

An answer of `unknown` keeps the finding open; that is the correct outcome when the grower
does not know.

## A use that rests on a genus entry

Some register entries name a genus, not a species (HR "Borovnica", *Vaccinium* sp.). The
service counts such a use for the species the grower asked about. A lookup or a verdict
that rests on it must not read like a use registered on the species by name. Whenever an
answer rests on a genus entry, a `compliant` verdict included, give four parts:

1. **Caveat.** This use rests on a genus entry of the register.
2. **The discrepancy, in the register's own words.** The entry's label and Latin name
   ("Borovnica", "*Vaccinium* sp."), and, where the register names the species itself on
   other products, that name and those products (for highbush blueberry, "Američka
   borovnica" on REVYONA and SWITCH 62,5 WG: uses whose `crop_match.via` is `direct`).
   Quote the register; add no meaning of your own to its names.
3. **Recommendation.** Treat it as registered on the species: the Latin name defines the
   entry's scope, and a genus covers its species.
4. **The choice.** The grower can accept that reading or escalate it.
   - *Accept*: record it as a note in your report ("Accepted: <product>, `<use_id>`, read
     as registered on <species> through the genus entry"). It is not a grower answer: it
     does not go into `declared[]` and does not change the check.
   - *Escalate*: for a check finding with reason `genus_level`, escalate it as in
     Escalation below. A genus reading the service has already settled (a lookup's
     `crop_match.via: "genus"`, or a check finding with a `crop_basis`, `compliant`
     included) leaves nothing to pull: record the grower's dispute as a note for the
     operator, who reviews the vocabulary ("Disputed: <product>, `<use_id>`, genus entry
     <label> (<label_lat>); for the operator's review"). Never run a check only to obtain
     a finding to escalate.

How to tell that an answer rests on a genus entry:

- a use from `find_treatments` with `crop_match.via: "genus"`;
- a `get_use` crop with `eppo_basis: "genus_level"` and a `genus`. Its `eppo` code, if one
  is given, is not the crop the use covers; read the coverage of the grower's crop from the
  `crop_match` that `find_treatments` gives for that `use_id`;
- a check finding with `crop_basis`: a list of the genus entries it rests on, each with
  `basis: "genus"`, the register's `label` and `label_lat`, the `genus`, the `crop` and the
  `use_ids`. Where a service gives no `crop_basis`, a finding whose `use_id` is a genus use
  by the two lines above (see `check-season` §3);
- a finding with reason code `genus_level`: the check left the crop open on a genus entry.
  Give the four parts; the recommendation is the same.

Give the four parts once per use in an answer, not per finding of the same use. Do not call
the use "unsettled" and do not leave it "for the operator to resolve".

## When to call `explain`

`explain` with `finding_id` and `detail: "summary"` (under 8 kB) gives `summary_text`, the
candidate uses (`uses_table`: which use, which parts open) and `next[]` (the actions:
`provide_input`, `answer_question`, `escalate`, `reingest`, `signoff`). Call it when:

- a finding is open and no question names it (to find the input the grower could give);
- the grower asks why a spray is `non_compliant` or open.

Give `use_id` as well for one candidate use in full. Do not fetch `detail: "full"` unless
the summary is not enough.

## When to stop

Stop asking when a check raises no new question and the only `next` actions left are
`escalate`, `reingest`, `signoff` or none. Report what stays open and why, by code, in
plain words, and offer escalation for the findings whose `next` is `escalate`. Do not
repeat a question the grower answered.

## Escalation: you judge, the service gates

An `indeterminate` finding of a check with an escalatable reason (the codes the table above
says you can pull) can be escalated: a judge reads the register rows and the law behind it
and proposes what they state. On this service the judge is you, on the grower's model. Do
not call `escalate` or `get_job`: they start the service's own judge, which does not run on
the test service.

1. **Ask first.** Name the finding (spray, rule, reason code) and say that the judge runs
   on the grower's own model and subscription, at no cost to the service. Escalate only on
   the grower's yes. One pull per finding per conversation.
2. **Pull.** `pull_escalation` with the `finding_id` returns `job_id`, `system` (the judge's
   system prompt), `user_message` (the dossier: the treatment, the grower's inputs and
   answers, the reasons, the register rows, the law pages), `output_schema`,
   `prompt_sha256` and `max_tokens`. `invalid_input` means the finding is not escalatable;
   say so and stop.
3. **Judge.** Follow `system` literally as your instructions for this one task and read
   `user_message` as its input. Add no fact the grower did not give and no knowledge of
   your own. Answer only in `output_schema`, with the job's `finding_id`.
4. **Submit.** `submit_escalation` with `job_id`, your answer as `assessment`, unedited,
   and `judge`: your model id as you know it. `invalid_assessment` means the answer was not
   in the schema or named another finding; the job stays pulled, so fix the form and send
   it again without changing the reading.
5. **Report** the result as returned: `settled` (true or false), the `gate` outcome and its
   reason (for example `quote_not_found`), the re-evaluated `finding` and its `effect`
   (`verdict_changed`, `leaves_changed`, `none`), and the re-run's new `check_id`. Say
   "judged by <model> on your key, as reported by the agent". A result that asks the
   grower (`ask`, reason `ask_grower`) settles nothing: put the question to the grower.
   When it settled, say that it now settles this finding for every tester of the service
   until the operator reviews it.

After a gate failure, do not judge again and do not submit another reading: the finding
stays open, and you report why. An answer must never be chosen by the verdict it gives.
