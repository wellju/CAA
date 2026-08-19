# sap-change-advisor

A GitHub Copilot plugin that turns a raw SAP change request into a review-ready
advisory: normalized requirement, open questions, risk matrix, compliance
checks, test cases and a handover package.

Works in the GitHub Copilot app, in Copilot CLI and in VS Code.

Ported from the PowerShell `CAA/` in [valuept/Agent1](https://github.com/valuept/Agent1),
which carried these seven prompts but never loaded them: its runtime rendered
fixed text and ignored its own `skills/`, `memory/` and `config.yaml` entirely.

---

# User manual

Written for people who have not worked with an AI agent before. If you already
use Copilot CLI daily, skip to [Quick reference](#quick-reference).

## 1. What this is, and what it is not

**It is** a first-pass analyst. It reads a change request and produces the
document a consultant would bring to a change advisory board: what is actually
being asked, what nobody has answered yet, what could go wrong, what has to be
tested, and what still needs a decision.

**It is not** an approver, and it is not a source of SAP truth. It works from
the text you give it plus the organization context you configure. It does not
read your SAP system, your transport requests, or your customizing tables.

The most valuable thing it produces is usually the list of questions, not the
list of answers. A request that survives its questions is a request worth
approving.

## 2. What you need

| | |
|---|---|
| A GitHub Copilot subscription | Copilot Free works; usage counts against your plan |
| GitHub Copilot CLI | `npm install -g @github/copilot`, needed to install the plugin |
| The GitHub Copilot app | Optional. If you use it, the agent shows up there too |

This is a **plugin**. It packages one custom agent plus seven skills. Plugins
load in the GitHub Copilot app, in Copilot CLI and in VS Code, so once it is
installed you can use it wherever you already work.

## 3. Setup, once

**Log in.** Start the CLI and authenticate:

```bash
copilot
```

Inside it, type `/login` and follow the browser flow. Without this every command
fails with `Error: No authentication information found.` If you are already
signed in to the Copilot app, the CLI still needs its own login.

**Install the plugin:**

```bash
git clone https://github.com/wellju/CAA.git
cd CAA
copilot plugin install ./
```

Installing through the CLI is enough for both surfaces. It writes to
`~/.copilot/installed-plugins/`, which the Copilot app reads as well.

**Check that it worked:**

```bash
copilot skill list
```

You should see seven skills listed under `Plugin skills`, from
`compliance-checker` through `testcase-designer`. If any are missing, see
[Troubleshooting](#10-troubleshooting).

## 4. Your first advisory

**In the Copilot app:** type `/agent` in the prompt box and pick
`change-advisor`, or select it from the agent picker dropdown.

**In the terminal:**

```bash
copilot --agent change-advisor
```

Either way you are now in a conversation. Describe your change in plain
language. You do not need a special format, and you do not need to know any
prompt tricks. This is enough:

> We want to move payroll for the Austrian entity from the current
> on-premise system to S/4HANA. Around 4000 employees. Finance and HR are
> involved. Target is end of Q4. We know the custom ABAP reports are a
> problem but nobody has looked at them yet.

If your request already exists as a file, point at it instead:

> Read change-request.json and give me an advisory.

The more concrete you are about systems, dates, effort and known risks, the
less the agent has to mark as unknown. Vagueness in produces questions out,
which is the correct behaviour, not a failure.

## 5. What happens next

The agent works through six phases in order, loading a specialist skill for
each one:

| Phase | Skill | What it produces |
|---|---|---|
| 1 | `requirement-normalizer` | What is actually being asked, in scope and out of scope |
| 2 | `gap-question-generator` | The questions stakeholders must answer first |
| 3 | `impact-analyzer` | Risk matrix, affected systems, user impact |
| 4 | `compliance-checker` | Governance, security and data-protection checks |
| 5 | `testcase-designer` | How the change gets proven correct |
| 6 | `handover-writer` | The final advisory document |

If your request is too thin to work with, it stops after phase 2 and hands you
the questions. That is deliberate. A question list is a useful answer; an
analysis built on guesses is not.

## 6. Reading the result

Three things in the output deserve your attention:

**The recommendation** is `GO`, `CONDITIONAL` or `NO-GO`. `CONDITIONAL` is the
common one and the condition is always stated. Read the condition, not just the
verdict.

**`[HUMAN DECISION REQUIRED]`** marks a fork the agent deliberately refused to
decide for you. It lists the options and their consequences. These are the
places where the advisory needs a person.

**Known unknowns** are gaps the agent found and did not fill. Treat this as the
most honest section of the document. A short unknowns list on a vague request
means the agent invented things, and you should push back.

## 7. Steering it

You are in a conversation, so you can keep talking:

| You want | Say something like |
|---|---|
| More pressure on a weak business case | "Grill me on this request." |
| A deeper look at one area | "Expand the risk analysis around the ABAP custom code." |
| It to correct itself | "The target date is Q1, not Q4. Redo the timeline impact." |
| The result as a file | "Write the advisory to CHG-2026-001-advisory.md." |

The agent writes files only when you ask. It will request your permission
before touching anything on disk; approve or decline when prompted.

## 8. Rules you must follow

**No personal data.** Change requests often carry names, personnel numbers or
org units. Do not paste them. The agent is instructed to work in roles and to
stop if a request is built around individual employee data, but that is an
instruction to a model, not a technical barrier. Strip the data before you
paste it.

**Verify every SAP specific.** The agent is instructed never to invent table
names, function modules, BAdIs, transactions or infotypes. Instructions reduce
that risk; they do not remove it. Anything concrete enough to act on is
concrete enough to check.

**It does not know your landscape** until you tell it. See the next section.

## 9. Making it yours

Out of the box the advice is generic SAP advice. Open
[`agents/change-advisor.agent.md`](agents/change-advisor.agent.md) and fill in
the **Organization context** block: your systems and releases, your payroll
scope, your development model, your change-classification thresholds, your
named approvers, the frameworks you are bound by.

This is the single highest-value edit in the whole plugin. Everything else is
carried over from the original prompts; this block is what makes the output
about your organization instead of about SAP in general.

After editing, reinstall so the change takes effect:

```bash
copilot plugin install ./
```

The installer copies files, it does not link them. Edits do nothing until you
reinstall, in the app as well as in the terminal.

## 10. Troubleshooting

| Symptom | Cause and fix |
|---|---|
| `Failed to install plugin: Invalid plugin spec` | You wrote `install .` instead of `install ./`. The trailing slash is what marks the argument as a path. |
| `Error: No authentication information found.` | Not logged in. Run `copilot`, then `/login`. |
| A skill is missing from `copilot skill list` | Its `SKILL.md` frontmatter failed to parse. A colon in an unquoted `description` is the usual cause; wrap the value in double quotes. |
| Your edits have no effect | You did not reinstall. Run `copilot plugin install ./` again. |
| `Warning: Direct plugin installs are deprecated` | Expected. Local installs still work; GitHub plans to require marketplace installs eventually. |
| Skills missing when mounted with `--plugin-dir` | Observed but unexplained. Use `copilot plugin install ./` instead. |
| The agent does not appear in the Copilot app | Restart the app. It reads the plugin directory at startup. |

## Quick reference

```bash
copilot plugin install ./         # install or refresh after editing
copilot skill list                # verify the seven skills loaded
copilot --agent change-advisor    # start the agent in the terminal
```

In the Copilot app: `/agent`, then `change-advisor`.

## Layout

| | |
|---|---|
| `plugin.json` | manifest |
| `agents/change-advisor.agent.md` | orchestrator: phase order, rules, organization context |
| `skills/*/SKILL.md` | the seven prompts, carried over near-verbatim |

## Known limits

- No PII masking. Section 8 is a rule for you to follow, not a guard that
  enforces it.
- The agent has no connection to any SAP system. It reasons about the text it
  is given.
- Risk scores are arguments, not measurements. The agent is instructed to
  justify each one; uniform scores across all risks mean the analysis was thin.
