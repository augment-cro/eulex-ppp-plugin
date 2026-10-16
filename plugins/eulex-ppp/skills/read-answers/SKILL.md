---
name: read-answers
description: Read and explain the answers of the treatment-api MCP - what compliant, non_compliant and indeterminate mean, the reason codes, the notice and rules_not_signed_off, when to call explain, when to stop, and that escalation is switched off on the test service. Use after check_sequence, or when a grower asks why a spray is open or not compliant.
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
| `unknown_attribute`, `typed_missing`, `text_only`, `attribute_not_testable`, `ambiguous_statements` | the register text is not typed well enough for the ruler | operator (escalation); not the grower |
| `genus_level` | the use names the crop only at genus level and the check did not settle it for the grower's crop | operator (escalation); not the grower |

An answer of `unknown` keeps the finding open; that is the correct outcome when the grower
does not know.

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
plain words. Do not repeat a question the grower answered.

## Escalation is switched off on the test service

`escalate` runs a model over the register text behind an open finding, and `get_job`
follows that job. Escalation is switched off on the test service: do not call either tool.
When a finding stays open on a code the table gives to the operator, name its `finding_id`
and reason code so the operator can escalate it. Recommend escalation only for such a
finding of a check; a register lookup has nothing to escalate.
