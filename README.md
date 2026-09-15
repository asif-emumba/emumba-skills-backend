# emumba-skills-backend

Backend engineering skills for Emumba, distributed to developers through the
LiteLLM gateway catalogue as the plugin `emumba-backend`.

> **Status: draft.** Every skill here is marked *pending backend-guild
> sign-off*. Nothing in this repo is ratified Emumba policy yet.

## What is in here

| Skill | Scope |
|---|---|
| `rest-api-conventions` | Stack-agnostic HTTP conventions — naming, status codes, error envelope, pagination, versioning |
| `spring-boot-service` | Java / Kotlin — layering, transactions, persistence, testing |
| `node-express-service` | Node / TypeScript — async error handling, validation, config, shutdown |

One plugin, many stacks: a backend developer installs once and gets the whole
set, rather than a separate install per framework.

## How a developer gets these

They do not clone this repo by hand. The gateway publishes a catalogue and
Claude Code clones from it:

```bash
claude plugin marketplace add http://localhost:4000/claude-code/marketplace.json
claude plugin install emumba-backend@litellm
```

The catalogue row pointing here is:

```json
{
  "name": "emumba-backend",
  "source": { "source": "url", "url": "https://github.com/asif-emumba/emumba-skills-backend.git" }
}
```

## Layout rules that actually bite

**Skills must be flat.** Claude Code discovers only
`<repo-root>/skills/<name>/SKILL.md`. Grouping them by category —
`skills/api/rest-api-conventions/SKILL.md` — installs cleanly and loads
**zero** skills, with no error to tell you why.

**Keep `.claude-plugin/plugin.json`.** Without it, skills are namespaced by the
version directory (`0.1.0:rest-api-conventions`) instead of the plugin name
(`emumba-backend:rest-api-conventions`).

**Use the `url` source form, not `github`.** A source of
`{"source": "github", "repo": "org/repo"}` makes Claude Code clone over SSH,
which fails on any machine without a github.com host key. The `url` form clones
over HTTPS and works everywhere.

## Releasing a change

The version must agree in **three** places or `claude plugin update` will
mislead you — it reports "already at the latest version" while the dashboard
implies an update exists:

1. `.claude-plugin/plugin.json` in this repo
2. the `version` on the gateway catalogue row
3. whatever each developer has installed

So a release is:

1. Merge the skill change here and bump `plugin.json`
2. Update the catalogue row to the same version — `PUT /claude-code/plugins/emumba-backend`, which requires the **full** entry, `source` included; a partial body with only `version` returns 422
3. Developers run `claude plugin update emumba-backend`

## Ownership

`.github/CODEOWNERS` assigns this repo to the backend guild. That line is only
as strong as the repo's write permissions and branch protection — set both.

Note that repo ownership protects the *content*, not the *catalogue row*. The
gateway records `created_by` on a plugin but never enforces it, so anyone who
can reach `/claude-code/plugins` can repoint or delete this plugin's row. Keep
that endpoint to a small admin group.
