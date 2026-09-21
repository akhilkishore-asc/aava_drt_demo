# Specification: 1858 Core ASK API – Get Core ASK by ID (US-ASK-002)

## Feature Overview
This feature provides an API endpoint to retrieve a single Core ASK record by its identifier through a GET operation. The endpoint returns the current version of the ASK, its business data and its current workflow status. It is read-only and changes no data.

## Business Objective
Allow authorized users to open an existing Core ASK so they can review it, continue editing a draft, or act on it in the review workflow, without searching through lists.

## Functional Requirements

| ID | Requirement | Source |
|----|-------------|--------|
| FR1 | Expose GET /core-asks/{id} endpoint to retrieve one Core ASK record | ADO Description - API Details |
| FR2 | Accept the ASK identifier (askId) as a path parameter | ADO Description - API Details |
| FR3 | Return askId, askDetailId, version, statusId and coreAskDetails for the current version | ADO Description - API Details Response |
| FR4 | Return createdOn and lastModifiedOn timestamps | ADO Description - API Details Response |
| FR5 | Return the same coreAskDetails fields that the create operation accepts | ADO Description - API Details |
| FR6 | Return not found when no ASK exists with the given identifier | ADO Description - Business Rules |
| FR7 | Never change any data when called | ADO Description - Scope |

## Expected Behaviour

**Scenario: Retrieve an existing Core ASK**
- When a caller with view permission requests an existing askId
- The API returns HTTP 200 with the current version of the ASK and its workflow status

**Scenario: Core ASK does not exist**
- When a caller requests an askId that does not exist
- The API returns HTTP 404

**Scenario: Invalid identifier**
- When a caller sends an askId that is not a valid GUID
- The API returns HTTP 400

**Scenario: Caller without permission**
- When a caller without view permission requests an ASK
- The API returns HTTP 403

## Business Rules

| ID | Rule | Source |
|----|------|--------|
| BR1 | askId must be a valid GUID | ADO Description - Business Rules |
| BR2 | Only the current (latest) version of the ASK is returned | ADO Description - Business Rules |
| BR3 | outgoingResource is returned as stored; it is derived at create time and never recalculated on read | ADO Description - Business Rules |
| BR4 | statusId is the current workflow status: 123 In Progress, 127 PPL Review or 145 DPP Ops Review | ADO Description - Business Rules |
| BR5 | A caller may view an ASK only if they hold the View ASK permission for its business unit | ADO Description - Security |

## Acceptance Criteria

| ID | Criterion |
|----|-----------|
| AC1 | Given an existing askId, when GET is called, then 200 is returned with askId, askDetailId, version, statusId, coreAskDetails, createdOn and lastModifiedOn |
| AC2 | Given an askId that does not exist, when GET is called, then 404 is returned with ProblemDetails |
| AC3 | Given an askId that is not a GUID, when GET is called, then 400 is returned with ProblemDetails |
| AC4 | Given a caller without View ASK permission, when GET is called, then 403 is returned with ProblemDetails |
| AC5 | Given any call, when GET completes, then no data has changed |

## Dependencies
- Core ASK records created through POST /api/v1/core-asks (ADO 1857)
- Entra ID authentication (SEC 001)
- RBAC permission View ASK (SEC 002)

## API and Integration Considerations
- Read-only operation; no idempotency key is needed
- No request body
- Dates returned in ISO 8601 UTC

## Security Considerations
- Authentication through Entra ID, OAuth 2.0 Authorization Code with PKCE (SEC 001)
- Authorization enforced server-side per business unit (SEC 002); the route ID is not authorization evidence (SEC 003)

## Error Behaviour

| Status | When |
|--------|------|
| 400 | askId is not a valid GUID |
| 401 | Caller is not authenticated |
| 403 | Caller lacks View ASK permission for the record's business unit |
| 404 | No ASK exists with the given askId |
| 503 | A dependent service is unavailable |

All error responses return ProblemDetails.

## Assumptions
1. An ASK that the caller is not allowed to see returns 403, not 404.
2. Draft ASKs (status 123) are visible to callers with View ASK permission, not only to their creator.

## Out of Scope
- Listing or searching ASKs
- Returning previous versions
- Returning comments, attachments or audit history

## Open Questions
- Should a caller without permission receive 404 instead of 403 to avoid revealing that the record exists?

## Traceability
US-ASK-002 · ADO 1858 · FR1-FR7 · BR1-BR5 · AC1-AC5
