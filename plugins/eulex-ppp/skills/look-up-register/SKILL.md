---
name: look-up-register
description: Look up registered plant-protection uses in the DE (BVL) or HR (FIS) register mirrored by the treatment-api MCP - which products are registered on a crop, against a pest, with which dose, water, interval, number of applications and pre-harvest interval, and the authorisation number of a product. Uses list_scopes, resolve_term, find_treatments and get_use.
---

# Look up the register

## 0. First

Call `list_scopes` once per session: per country it gives the snapshot and data date, the
fields with their fill rates, the rules it can decide and the limits, under the `notice`
(test service; rules not signed off). Say the data date when you quote a use.

## 1. Names to codes: `resolve_term`

```json
{"term": "Vaccinium corymbosum", "kind": "crop", "country": "DE"}
```

`kind` is `crop`, `pest` or `substance`; `lang` (`hr`, `de`, `en`) is optional. The result
gives candidates, the resolved code, or why not (`ambiguous`, `unresolved`, `genus_level`).
`find_treatments` and `check_sequence` resolve crop names the same way. For a target, resolve
first and send the code: `check_sequence` does not report a target name it could not resolve.

## 2. Uses on a crop: `find_treatments`

```json
{"country": "DE", "crop": "VACCO", "target": "Botrytis cinerea", "detail": "summary", "limit": 10}
```

- Always `detail: "summary"` and a small `limit` (≤ 10), then page with `offset`.
  `data.total` says how many there are. A summary use is 1–6 kB (the tool description says
  about 1 kB); twenty can pass the client's tool-result limit.
- Filters: `target` (code, Latin or common name), `active_substance` and `holder`
  (case-insensitive substrings), `status` (`authorised` by default; `grace`, `withdrawn`,
  `unknown`, `any`).
- `via_groups`: `fruit` (default) also counts uses on a fruit crop group that covers the
  crop; `none` gives only uses that name the crop directly.
- There is no product-name filter. To find a product's authorisation number, filter by the
  target or active substance the grower named and match `authorisation.product_name`;
  `authorisation.number` is what `check_sequence` needs as `product_auth_no`.

Each use carries `use_id`, `crop_match` (direct or through which group), `authorisation`
(number, product name, holder, status, valid until), `targets`, `dose`, `water`, `phi`,
`bbch`, `max_applications`, `interval_days` and `restrictions`. A limit with `how: "text"`
or `"null"` was not typed: quote it, do not read a number into it.

## 3. One use in full: `get_use`

```json
{"use_id": "de:00B035-00/01-003"}
```

The whole record with its evidence (snapshot hash, JSON paths) and the source URL. Quote
values with the `use_id` and the URL. Use it for one use at a time.

## Rules

- Report what the register states, with the use id. Never fill a gap from general
  knowledge.
- This answers "what is registered". Whether a grower's sprays comply is the
  `check-season` skill.
