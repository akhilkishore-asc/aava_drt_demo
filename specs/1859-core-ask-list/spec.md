# Specification: 1859 Core ASK API – List Core ASKs (US-ASK-003)

## Feature Overview
This feature provides an API endpoint that returns a page of Core ASK records the caller is allowed to see, so users can find an ASK without knowing its identifier. Results can be filtered by workflow status and are returned in a stable order, one page at a time.

## Business Objective
Give reviewers and requestors a working list of Core ASKs so they can pick up drafts, track submissions and act on items waiting for review.

## Functional Requirements

| ID | Requirement | Source |
|----|-------------|--------|
| FR1 | Expose GET /core-asks endpoint to list Core ASK records | ADO Description - API Details |
| FR2 | Return results one page at a time, using pageSize and a continuation token | ADO Description - API Details |
| FR3 | Allow filtering by statusId | ADO Description - API Details |
| FR4 | Return for each ASK its askId, coreAskName, statusId, version and lastModifiedOn | ADO Description - API Details Response |
| FR5 | Return a continuation token when more results exist, and none on the last page | ADO Description - API Details Response |
| FR6 | Return results newest first by lastModifiedOn, with askId as the tie-breaker | ADO Description - Business Rules |
| FR7 | Only return ASKs in business units the caller may view | ADO Description - Security |
| FR8 | Never change any data when called | ADO Description - Scope |

## Expected Behaviour

**Scenario: First page**
- When a caller with View ASK permission calls the endpoint without a continuation token
- The API returns HTTP 200 with up to pageSize items, newest first, and a continuation token if more exist

**Scenario: Next page**
- When the caller passes the continuation token from the previous response
- The API returns the next page in the same order

**Scenario: Filter by status**
- When the caller passes statusId 127
- The API returns only ASKs currently in status 127

**Scenario: Nothing to show**
- When no ASK matches
- The API returns HTTP 200 with an empty items list and no continuation token

**Scenario: Invalid page size**
- When pageSize is below 1 or above 100
- The API returns HTTP 400

## Business Rules

| ID | Rule | Source |
|----|------|--------|
| BR1 | pageSize is optional, defaults to 25, and must be between 1 and 100 | ADO Description - Business Rules |
| BR2 | statusId, when given, must be one of 123, 127 or 145 | ADO Description - Business Rules |
| BR3 | The continuation token is opaque to the caller and must be passed back unchanged | ADO Description - Business Rules |
| BR4 | Sort order is lastModifiedOn descending, then askId ascending | ADO Description - Business Rules |
| BR5 | Results are limited to business units where the caller holds View ASK | ADO Description - Security |

## Acceptance Criteria

| ID | Criterion |
|----|-----------|
| AC1 | Given ASKs exist, when GET is called with no parameters, then 200 is returned with up to 25 items sorted newest first |
| AC2 | Given more than pageSize ASKs, when GET is called, then a continuationToken is returned and passing it returns the next page |
| AC3 | Given statusId 127, when GET is called, then every item has statusId 127 |
| AC4 | Given pageSize 0 or 101, when GET is called, then 400 is returned with ProblemDetails |
| AC5 | Given statusId 999, when GET is called, then 400 is returned with ProblemDetails |
| AC6 | Given a caller without View ASK permission, when GET is called, then 403 is returned with ProblemDetails |
| AC7 | Given an invalid or expired continuation token, when GET is called, then 400 is returned with ProblemDetails |

## Dependencies
- Core ASK records created through POST /api/v1/core-asks (ADO 1857)
- Entra ID authentication (SEC 001)
- RBAC permission View ASK (SEC 002)

## API and Integration Considerations
- Read-only; no idempotency key is needed
- No request body
- Only allowlisted filters are accepted; no free-form filter expression (DRT API Design Standards section 6)
- Dates returned in ISO 8601 UTC

## Security Considerations
- Authentication through Entra ID, OAuth 2.0 Authorization Code with PKCE (SEC 001)
- Business-unit scoping enforced server-side (SEC 002)

## Error Behaviour

| Status | When |
|--------|------|
| 400 | pageSize out of range, statusId not allowed, or continuation token invalid |
| 401 | Caller is not authenticated |
| 403 | Caller holds View ASK in no business unit |
| 503 | A dependent service is unavailable |

All error responses return ProblemDetails.

## Assumptions
1. A caller with View ASK in at least one business unit receives 200, even when no ASK is visible to them.
2. The continuation token is valid for 24 hours.

## Out of Scope
- Free-text search
- Sorting by any other field
- Returning full coreAskDetails for each item (use GET /api/v1/core-asks/{id})

## Open Questions
- Should the total count of matching ASKs be returned?

## Traceability
US-ASK-003 · ADO 1859 · FR1-FR8 · BR1-BR5 · AC1-AC7
