# Examples

Copy the shape, not the product details.

## Feature

**Repo:** `epms-control-center-ui`
**Title:** `[FEAT] Center: PCO can assign a chat to themselves`

```markdown
### User Story
As a PCO
I want to assign a New conversation to myself in one click
So that the queue moves without opening a menu

### Why are we building this?
New conversations sit untouched because assign is buried. One-click assign is the smallest change that makes Mine reflect real work.

### Acceptance Criteria
- [ ] Scenario: PCO assigns a New chat to themselves
  - Given a conversation in New
  - When they click Assign to me
  - Then the card appears in Mine and the New count drops by 1
- [ ] Scenario: already assigned chat hides Assign to me
  - Given a conversation in Mine owned by this PCO
  - When they open it
  - Then Assign to me is not shown

### Figma
https://www.figma.com/design/example

### Design states
- Ideal: See Figma
- Empty: See Figma
- Loading: Assign to me is disabled until the request finishes
- Error: toast under the card, card stays in New

### Technical sub-issues
Devs add a Task in each implementing repo and link it as a sub-issue.
```

## Bug

**Repo:** `epms-control-center-ui`
**Title:** `[BUG] Center: Self-booking submit button returns 500 error`

```markdown
### Description
The self-booking submit button returns a 500 when the patient has no default clinic.

### Steps to Reproduce
1. Go to '/booking'
2. Sign in as a patient with no default clinic
3. Click 'Schedule Appointment'
4. Fill required fields
5. Click Submit
6. See error

### Expected Behavior
Should save the booking and show the confirmation screen.

### Actual Behavior
The API returns 500 and the button stays in a loading state.

### App
Control Center

### Environment
Production

### OS
macOS

### Browser
Chrome 126

### Device
Desktop

### Visual Proof
Screenshot attached.

### Additional Context
Request id on the failed call: `req_example`. No patient name in this ticket.
```

## Task

**Repo:** `epms-api`
**Title:** `[TASK] API: Persist lostReason when a conversation is marked LOST`
**Parent:** Feature in `epms-control-center-ui`

```markdown
### Parent
https://github.com/E-Konsulta-Medical-Clinic/epms-control-center-ui/issues/880

### This ships in
epms-api

### What this ships
LOST requires a lostReason. The API rejects LOST without it and stores the reason on the conversation.

### Done when
- [ ] Scenario: PCO marks a chat LOST with a reason
  - Given a conversation in an open state
  - When they submit LOST with a lostReason
  - Then the conversation is LOST and the reason is returned on GET
- [ ] Scenario: LOST without a reason is rejected
  - Given a conversation in an open state
  - When they submit LOST with no lostReason
  - Then the API returns 400 and the state does not change

### Out of scope
Control Center form. Messaging consumer.

### Watch-outs
Failed sends must not change conversation state.
```

## PR

**Branch:** `feat/451-lost-reason`
**Title:** `feat: #451 [API] Persist lostReason on conversation close`

```markdown
## Closes
Closes #451

## Parent
Part of E-Konsulta-Medical-Clinic/epms-control-center-ui#880

## What changed
- Reject LOST when lostReason is missing
- Persist lostReason on the conversation
- Return lostReason on conversation GET

## How to QA
- [ ] Scenario: PCO marks a chat LOST with a reason
  - Given a conversation in an open state
  - When they submit LOST with a lostReason
  - Then the conversation is LOST and the reason is returned on GET
- [ ] Scenario: LOST without a reason is rejected
  - Given a conversation in an open state
  - When they submit LOST with no lostReason
  - Then the API returns 400 and the state does not change

## Risk
- [x] No new patient data (PII) exposed
- Extra: none
```
