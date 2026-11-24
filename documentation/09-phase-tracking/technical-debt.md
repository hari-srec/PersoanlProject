# BakeFlow – Technical Debt Register

> **Step:** 1.1 · **Status:** No entries yet

## What this file is for

**Technical debt** is any shortcut we knowingly take to deliver faster, and which we will have to "pay back" later with extra work. Examples: a missing test, a hard-coded setting, a temporary workaround, or a query that is fine now but will be slow at higher volumes.

Debt is not always bad. It is often the right trade-off for a single developer. It becomes dangerous only when it is **invisible**. This register keeps it visible so it can be planned into Phase 4 (advanced engineering) or Phase 5 (hardening), or fixed in a later module.

## When to add an entry

Add an entry whenever:

- a module is marked Done with a known shortcut,
- an NFR target is temporarily not met,
- a decision in [design-decisions.md](../02-system-design/design-decisions.md) is implemented only partly, or
- a code review finds something worth fixing but not urgent.

## Entry format

| ID | Date | Module | Description | Why it was accepted | Impact / risk | Plan to resolve (target phase) | Status |
|----|------|--------|-------------|---------------------|---------------|-------------------------------|--------|

IDs use the pattern `TD-01`, `TD-02`, … and are never reused. Status values: `Open`, `Planned`, `Resolved`.

## Current entries

There are **no entries yet**. Phase 1 produces documentation only, so no code-level debt exists. This register will be filled from **Phase 2** onward.

Known design-level risks and open decisions are tracked elsewhere:

- Open questions and limitations: [scope-and-assumptions.md](../00-project-overview/scope-and-assumptions.md)
- Concerns about accepted decisions: the "Consequences" sections in [design-decisions.md](../02-system-design/design-decisions.md)

## Related documents

- [implementation-status.md](implementation-status.md)
