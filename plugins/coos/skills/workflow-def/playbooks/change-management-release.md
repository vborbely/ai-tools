# Playbook: Change Management / Release Process

Internal reference only — grounding for `workflow-def`'s best-practice suggestions. Never shown verbatim to the interviewee; use it to recognize which canonical step a described action maps to, and to notice gaps.

## Canonical steps

1. Change proposed (code change, config change, infra change) with a clear description of intent.
2. Risk/impact assessed (blast radius, affected systems, rollback complexity).
3. Peer review / approval before merge (code review or change-advisory sign-off for higher-risk changes).
4. Automated checks run (tests, linting, security scans) before merge.
5. Change scheduled with awareness of blackout windows/high-traffic periods for risky changes.
6. Deployment executed via a repeatable, defined process (not ad hoc manual steps).
7. Post-deploy verification (health checks, smoke tests, key metrics watched).
8. Rollback plan defined and ready before deploying, not improvised after a failure.
9. Stakeholders notified of the change (especially customer-facing or breaking changes).
10. Change recorded in a changelog/audit trail.
11. Post-deploy monitoring window before considering the change fully "done."

## Commonly missed steps

- No rollback plan prepared in advance — one gets improvised under pressure during an incident.
- No risk/impact classification — trivial and high-risk changes go through the identical process (or worse, the identical *lack* of process).
- No post-deploy verification step; "done" means "merged," not "confirmed working."
- No blackout-window awareness for risky changes near high-traffic periods.
- Changes not recorded anywhere centrally, making later root-cause analysis (\"what changed?\") slow.

## Needed documents

- Change request template
- Risk/impact assessment checklist
- Rollback plan template
- Release/deployment runbook
- Change approval record template
- Post-deploy verification checklist
