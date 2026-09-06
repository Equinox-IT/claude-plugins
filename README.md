# Equinox Claude Code plugins

The approved plugin marketplace for Equinox IT. One file matters:
[`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).

Endpoints do not install plugins from the internet. They install from here, and
this repo pins every external plugin to a specific reviewed commit. Nothing an
upstream author pushes reaches an Equinox machine until someone merges a change
to that file.

## What is in it

| Plugin | Source | Pinned | Approved |
|---|---|---|---|
| `diagram-design` | [cathrynlavery/diagram-design](https://github.com/cathrynlavery/diagram-design) | `4451eadc` | [record](docs/approval-record/diagram-design.md) |

## How a plugin gets in

1. Review it against [the approval checklist](docs/plugin-approval-checklist.md).
2. Write the findings to `docs/approval-record/<name>.md`.
3. Open a PR adding the entry, pinned by `sha`. See [adding a plugin](docs/adding-a-plugin.md).
4. Merge, tag, then update the pin in `claude-policy`.

## The one rule

**Every external entry carries a 40-character lowercase `sha`.** A `ref` alone
tracks a moving branch and is not a pin. `claude plugin validate` enforces the
format; nothing but review enforces that the commit was actually read.

## Status

Interim build, 4 September 2026. CI is not wired yet — validation, nightly pin
refresh and the policy-scan gate are the next phase. Until then, run
`claude plugin validate .claude-plugin/marketplace.json --strict` locally before
opening a PR.
