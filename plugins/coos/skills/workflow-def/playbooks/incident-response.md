# Playbook: Incident Response / Postmortem

Internal reference only — grounding for `workflow-def`'s best-practice suggestions. Never shown verbatim to the interviewee; use it to recognize which canonical step a described action maps to, and to notice gaps.

## Canonical steps

1. Detection — alert fires, or a report comes in (monitoring, customer, internal).
2. Triage — severity assessed and assigned (e.g. SEV1-4) against a defined rubric.
3. Incident declared and an incident commander/owner assigned.
4. Communication channel opened (dedicated chat/bridge) for coordinated response.
5. Stakeholder notification (internal first, then customer-facing status page/comms if needed).
6. Mitigation — stop the bleeding (rollback, feature flag, failover) before full root-causing.
7. Root cause investigation once mitigated.
8. Resolution confirmed and incident formally closed.
9. Postmortem written (blameless), including timeline and root cause.
10. Action items logged with owners and due dates.
11. Action items tracked to completion, not just written down.
12. Postmortem shared/reviewed with the wider team for learning.

## Commonly missed steps

- No defined severity rubric — every incident is triaged ad hoc, causing inconsistent response speed.
- No single incident commander — response is diffuse and decisions stall.
- Mitigation and root-causing conflated, delaying the fix while a full diagnosis is pursued.
- Postmortem written but action items never tracked to closure.
- No customer-facing communication step for incidents that affect them.

## Needed documents

- Incident severity/triage rubric
- Incident response runbook (per system/service)
- Postmortem template (blameless format)
- On-call escalation policy / schedule
- Customer-facing communication/status-page template
- Action item tracker
