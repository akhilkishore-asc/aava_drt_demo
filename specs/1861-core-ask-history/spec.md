# Specification: 1861 Core ASK API – Core ASK history (US-ASK-005)

## Feature Overview
This feature provides an API endpoint that returns the history of a single Core ASK, one page at a time, newest first. Each entry records what happened to the ASK, who did it, when, and how its status and version changed. It is read-only.

## Business Objective
Give requestors, reviewers and auditors a clear record of how an ASK moved through the workflow, without querying the database.

## Functional Requirements

| ID | Requirement | Source |
|----|-------------|--------|
| FR1 | Expose GET /core-asks/{id}/history to list the history of one Core ASK | ADO Description - API Details |
| FR2 | Return entries newest first, one page at a time, using pageSize and a continuation token | ADO Description - API Details |
| FR3 | Return for each entry auditId, action, fromStatusId, toStatusId, version, performedBy and performedOn | ADO Description - API Details Response |
| FR4 | Return not found when the ASK does not exist | ADO Description - Business Rules |
| FR5 | Never change any data when called | ADO Description - Scope |

## Expected Behaviour

**Scenario: History of an existing ASK**
- When a caller with View ASK permission requests the history of an existing ASK
- The API returns HTTP 200 with up to pageSize entries, newest first, and a continuation token if more exist

**Scenario: ASK does not exist**
- When the id does not match any ASK
- The API returns HTTP 404

**Scenario: Invalid page size**
- When pageSize is below 1 or above 100
- The API returns HTTP 400

## Business Rules

| ID | Rule | Source |
|----|------|--------|
| BR1 | id must be a valid GUID | ADO Description - Business Rules |
| BR2 | pageSize is optional, defaults to 25, and must be between 1 and 100 | ADO Description - Business Rules |
| BR3 | action is one of Created, Saved, Submitted | ADO Description - Business Rules |
| BR4 | fromStatusId is absent on the Created entry, because there was no earlier status | ADO Description - Business Rules |
| BR5 | Sort order is performedOn descending, then auditId ascending | ADO Description - Business Rules |
| BR6 | A caller may view the history only if they hold View ASK for the ASK's business unit | ADO Description - Security |

## Acceptance Criteria

| ID | Criterion |
|----|-----------|
| AC1 | Given an existing ASK, when history is requested, then 200 is returned with entries newest first |
| AC2 | Given more entries than pageSize, when history is requested, then a continuationToken is returned and passing it returns the next page |
| AC3 | Given an unknown id, when history is requested, then 404 is returned with ProblemDetails |
| AC4 | Given pageSize 0 or 101, when history is requested, then 400 is returned with ProblemDetails |
| AC5 | Given a caller without View ASK permission, when history is requested, then 403 is returned with ProblemDetails |

## Dependencies
- Core ASK records and audit history written by ADO 1857 and ADO 1860
- Entra ID authentication (SEC 001)
- RBAC permission View ASK (SEC 002)

## API and Integration Considerations
- Read-only; no idempotency key is needed
- No request body
- Dates returned in ISO 8601 UTC

## Security Considerations
- Authentication through Entra ID, OAuth 2.0 Authorization Code with PKCE (SEC 001)
- Authorization enforced server-side per business unit (SEC 002); the route id is not authorization evidence (SEC 003)

## Error Behaviour

| Status | When |
|--------|------|
| 400 | id is not a GUID, pageSize out of range, or continuation token invalid |
| 401 | Caller is not authenticated |
| 403 | Caller lacks View ASK for the ASK's business unit |
| 404 | No ASK exists with the given id |
| 503 | A dependent service is unavailable |

All error responses return ProblemDetails.

## Assumptions
1. The history includes every status change and every save, but not reads.
2. performedBy is the internal user identifier; display names are resolved by the front end.

## Out of Scope
- Filtering history by action or date
- Returning the field-level changes of each save

## Open Questions
- Should field-level changes be returned in a later version?

## Traceability
US-ASK-005 · ADO 1861 · FR1-FR5 · BR1-BR6 · AC1-AC5
