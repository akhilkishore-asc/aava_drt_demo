# Specification: 1860 Core ASK API – Submit a draft Core ASK for review (US-ASK-004)

## Feature Overview
This feature provides an API endpoint that submits an existing draft Core ASK for review. A requestor who saved an ASK with Save & Exit can later send it into the review workflow without re-entering it. The endpoint moves the ASK from status 123 In Progress to 127 PPL Review, or to 145 DPP Ops Review when the submitter belongs to the leadership group, and creates the review task.

## Business Objective
Let requestors finish and submit drafts in a separate step, so work started earlier is not lost and reaches reviewers promptly.

## Functional Requirements

| ID | Requirement | Source |
|----|-------------|--------|
| FR1 | Expose POST /core-asks/{id}/submissions to submit a draft Core ASK | ADO Description - API Details |
| FR2 | Accept an optional comment with the submission | ADO Description - API Details |
| FR3 | Require the caller to state the version they are submitting, so a stale copy cannot be submitted | ADO Description - Business Rules |
| FR4 | Move the ASK to status 127, or to 145 for a leadership-group submitter | ADO Description - Business Rules |
| FR5 | Create the review task and return its identifier | ADO Description - API Details Response |
| FR6 | Return askId, version, statusId, taskId and submittedOn | ADO Description - API Details Response |
| FR7 | Record the submission in the audit history | ADO Description - Business Rules |
| FR8 | Make a repeated submission with the same Idempotency-Key return the original result | ADO Description - Business Rules |

## Expected Behaviour

**Scenario: Submit a draft**
- When the creator of a draft ASK in status 123 submits it with the current version
- The API moves it to status 127 (or 145 for a leadership-group submitter), creates the review task, and returns HTTP 200

**Scenario: Stale version**
- When the caller submits with a version that is no longer current
- The API returns HTTP 409 and changes nothing

**Scenario: Already submitted**
- When the ASK is not in status 123
- The API returns HTTP 409 and changes nothing

**Scenario: Retry**
- When the caller repeats the same request with the same Idempotency-Key
- The API returns the original result and does not submit twice

**Scenario: Not found**
- When no ASK exists with the given identifier
- The API returns HTTP 404

## Business Rules

| ID | Rule | Source |
|----|------|--------|
| BR1 | Only an ASK in status 123 In Progress can be submitted | ADO Description - Business Rules |
| BR2 | expectedVersion must equal the ASK's current version | ADO Description - Business Rules |
| BR3 | A leadership-group submitter routes to 145, anyone else to 127 | ADO Description - Business Rules |
| BR4 | comment is optional and at most 1000 characters | ADO Description - Business Rules |
| BR5 | Only the ASK's creator, or a caller with Submit ASK permission for its business unit, may submit | ADO Description - Security |
| BR6 | Idempotency-Key is required; a repeat with the same key and body returns the original result | ADO Description - Business Rules |

## Acceptance Criteria

| ID | Criterion |
|----|-----------|
| AC1 | Given a draft in status 123 and the current version, when submitted, then 200 is returned with statusId 127 and a taskId |
| AC2 | Given a leadership-group submitter, when submitted, then statusId is 145 |
| AC3 | Given a stale expectedVersion, when submitted, then 409 is returned with ProblemDetails and nothing changes |
| AC4 | Given an ASK not in status 123, when submitted, then 409 is returned with ProblemDetails |
| AC5 | Given an unknown id, when submitted, then 404 is returned with ProblemDetails |
| AC6 | Given a caller who may not submit this ASK, when submitted, then 403 is returned with ProblemDetails |
| AC7 | Given the same Idempotency-Key twice, when submitted, then the second call returns the first result |

## Dependencies
- Core ASK records created through POST /api/v1/core-asks (ADO 1857)
- Entra ID authentication (SEC 001)
- RBAC permission Submit ASK (SEC 002)
- Workflow task service (WF 001)

## API and Integration Considerations
- Retry-sensitive workflow command, so an Idempotency-Key header is required (REL 001)
- Concurrency is controlled with expectedVersion in the request body (DRT API Design Standards section 6)
- Dates returned in ISO 8601 UTC

## Security Considerations
- Authentication through Entra ID, OAuth 2.0 Authorization Code with PKCE (SEC 001)
- Leadership-group membership is read from the caller's server-side profile, never from the request (SEC 003)

## Error Behaviour

| Status | When |
|--------|------|
| 400 | Missing or malformed field, comment too long, or Idempotency-Key missing |
| 401 | Caller is not authenticated |
| 403 | Caller may not submit this ASK |
| 404 | No ASK exists with the given id |
| 409 | expectedVersion is stale, the ASK is not in status 123, or the Idempotency-Key was reused with a different body |
| 503 | A dependent service is unavailable |

All error responses return ProblemDetails.

## Assumptions
1. Submitting increments the ASK version by one.
2. The comment, when given, is stored as a new comment on the ASK.

## Out of Scope
- Withdrawing a submitted ASK
- Editing ASK fields during submission

## Open Questions
- Should the review task assignee be returned in the response?

## Traceability
US-ASK-004 · ADO 1860 · FR1-FR8 · BR1-BR6 · AC1-AC7
