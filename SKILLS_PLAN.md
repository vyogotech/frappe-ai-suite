# Assistant Skills — Plan

Status: **Implemented.** The standalone repository `vyogotech/frappe-live-assist-skill` contains all 9 assistant skills, YAML scenarios, test scripts, and CI workflow. The agent loads skills on demand via the `load_skill` tool and catalog in the system prompt, supports `risk: always-ask` in confirmation cards and policy enforcement, and provides the per-site `AI Skill` DocType in `frappe_ai`.

## Why skills

The agent's system prompt carries every rule. Each new capability (print formats, user
management, client scripts) would make it longer, and a small model follows a short prompt far
better than a long one. A skill is a small markdown document the model loads **on demand**:
the prompt holds a one-line catalog, and a read-only `load_skill` tool returns the body of the one
the task needs.

Format: the open `SKILL.md` convention (YAML frontmatter with `name` and `description`, then a
Markdown body), so any runtime that reads it can use the same files.

## Two bundles, not one

| | Developer skills (`frappe-apps-manager`) | Assistant skills (`frappe-live-assist-skill`) |
| --- | --- | --- |
| Reader | a developer with a shell and a bench | a business user in the Desk |
| Tools | files, bench, git, tests | read, create, update and delete **records**, web search and fetch |
| Delivery step | write a file into an app | create a record with these fields |
| Size | long, with reference folders | short: under about 8 KB each, so a small model keeps the thread |
| Risk rules | none | always-ask, never handle passwords, show full code |

They share topics but not text. Porting is by hand, so each assistant skill is rewritten around
the record tools.

## Licensing and provenance

* 31 of the 90 skills in `frappe-apps-manager` are byte-for-byte copies of the OpenAEC Foundation's
  Frappe skill package, which is **LGPL-3.0**; `frappe-apps-manager` is MIT. Do not bundle those.
  Resolving it in that repo (remove, keep with the LGPL notice, or relicense the folder) is a
  separate decision.
* The assistant bundle is written fresh. Third-party packages are read only to learn which
  topics and pitfalls matter. No text is copied.
* The repo carries a `NOTICE` naming the sources consulted for coverage.
* A skill is a trusted instruction, so it only comes from this repo or from a System Manager's
  site settings, never from a web page or a tool result.

## Repository: `vyogotech/frappe-live-assist-skill` (MIT)

```text
skills/<name>/SKILL.md        one folder per skill, folder name = skill name
scenarios/<name>.yaml         a scripted conversation per skill, invented data only
scripts/validate.py           format, size, link and no-real-data checks
scripts/check_fields.py       every field a skill names exists on a real bench
NOTICE.md  README.md  CONTRIBUTING.md  LICENSE  .github/workflows/ci.yml
```

Each skill names the records and fields it uses in a machine-readable comment, so
`check_fields.py` can verify them against the DocType definitions of a running site.

## First nine skills

| Skill | What it covers | Risk |
| --- | --- | --- |
| `system-health` | Read the System Health Report; scheduler, queues, failed jobs, error log, email queue, disk and backups; report ok / warn / fail with next steps | read-only |
| `user-create` | Create a user with a welcome email so they set their own password; check duplicates; confirm roles exist | always-ask |
| `user-password-reset` | Never accept or set a password. Explain the reset flow; say what the assistant cannot do; what to do if a password was pasted into chat | read-only |
| `user-access` | Enable or disable users, add or remove roles (read the current roles first: the list is replaced, not merged), user permissions; never remove the last System Manager | always-ask |
| `print-format` | Jinja print formats from a real DocType's fields; child-table loops; preview and confirm | writes |
| `letter-head` | Letter heads with HTML source; one default; no invented image addresses | writes |
| `custom-field` | Custom Field and Property Setter: add, hide, make mandatory; duplicate and `insert_after` checks | writes |
| `client-script` | Small form scripts; plain-words explanation first; show the full code; how to disable | always-ask |
| `web-research-records` | Search, read each source page, verify addresses, deduplicate, create one record per reply, summarise in a table | writes |

Later candidates: workspace and dashboard cards, email templates and notifications, workflows,
role profiles, data import cleanup.

## Design rules every skill follows

1. Say which tools it uses and nothing else.
2. Learn the fields from the DocType definition before writing; never guess a fieldname.
3. Child tables are lists of row objects.
4. Check for an existing record before creating one.
5. Addresses and emails come only from pages read this turn.
6. A password never goes through the chat. Chat messages are stored and sent to the model provider.
7. State what the assistant cannot do rather than imply it did.
8. Examples use invented names and reserved domains (`.test`, `example.org`).

## Tests

* **Validator** (CI): frontmatter, name equals folder, description length, body under the size cap,
  no real-looking addresses outside an allow-list, no password fields in examples.
* **Field check** (needs a bench): run against a Frappe SNE container in CI.
* **Scenarios**: each skill ships a YAML conversation giving the prompt, the records expected, and
  what must **not** happen (a guessed field, a duplicate, a password requested). Run through the
  real agent with a hosted model; a skill without a passing scenario does not ship.

## Agent changes needed (separate from the content)

1. `load_skill` read-only tool and a catalog line per skill in the system prompt.
2. Skill sources: the bundled folder, plus a per-site `AI Skill` doctype a System Manager can edit.
   Shipped skills win on name clashes unless explicitly overridden.
3. A `risk: always-ask` flag that makes the sidebar card show **full code or full text**, never a
   one-line summary, and is never covered by auto-approve.
4. A user-visible way to trigger a password reset email is a **new tool**, not a skill; until it
   exists, `user-password-reset` only explains.

## Order of work

1. Ship the suite with web search (separate deployment plan). **First.**
2. Loader and catalog in the agent, with one skill (`print-format`) end to end.
3. Repo scaffold: validator, field check, scenario runner, CI.
4. The other eight skills, one at a time, each with its scenario.
5. Per-site `AI Skill` doctype.
