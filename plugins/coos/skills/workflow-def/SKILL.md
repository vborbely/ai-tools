---
name: workflow-def
description: Interactive "grill me"-style interviewer that documents one company workflow in depth. Asks targeted questions to break a colleague's process into atomic steps, cross-checks against de-facto best-practice playbooks to surface missing steps as flagged recommendations, and writes the result into a Gitea-hosted Company OS knowledge base repo as a branch + Pull Request (never directly to main). Requires a connected Gitea MCP server; if a coos-mcp MCP server is also connected, uses it to search existing docs instead of full-text tree reads, for a faster duplicate check. Use when the user says "document this workflow", "grill me about our process", "build our company OS", "capture how X team does Y", "interview me about a process", or wants to turn a colleague's tacit process knowledge into a Company OS playbook.
allowed-tools:
  - Read
  - mcp__gitea__get_file_contents
  - mcp__gitea__get_repository_tree
  - mcp__gitea__list_my_repos
  - mcp__gitea__list_org_repos
  - mcp__gitea__create_branch
  - mcp__gitea__create_or_update_file
  - mcp__gitea__delete_file
  - mcp__gitea__pull_request_write
  - mcp__gitea__pull_request_read
  - mcp__coos-mcp__search_knowledge
---

# Workflow-Def: Company OS Workflow Interviewer

## Purpose

This skill interviews someone about **one specific workflow** — a repeatable process a colleague runs (onboarding a client, handling a support ticket, shipping a release, reimbursing an expense, ...) — and turns their tacit knowledge into a structured, reviewable document inside the company's "Company OS" knowledge base. It has two jobs at once:

1. **Document reality.** Capture the process as it is actually run today, broken into atomic steps, faithfully — not how it's supposed to run on paper.
2. **Surface gaps.** Compare the described process against curated best-practice playbooks for common workflow archetypes and flag steps that are de-facto standard elsewhere but missing here — as clearly-tagged *recommendations*, never silently merged in as if they were already practiced.

## Principles

- **One atomic action per step.** If an answer describes two actions, split it into two steps.
- **Suggestions are never facts.** A best-practice step or document the interviewee hasn't confirmed stays in a separate `## Recommendations` / `## Needed documents` section, tagged as a suggestion, until they explicitly confirm it's actually practiced or already exists.
- **Reality beats the playbook.** When the interviewee's actual practice conflicts with a canonical playbook step, document what they actually do — the playbook only exists to prompt questions, not to overrule the interview.
- **Never fabricate detail.** If a step is unclear, ask — don't infer specifics the interviewee didn't state.
- **Respect the target repo's own boundaries.** If the interview surfaces confidential material (pricing, strategy, unreleased plans) or personal/HR data (PII), stop documenting that part, tell the user it belongs in the company's separate confidential repo (commonly named `<name>-confidential`), and do not write it into this repo. This mirrors the target repo's own `AGENTS.md`/`CLAUDE.md` boundaries — always defer to what those files actually say over this assumption.

## Workflow

### Phase 0 — Kickoff

1. Ask for a **one-line description** of the workflow being documented (e.g. "how we onboard a new B2B client").
2. Ask what **output language** the final document should be written in — the document must be human-readable by the interviewee's colleagues, so don't assume it matches the conversation's language without confirming.
3. Determine the **target Gitea repo** (owner + repo name). If not already established in this conversation, look for candidates with `mcp__gitea__list_my_repos` / `mcp__gitea__list_org_repos`, favoring a repo whose root `START_HERE.md` matches the "Company OS — Start Here" skeleton signature (frontmatter `type: index`, mentions of a general-tier knowledge base, `knowledge/` domain table). Confirm the exact owner/repo with the user before writing anything — never guess silently.
4. Derive a kebab-case slug from the one-liner (used for the branch name and file name).

### Phase 1 — Load live conventions

Before writing anything, `mcp__gitea__get_file_contents` the target repo's **actual, current**:
- `START_HERE.md` — orientation and domain map.
- `CLAUDE.md` (preferred) or `AGENTS.md` — the working rules for agents in this repo.
- `KNOWLEDGE_BASE.md` — the index of what already exists.
- `templates/concept.md` — the frontmatter + body contract every file must follow.

**Always defer to what these files actually say over anything assumed here** — a customer may have customized their skeleton. If `START_HERE.md` and `templates/concept.md` are both missing, this isn't a Company OS repo: stop, explain what's missing, and ask the user to confirm the target or point at `template/WORKFLOW-TEMPLATE.md`'s fallback contract only as a last resort.

### Phase 2 — Duplicate check & domain placement

1. **Check whether `coos-mcp` is connected first** — look for `mcp__coos-mcp__search_knowledge` among this session's available tools.
   - **If connected:** call `search_knowledge` with `query` built from the one-liner's key terms, scoped to the target repo via `org`/`repo`, and `type: "playbook"` to narrow to workflow docs. If the first query comes back empty, retry once with a broader synonym before concluding nothing exists. This returns titles/descriptions/tags/400-char snippets across the whole repo in a single call, instead of a recursive tree walk plus full-text reads of every candidate file — much cheaper and faster, and it's the preferred path whenever available.
   - **If not connected (fallback):** check `KNOWLEDGE_BASE.md`'s domain table and search `knowledge/teams/**` (via `mcp__gitea__get_repository_tree`, recursive) for a file that already documents this workflow — per the target repo's own "do not create a duplicate concept" rule.
2. **If a plausible match turns up** (via either path): `get_file_contents` only that one candidate file to load its full content, summarize its current steps back to the user, and treat this session as a **refine** pass. Ask whether this is (a) an in-place edit/extension of the same file, or (b) a full replacement — a replacement sets `supersedes`/`superseded_by` and archives the old file in Phase 7.
3. **If nothing is found:** ask which team/domain owns this workflow (sales / marketing / design / brand / product / eng / ops / other — offer "other" with a custom folder name) to target `knowledge/teams/<domain>/<slug>.md`. Do not default to `knowledge/source-of-truth/` — that folder is for already-reviewed, promoted knowledge only, and this is a fresh, unverified draft.

### Phase 3 — Branch + draft PR

1. `mcp__gitea__create_branch` off `main`, named `workflow-def/<slug>`.
2. Initialize the file's frontmatter from the **live** `templates/concept.md` contract read in Phase 1:
   - `type: playbook`
   - `title`: the workflow name
   - `description`: the one-liner
   - `classification: public-internal` (this skill never writes `confidential` or `restricted-pii` — see Principles)
   - `provenance: agent-draft-unverified`
   - `authority`: ask who owns / is authoritative for this workflow (a team or a named role)
   - `verified_by` / `verified_on`: left empty
   - `review_by`: propose roughly 6 months out, confirm with the user
   - `supersedes` / `superseded_by`: per the Phase 2 decision, otherwise `none`
   - `tags`: a few relevant keywords
3. Commit this skeleton via `mcp__gitea__create_or_update_file`, then open a **draft/WIP Pull Request** (`mcp__gitea__pull_request_write`, title prefixed `[WIP]`) from the branch into `main`, with a body linking back to the one-liner and noting it's an in-progress interview. This makes progress visible from the start.

### Phase 4 — Playbook matching

Match the one-liner against this skill's own bundled reference files in `playbooks/` (local skill assets, read via `Read` — separate from the target repo):

- `playbooks/employee-onboarding.md`
- `playbooks/employee-offboarding.md`
- `playbooks/incident-response.md`
- `playbooks/expense-reimbursement.md`
- `playbooks/procurement-vendor-onboarding.md`
- `playbooks/sales-lead-to-cash.md`
- `playbooks/customer-support-ticket.md`
- `playbooks/change-management-release.md`

Each playbook carries three lists: **Canonical steps**, **Commonly missed steps**, and **Needed documents** (the templates/forms/checklists that typically support this workflow — offer letters, runbooks, approval forms, and the like). Load the 0-2 most relevant playbooks as internal grounding for later best-practice suggestions, covering both missing steps and missing documents. If none genuinely match, say so and proceed on general knowledge alone — don't force a fit onto an unrelated workflow.

### Phase 5 — Interview loop (the "grill" core)

For each step, ask specific, narrow questions to extract **one atomic action at a time**:
- What triggers this step?
- Who does it (actor/role)?
- What exactly do they do?
- What tool/system do they use?
- What input do they consume, and what output do they produce?
- Is there a decision point or branch here?
- Who/what do they hand off to next?
- What exceptions or edge cases exist?
- Does this step use, produce, or require a specific document/form/template (contract, checklist, approval form, runbook, ...)? If so, does that document already exist and where does it live, or is it handled ad hoc?

Split compound answers into multiple atomic steps rather than storing a paragraph as one step.

After each step is captured, check it against the loaded playbook(s) from Phase 4. If a canonical neighboring step from the playbook's "Canonical steps" or "Commonly missed steps" list is absent from what's been described, propose it explicitly, e.g.: *"Companies doing [X] usually also [Y] around here — is that something you do too, or should I flag it as a gap?"* Three possible outcomes:
- **Confirmed as practiced** → document it as a normal as-is step.
- **Flagged as recommendation** → add it to the `## Recommendations` section, tagged as a suggestion, not merged into the as-is steps.
- **Not applicable** → drop it, don't mention it again for this workflow.

Apply the same three-outcome pattern to the playbook's **Needed documents** list: for each document the interviewee hasn't already mentioned, ask whether it already exists (confirmed → recorded in `## Needed documents` as Exists, with a link/location if known), should be created (flagged as recommendation → recorded as Recommended), or doesn't apply here (dropped).

After every single add/edit/delete, commit the updated file via `mcp__gitea__create_or_update_file` — each change is a new commit on the open branch/PR. This is the incremental-persistence behavior: nothing is held only in conversation state.

### Phase 6 — Menu-driven refinement

After each unit of interview progress, present a numbered menu and wait for the user's choice:

1. Continue to the next step
2. Edit an existing step (by number)
3. Delete a step
4. Reorder / insert a step at a specific position
5. Deep-dive an existing step (more detail, exceptions, sub-notes)
6. Review pending best-practice recommendations — steps and needed documents (accept as practiced/existing / keep as suggestion / reject)
7. Show the current draft
8. Finalize
9. Pause / exit (already safely committed to the branch — nothing is lost)

Loop here until the user picks Finalize or Exit. Every mutating choice (1-6) triggers another commit per Phase 5's persistence rule.

### Phase 7 — Finalize

1. Do a consistency pass: renumber steps, check actor/tool naming is consistent throughout.
2. Keep confirmed recommendations in the distinct `## Recommendations` section, and confirmed/proposed documents in `## Needed documents` — never silently merge either into `## Detail`.
3. If this document supersedes an older one (per the Phase 2 decision): update the old file's frontmatter (`superseded_by: <new file path>`) and move it to `knowledge/archive/` — `mcp__gitea__create_or_update_file` for the archive copy, `mcp__gitea__delete_file` for the old path, both on the same branch.
4. Take the Pull Request out of draft, update its title (drop `[WIP]`) and description with a finalize summary. Do **not** merge it — `main` is branch-protected and requires human review, per the target repo's own rules.
5. Report the PR URL and a short summary to the user: step count, open recommendation count, needed-documents count, and files touched.

## Quality gates

- Never write to `main` directly — every change goes through the branch + PR opened in Phase 3.
- Never invent a step's detail — if unclear, ask.
- Never write `classification: confidential` or `restricted-pii`, and never document PII/HR specifics — redirect that content out of scope per Principles.
- Always re-check the *live* target repo's `templates/concept.md` and `AGENTS.md`/`CLAUDE.md` before assuming this skill's own bundled fallback template or conventions apply — the target repo is the source of truth for its own contract.
- Before creating a new file, always check for an existing one on the same topic (Phase 2) — do not create duplicates.
