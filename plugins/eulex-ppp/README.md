# eulex-ppp — a Claude Code plugin for the Treatment API test service

EULEX plant protection products: a grower's agent for the Treatment API MCP. It connects
Claude Code to the service and gives it three skills:

| Skill | Does |
|---|---|
| `eulex-ppp:check-season` | turns a grower's spray programme into a `check_sequence` request, puts every question to the grower and carries the answers back in `declared[]` (at most four checks) |
| `eulex-ppp:look-up-register` | `list_scopes`, `resolve_term`, `find_treatments`, `get_use` on the DE and HR registers as mirrored |
| `eulex-ppp:read-answers` | what `compliant`, `non_compliant`, `indeterminate` and the reason codes mean, the notice, when to call `explain`, when to stop |

**What it is not.** The service is a test service whose rules are not signed off
(`rules_not_signed_off`): its results are for evaluation, not a basis for a treatment
decision. Use rules are judged against the mirrored register only. The agent never answers
a grower question itself; an unanswered question stays `indeterminate`. Escalation is
switched off on the test service: the agent names the open findings the operator can
escalate.

## Install

You need Claude Code and a tester key from the operator.

1. Put the key in your environment (never in a file under this directory):

   ```bash
   export TREATMENT_API_KEY=<your key>
   ```

   The service is expected at `https://treatment-api-5s5qu3wura-ez.a.run.app/mcp`. For
   another host, such as a local container, set `TREATMENT_API_URL` to its `/mcp` URL.

2. Install it from the marketplace:

   ```
   /plugin marketplace add augment-cro/eulex-ppp-plugin
   /plugin install eulex-ppp@eulex-ppp
   ```

   Or load this directory for one session:

   ```bash
   claude --plugin-dir <path to this directory>
   ```

3. Check that it loaded: `/mcp` lists `plugin:eulex-ppp:treatment-api` as connected with
   ten tools, and `/eulex-ppp:check-season` is offered. A 401 means the key is missing
   or revoked.

## Files

- `.claude-plugin/plugin.json`: the manifest.
- `.mcp.json`: one HTTP (streamable) server, `treatment-api`, with the header
  `X-API-Key: ${TREATMENT_API_KEY}`; the key is read from the environment when Claude Code
  starts. Its tools appear as `mcp__plugin_eulex-ppp_treatment-api__<tool>`.
- `skills/*/SKILL.md`: the three skills.
