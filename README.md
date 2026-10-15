# eulex-ppp — Claude Code plugin for the Treatment API test service

This repository is a Claude Code plugin marketplace with one plugin, `eulex-ppp` (EULEX plant
protection products). The plugin connects Claude Code to the Treatment API test service (DE
and HR registers of plant protection products) through its MCP endpoint, and gives the agent
three skills:

| Skill | Does |
|---|---|
| `eulex-ppp:check-season` | turns a grower's spray programme (products, dates, doses, water, crop, pickings) into a check against the register, puts every question the check raises to you, and sends back only the answers you give |
| `eulex-ppp:look-up-register` | which products are registered on a crop, against a pest, with which dose, interval, number of applications and pre-harvest interval |
| `eulex-ppp:read-answers` | what `compliant`, `non_compliant` and `indeterminate` mean, why a finding stays open, when to stop |

**What it is not.** The service is a test service: its results are for evaluation and are
not a basis for a treatment decision. Use rules are judged against the register as mirrored
(emergency authorisations not covered), and the rules are not signed off. The agent never
answers a question for you; a question you cannot answer stays open (`indeterminate`).

## Install (two steps)

You need Claude Code and a tester key from Augment. The key goes into your environment,
never into a file of this repository.

1. In the shell you start Claude Code from:

   ```bash
   export TREATMENT_API_KEY=<your key>
   ```

2. In Claude Code:

   ```
   /plugin marketplace add augment-cro/eulex-ppp-plugin
   /plugin install eulex-ppp@eulex-ppp
   ```

   From a shell instead: `claude plugin marketplace add augment-cro/eulex-ppp-plugin`,
   then `claude plugin install eulex-ppp@eulex-ppp`.

Then start a new session (or run `/reload-plugins`). `/mcp` lists
`plugin:eulex-ppp:treatment-api` as connected. Describe your season in your own words, or
start with `/eulex-ppp:check-season`.

## Another host

The plugin talks to `https://treatment-api-5s5qu3wura-ez.a.run.app/mcp` unless
`TREATMENT_API_URL` names another MCP URL, for example a local container:

```bash
export TREATMENT_API_URL=http://127.0.0.1:8000/mcp
```

Set it before you start Claude Code, like the key.

## Troubleshooting

- `/mcp` shows the server failing, or every call answers 401: the key is missing from the
  environment Claude Code was started in, or it was revoked. Export it and start a new session.
- Updates: third-party marketplaces do not update on their own. Run
  `/plugin marketplace update eulex-ppp`, then `claude plugin update eulex-ppp@eulex-ppp`
  and start a new session.
- Remove: `claude plugin marketplace remove eulex-ppp` removes the marketplace and its plugin.

The plugin in `plugins/eulex-ppp/` is a copy; it is maintained in Augment's project
repository and synced here.
