<!--
Fallback template for workflow-def output files.

This mirrors the frontmatter + body contract used by the Company OS skeleton's
own `templates/concept.md`. It exists only as a safety net: workflow-def always
prefers to read the LIVE `templates/concept.md` from the target repo first (a
customer may have customized their skeleton), and only falls back to this copy
if that file cannot be found. Do not treat this as authoritative over the live
repo's own template.
-->
---
type: playbook                         # concept | playbook | decision | meeting | reference | index
title: <Workflow name, human-readable>
description: <One sentence — what this workflow is and who runs it, for search/retrieval>
classification: public-internal        # public-internal | confidential | restricted-pii (never confidential/PII — see workflow-def's boundaries)
provenance: agent-draft-unverified      # official-policy | sme-verified | agent-draft-unverified | opinion
authority: <owning team or person>      # who was interviewed / who owns this process
verified_by:
verified_on:
review_by: <YYYY-MM-DD>                 # freshness / decay — CI flags overdue
expires: none
supersedes: none                        # path of the workflow doc this replaces, or none
superseded_by: none                     # set when this file is archived
tags: []
resource:                               # optional canonical external URL, if one exists
---

# <Workflow Name>

<One-line summary of the workflow — what triggers it and what it produces.>

## Context

<Why this workflow exists / when it applies. Trigger event. Actors/roles involved.
Tools & systems used.>

## Detail

### Trigger
<What starts this workflow.>

### Actors & roles
<Who is involved and what each role does.>

### Tools & systems
<What software/systems are used to execute this workflow.>

### Step-by-step (as-is)

| # | Actor | Action | Input | Output | Tool | Notes / branches |
|---|-------|--------|-------|--------|------|-------------------|
| 1 |       |        |       |        |      |                   |

### Exceptions & edge cases
<What happens when the normal path doesn't apply.>

## Needed documents

<Documents, forms, checklists, or templates that support this workflow (contracts,
approval forms, runbooks, checklists, ...). One row per document, sourced from what
the interviewee confirmed plus any playbook-suggested documents they accepted.>

| Document | Purpose | Status | Location / owner |
|----------|---------|--------|-------------------|
|          |         | Exists / Recommended |  |

## Recommendations

<De-facto best-practice steps that are NOT currently part of the as-is process,
confirmed by the interviewee as worth adopting. Never merge these into the
as-is steps above until the interviewee confirms they're actually practiced —
this section stays separate until someone decides to adopt and re-document
them as real steps.>

## Open questions

<Anything unresolved at the end of the interview.>

## Related

- [related concept](...)
