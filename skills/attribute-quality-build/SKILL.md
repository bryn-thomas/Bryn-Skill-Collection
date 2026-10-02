---
name: attribute-quality-build
description: Build or extend the Attribute Quality Monitoring dbt models for a single Identity Platform (IDP) attribute across the seven data-quality lenses (Data Completeness, Alignment Consistency, Validation Conformity, Sensical Quality, User Verifications, Change Velocity, Risk Impact). Use whenever the user asks to build, investigate, extend or fix quality/lens models for an attribute (e.g. "build dateOfBirth", "do the phone number quality model", "add sense checks for email", "investigate validations for address"), or touches files under attribute_quality_monitoring/.
argument-hint: '<attributeName in camelCase, e.g. dateOfBirth>'
---

# Attribute Quality Build

This skill makes you follow the playbook. **The rules live in the playbook, not here.** The playbook is in
the `airflow-verification` dbt repo (normally `/workspace/airflow-verification`). Read both files in full
before you do anything else:

- `airflow-verification/dbt/models/servicing_platform/intermediate/identities_and_attributes/attribute_quality_monitoring/README.md`
- `airflow-verification/dbt/models/servicing_platform/intermediate/identities_and_attributes/attribute_quality_monitoring/TRACKER.md`

If you can't find the `airflow-verification` repo, stop and ask the user where it is.

If the skill and the README ever disagree, the README wins. Tell the user about the conflict.

## How to run it

0. **Check the attribute inventory.** If the tracker's **Last inventory refresh** is more than 30 days old
   or has never run, run the README's "Attribute inventory refresh" before anything else. Use the dbt MCP,
   with aggregates only. Every attribute is marked (⏸ excluded / 🆕 unbanded / 🗄 retired), never removed.
   Tell the user about any changes. If the requested attribute is ⏸ excluded, say so and ask before
   building it.
1. **Find the attribute** from the argument or the request. Look it up in `TRACKER.md` to get its priority
   band and current status. If it's already partly built, start from the first phase whose exit condition
   isn't met. Never assume a 🟡 item is correct.
2. **Work through the README's "Build process: phases and gates" table in order.** At the start of each
   phase, say which phase you're in. Before moving to the next, state the exit condition and the evidence
   that it's met, e.g. "Phase 1 done: SHA `abc123` recorded in TRACKER.md".
3. **⛔ Gates are hard stops.** At phase 3 (findings report) and phase 7 (verify), present the required
   output and **end your turn**. Don't write SQL or yml before phase 3 is approved. Only an explicit
   approval of the report counts.
4. **Update the tracker as you go**, following the README's "Keep `TRACKER.md` current" table. Don't leave
   tracker updates until the end.
5. **Sync the Confluence Progress tracker** after phase 3 approval and at phase 8. Follow the README's "Sync
   to Confluence":
   - fetch the latest page
   - change **only** the Progress tracker section. Nothing else on the document may change: no wording,
     formatting, ordering or "tidy-ups" elsewhere, even if something looks wrong (tell the user instead).
     Diff the body outside the section before and after; it must be identical.
   - **present the draft and wait for approval before publishing**
   - re-fetch to confirm it rendered
   `TRACKER.md` is the source of truth. Never edit Confluence without updating the tracker too.

## Non-negotiables (quick reference; full detail in the README)

- Expectation and validation rules come **only from service code**, from a fresh IDP clone with the SHA
  recorded. They never come from dbt models or `/workspace/Investigations/...`, which you may use as a map
  but must re-verify.
- If the clone fails, **stop and ask**. Don't fall back to anything else.
- Our models are never the origin of a value. Always name the origin service, its table and the code that
  writes it.
- Sense checks: the README's examples are a starting point. Propose a fuller set for each attribute.
- SQL:
  - import CTEs are `select *` with light filters
  - logic lives in the middle CTEs
  - the `final` CTE only casts types
  - incremental logic follows README §4.1
- No validation rules → `is_valid = null`. A null lens never counts as a pass.
- Change velocity counts every event except `NO_CHANGE`, including `CREATE`. No actor split and no
  pass/fail flag.
- ymls: every column documented, Tier 2 tests **applied** (confirm with the `pipeline-tier-system`
  skill), tests scoped to `af_ts`.
- Query data with the dbt MCP, never the Lightdash MCP, which blocks `_clear` schemas.
- Don't commit, push or open a PR unless the user asks.

## Finishing

You're done only when every item in README §5 is ticked **and** `TRACKER.md` reflects the work. End with a
short summary covering:
- the lenses built (✅ / 🟡 / ➖)
- the IDP SHA
- any open questions left in the tracker
