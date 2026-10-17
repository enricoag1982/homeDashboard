---
last-reviewed: 2026-10-17
owner: "@enricoag1982"
---

# Principles

The few rules that guide trade-offs when no spec or ADR covers a situation. This is the project's "constitution": short, stable, and referenced from proposals and designs.
Changing a principle is a decision: record it as an ADR.

> [!IMPORTANT]
> Keep this page to about ten principles. If a rule applies only to one area, it belongs in a spec, an ADR or an area `CLAUDE.md`.

## Working principles

These come with the template. Keep, adapt or drop them.

1. **Specify before building.** Behavior changes start as an OpenSpec change that someone other than the author has read. Code follows the spec, not the other way round.
2. **Decisions are written down.** A choice that is costly to reverse gets an ADR before it is relied upon. Undecided questions stay visibly `proposed`; nobody, human or agent, treats them as settled.
3. **One home per fact.** Behavior lives in specs, reasoning in ADRs, current structure in architecture docs, and vocabulary in the glossary. Everything else links.
4. **Small, reversible steps.** Prefer changes that can be reviewed in one sitting and undone cleanly. Split large changes into several OpenSpec changes.
5. **Automate the rules that matter.** If a rule must always hold, make CI check it rather than relying on people or agents to remember it.
6. **Agents are contributors, not authorities.** Agent output goes through the same review, checks and definition of done as human work. Agents propose; humans accept decisions.

## Product and engineering principles

*To be defined.* Candidates are in the [options research](research/2026-10-17-tablet-home-dashboard-options.md#candidate-product-principles); adopting them is a decision for the owner.
