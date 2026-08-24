---
name: ek-github-workflow
description: >-
  Creates and refines E-Konsulta GitHub issues (Bug, Feature, Task) and pull
  request bodies using org templates, issue-home rules, and close rules.
  Use when the user asks to create or refine a ticket, write acceptance
  criteria, split sub-issues, open a PR, or write a PR description.
---

# E-Konsulta GitHub workflow

Org handbook (humans, sarcastic): https://github.com/E-Konsulta-Medical-Clinic/.github/blob/main/CONTRIBUTING.md

This skill is dry on purpose. Tickets and PRs are short English. No jokes. No essays.

**Ask before creating or editing anything on GitHub.** Draft first.

Filled examples: [examples.md](examples.md)

## Language

- Present tense. Short sentences. Checklists over prose.
- Each `Then` is something QA can see or click. Not "works well." Not implementation details.
- PR `What changed`: 3–5 bullets. A reviewer should understand it in 20 seconds.
- Never paste tokens, passwords, or patient names into issues or PRs.

## Pick the type

| Type | Use when | Title |
|---|---|---|
| **Feature** | New behaviour. Parent. Business value + QA checklist. | `[FEAT] <App>: <short outcome>` |
| **Task** | One repo will ship part of a Feature (or a Bug). This is what a PR closes. | `[TASK] <Repo>: <what this repo ships>` |
| **Bug** | Unexpected behaviour. Standalone or sub-issue of a Feature. | `[BUG] <App>: <present tense problem>` |

App names in titles: `Center`, `Patient`, `Doctor`, `Partner`, `Web`, `API`, `Messaging`.

If the request is "fix the button," it is a Bug. If it is "add a report," it is a Feature. Do not file a Feature named like a Bug.

## Pick the repo (issue home)

**Feature and standalone Bug** → the **owning repo**. Default: the primary app a human opens. The parent can live in a backend when that repo owns the work.

| If the story is mainly… | Parent repo |
|---|---|
| Control Center, PCO, Chat Manager UI | `E-Konsulta-Medical-Clinic/epms-control-center-ui` |
| Patient app | `E-Konsulta-Medical-Clinic/epms-patient-ui` |
| Doctor app | `E-Konsulta-Medical-Clinic/epms-doctor-ui` |
| Partner app | `E-Konsulta-Medical-Clinic/epms-partner-ui` |
| Public website (current site) | `E-Konsulta-Medical-Clinic/epms-web-ui` |
| API-owned, or no UI yet | `E-Konsulta-Medical-Clinic/epms-api` |
| Messaging-owned | `E-Konsulta-Medical-Clinic/epms-messaging` |
| A worker / consumer | that consumer repo |

**Task** → every repo that ships code for that Feature, linked as a GitHub sub-issue. Cross-repo is expected. Other **frontends** are Tasks too, not only API and workers.

Examples:
- Parent in Center → Tasks in Patient, Doctor, `epms-api`, `epms-messaging` as needed.
- Parent in `epms-api` → Tasks in Center / Patient / Doctor if those UIs change.
- Chat Manager: parent in Center **or** `epms-messaging` if backend-owned; UI Task still goes in Center.

Do not file another app's work as a comment on the parent. Make a Task in that app's repo.

## Refine a Feature before splitting Tasks

1. Read the issue (`gh issue view`).
2. Read the relevant code.
3. Rewrite Acceptance Criteria as a QA checklist (format below). Drop anything QA cannot observe.
4. Propose Tasks: one per implementing repo, including other frontends. Each Task `Done when` is the subset of parent AC that repo covers.
5. Show the draft. Create only after the user agrees.

Rule: if it is not in the parent AC list, it will not get built. Do not invent extra scope.

## Issue bodies

Use these headings. They match the GitHub Issue Forms. Write the body in a temp file and pass `--body-file`.

### Feature

```markdown
### User Story
As a [type of user]
I want to [the action]
So that [the business value]

### Why are we building this?
[One or two sentences.]

### Acceptance Criteria
- [ ] Scenario: [who does what]
  - Given [starting state]
  - When [the action]
  - Then [what QA can see]

### Figma
[URL or "None"]

### Design states
- Ideal: [what it looks like when it works / See Figma]
- Empty: [no data]
- Loading: [spinner, skeleton, disabled control]
- Error: [where the message goes]

### Technical sub-issues
Devs add a Task in each implementing repo and link it as a sub-issue. Do not leave fake `#___` checklists.
```

Design states are required. `See Figma` is allowed only when a Figma URL is present.

### Bug

```markdown
### Description
[One or two sentences. Present tense.]

### Steps to Reproduce
1. Go to '...'
2. Click on '...'
3. See error

### Expected Behavior
Should [verb] ...

### Actual Behavior
[Present tense.]

### App
[Control Center | Patient Portal | Doctor UI | Partner UI | EPMS API | Messaging | Notification Consumer]

### Environment
[Production | Staging | Local]

### OS
### Browser
### Device

### Visual Proof
[Note if screenshots will be attached.]

### Additional Context
```

Steps are imperative commands. Expected starts with `Should`.

### Task

```markdown
### Parent
[full issue URL]

### This ships in
[repo name]

### What this ships
[1–3 sentences. What this repo changes. Not how.]

### Done when
- [ ] Scenario: [copied from parent AC this Task covers]
  - Given ...
  - When ...
  - Then ...

### Out of scope
[optional]

### Watch-outs
[optional one-liners: permission, migration, flag, contract, PII]
```

No architecture novel. If the parent AC is still vague, refine the parent first.

## Create on GitHub

```bash
gh issue create \
  --repo E-Konsulta-Medical-Clinic/<repo> \
  --title "[FEAT] Center: ..." \
  --body-file /tmp/issue.md \
  --label enhancement
```

Bug uses `--label bug`. Task has no label.

`gh issue create` does not set Issue Type. Set it after create.

Org issue type node IDs (confirm with `gh api orgs/E-Konsulta-Medical-Clinic/issue-types` if create fails):

| Name | `issueTypeId` |
|---|---|
| Task | `IT_kwDODEbKJs4BkJLd` |
| Bug | `IT_kwDODEbKJs4BkJLe` |
| Feature | `IT_kwDODEbKJs4BkJLf` |

```bash
# node id of the new issue
ISSUE_ID=$(gh issue view <n> --repo E-Konsulta-Medical-Clinic/<repo> --json id --jq .id)

gh api graphql \
  -f query='mutation($id:ID!, $type:ID!) { updateIssue(input:{id:$id, issueTypeId:$type}) { issue { number issueType { name } } } }' \
  -f id="$ISSUE_ID" \
  -f type="IT_kwDODEbKJs4BkJLe"
```

### Link a Task (or Bug) as a sub-issue

```bash
PARENT_ID=$(gh issue view <parent> --repo E-Konsulta-Medical-Clinic/<parent-repo> --json id --jq .id)
CHILD_URL=$(gh issue view <child> --repo E-Konsulta-Medical-Clinic/<child-repo> --json url --jq .url)

gh api graphql \
  -f query='mutation($parent:ID!, $url:String!) { addSubIssue(input:{issueId:$parent, subIssueUrl:$url}) { subIssue { number url } } }' \
  -f parent="$PARENT_ID" \
  -f url="$CHILD_URL"
```

## Branches, PRs, close rules

One Task or Bug = one branch = one PR.

Branch (number is the issue **this PR closes**, never the parent Feature):

```
feat/<id>-<short-slug>
fix/<id>-<short-slug>
chore/<id>-<short-slug>
```

`feat` = Feature Task. `fix` = Bug. `chore` = debt / devops.

PR title: `feat: #451 [API] Persist lostReason on conversation close`

### PR body

```markdown
## Closes
Closes #<task-or-bug>

## Parent
Part of #<feature>

## What changed
- 

## How to QA
- [ ]

## Risk
- [ ] No new patient data (PII) exposed
- Extra: none | migration | permission | flag
```

| Issue | Close how |
|---|---|
| Task or Bug | `Closes #<id>` in the PR. GitHub closes it on merge. |
| Feature | `Part of #<feature>` only. Human closes after QA ticks parent AC on staging. |

Never put `Closes` on the parent Feature. Omit the Parent section on a standalone Bug.

How to QA = the `Done when` / AC this PR covers, as checkboxes. Not a staging sign-off of the whole Feature.

When opening the PR, use `gh pr create` with this body. Do not let `Closes` point at the Feature.


## Board statuses (EPMS Portals project)

Automation (`board-automation.yml`) moves tickets. Do not drag what it owns.

Manual moves only:

- PO / lead: For Business Refinement → Tech Refinement → Next Sprint → Todo (This Sprint)
- Dev: Todo (This Sprint) → In Progress when starting work
- QA: Staging → Ready to be Merged after staging sign-off
- Leads: → LOCKED (comment why). Sophia: Done → Announced. Admin: → COMPUTED

Automatic:

- Issue opened: Feature/Bug → For Business Refinement. Task → parent status, else Tech Refinement.
- PR ready for review → Code Review. Review requests changes → Changes Requested.
- PR merged to staging: Task → Done (Sub-Tasks). Bug → Staging. Parent Feature → Staging when all sub-issues close.
- Release (staging → main): Ready to be Merged items → Done (Feature) or Bugs - Fixed (Bug).

PR gate: every non-draft PR fails unless the ticket it closes has an issue Type, an assignee, Requestor, and Sprint. Fix the ticket, then re-run the failed check.

Full guide: https://github.com/E-Konsulta-Medical-Clinic/.github/blob/main/BOARD_GUIDE.md
