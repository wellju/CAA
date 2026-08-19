# sap-change-advisor

A GitHub Copilot CLI plugin. Turns a raw SAP change request into a review-ready
advisory: normalized requirement, gap questions, risk matrix, compliance checks,
test cases and handover package.

Ported from the PowerShell `CAA/` in [valuept/Agent1](https://github.com/valuept/Agent1),
which carried these seven prompts but never loaded them: its runtime rendered
fixed text and ignored its own `skills/`, `memory/` and `config.yaml` entirely.

## Install

```bash
git clone https://github.com/wellju/CAA.git
cd CAA
copilot plugin install .
copilot skill list
```

Re-run `plugin install .` after every edit; it copies into
`~/.copilot/installed-plugins/`, it does not symlink.

## Use

```bash
copilot --agent change-advisor
```

Then hand it a change request, as a file path or pasted text. The agent runs six
phases in order and pulls each skill in as it reaches that phase. Ask it to
"grill" a request to add the seventh.

## Before first real use

Fill the **Organization context** block in
[`agents/change-advisor.agent.md`](agents/change-advisor.agent.md). It ships with
placeholders and generic governance thresholds. Until it holds your landscape,
your approvers and your frameworks, the advice is generic SAP advice.

The old `memory/` files were dropped rather than ported: five files of textbook
SAP knowledge a model already has. Only the org-specific part survived, and it
lives in that one block.

## Layout

| | |
|---|---|
| `plugin.json` | manifest |
| `agents/change-advisor.agent.md` | orchestrator, phase order, rules, org context |
| `skills/*/SKILL.md` | the seven prompts, carried over near-verbatim |

## Known

- `copilot plugin install` warns that direct local installs are deprecated in
  favour of marketplace installs. Still works today.
- `--plugin-dir .` mounts the plugin, but its skills did not
  appear in `copilot skill list`. Unconfirmed whether they load at session time.
  Use `plugin install` until that is settled.
- No PII masking. Change requests carrying personnel numbers or names are the
  agent's problem to refuse, not a guard's to strip. The agent prompt instructs
  it to work in roles and stop on individual employee data; that is an
  instruction, not an enforcement.
