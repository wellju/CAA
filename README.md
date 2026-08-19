# SAP Change Advisor

A GitHub Copilot plugin. Give it a change request, get back the analysis a
consultant would bring to a change advisory board: what is actually being
asked, what nobody has answered yet, what could go wrong, what has to be
tested, and what still needs a human decision.

## Install

In the GitHub Copilot app: **Settings → Plugins**, add from repository, enter
`wellju/CAA`.

## Use

Type `/agent` in the prompt box and pick **change-advisor**. Then describe your
change in plain language. No special format, no prompt tricks:

> We want to move payroll for the Austrian entity from the current on-premise
> system to S/4HANA. Around 4000 employees. Finance and HR are involved.
> Target is end of Q4. We know the custom ABAP reports are a problem but
> nobody has looked at them yet.

It works through six phases: normalize the requirement, generate the open
questions, analyse impact and risk, check compliance, design the test cases,
package the advisory. Keep talking to it afterwards to correct facts, go deeper
on one area, or have it write the result to a file.

Read three things closely: the **GO / CONDITIONAL / NO-GO** recommendation and
its condition, every **`[HUMAN DECISION REQUIRED]`** marker, and the **known
unknowns**. A thin unknowns list on a vague request means it filled gaps with
guesses, and you should push back.

## Two rules

**No personal data.** Names, personnel numbers, org units: strip them before
you paste. The agent is told to work in roles, but that is an instruction to a
model, not a barrier.

**Verify every SAP specific.** It is told never to invent table names, function
modules, BAdIs, transactions or infotypes. That reduces the risk; it does not
remove it.

## Make it yours

Fill the **Organization context** block in
[`agents/change-advisor.agent.md`](agents/change-advisor.agent.md) with your
landscape, thresholds and approvers, then reinstall. Until you do, the advice
is generic SAP advice.
