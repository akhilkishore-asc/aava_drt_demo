# ASK-SUBMIT-CORE-ASK - POST /api/v1/core-asks/{id}/submissions

| Field | Value |
|---|---|
| API ID | ASK-SUBMIT-CORE-ASK |
| Operation | POST /api/v1/core-asks/{id}/submissions - operationId submitCoreAsk |
| Module | ASK |
| Purpose | Submit a draft Core ASK for review, moving it from In Progress to review and creating the review task. |
| Complexity | Medium - one workflow transition with a version check, an idempotency key and a routing rule |
| Sources | US-ASK-004; ADO 1860; FR1-FR8; BR1-BR6; AC1-AC7 |
| Standards | API 001, API 002, API 003, SEC 001, SEC 002, SEC 003, DATA 003, WF 001, WF 003, REL 001, OBS 001, TEST 001 |
| Status | Approved |

## Settled decisions

| Item | Spec said | Master LLD said | Settled |
|---|---|---|---|
| Route | `POST /core-asks/{id}/submissions` | `POST /api/v1/asks/{id}/submit` | `POST /api/v1/core-asks/{id}/submissions` - a noun sub-path, as the standards require for state-changing actions, under the collection settled in ADR-DRT-API-001 |
| Concurrency | "Submit the version you saw" | ETag or expectedVersion | **expectedVersion in the request body**, the same way on every DRT write |
| Business rule failures | Not stated | 400 or 409 | Wrong state and stale version are **409**. Malformed input is **400**. 422 is not used, per ADR-DRT-API-001 |
| Success status | Not stated | Not stated | **200 OK** with the updated ASK summary. Nothing new is created at this URL, so not 201 |

## Contract summary

| Field | Value |
|---|---|
| Path parameter | `id` - string, format uuid, required. The askId |
| Header | `Idempotency-Key` - string, required, maxLength 128 |
| Request schema | SubmitCoreAskRequest - expectedVersion, comment |
| Response schema | CoreAskSubmissionResponse - askId, version, statusId, taskId, submittedOn |
| Success status | **200 OK** |
| Idempotency | Required. A repeat with the same key and body returns the original result |
| Concurrency | expectedVersion must match the current version, otherwise 409 |

## Request fields - SubmitCoreAskRequest

| Field | Type | Format | Required | Constraints | Description |
|---|---|---|---|---|---|
| expectedVersion | integer | int32 | yes | minimum 1 | The version the caller is submitting |
| comment | string | | no | maxLength 1000 | Optional note for the reviewer |

## Response fields - CoreAskSubmissionResponse

| Field | Type | Format | Required | Constraints | Description |
|---|---|---|---|---|---|
| askId | string | uuid | yes | | Identifier of the ASK |
| version | integer | int32 | yes | minimum 1 | The ASK's version after submission |
| statusId | integer | int32 | yes | | New workflow status, 127 or 145 |
| taskId | string | uuid | yes | | Identifier of the review task created |
| submittedOn | string | date-time | yes | | When the ASK was submitted, ISO 8601 UTC |

## Authorization

| Control | Value |
|---|---|
| Authentication | SEC 001 - Entra ID, OAuth 2.0 Authorization Code with PKCE |
| Permission | The ASK's creator, or Submit ASK for its business unit (SEC 002) |
| Role condition | Leadership-group membership decides routing; read from the caller profile, never the request (SEC 003) |
| Audit | Submitter, time, old and new status, version |

## Processing flow

1. Validate the Idempotency-Key, the id and the body; otherwise return 400.
2. If the Idempotency-Key was used before with the same body, return the stored result.
3. Load the ASK; if none exists, return 404.
4. Check the caller may submit it; otherwise return 403.
5. If the ASK is not in status 123, or expectedVersion is not current, return 409.
6. Route to 127, or 145 for a leadership-group submitter.
7. In one transaction, update the status and version, create the review task, store the comment, write the audit record and the outbox event (DATA 003, WF 003).
8. Return 200 with CoreAskSubmissionResponse.

## Errors and tests

| Area | Content |
|---|---|
| Errors | 400 validation_failed - bad body, comment too long, or missing Idempotency-Key; 401 unauthenticated; 403 access_denied; 404 not_found; 409 conflict - stale version, wrong status, or key reused with a different body; 503 dependency_unavailable. Every non-2xx returns ProblemDetails |
| Tests | Draft with current version gives 200 and status 127; leadership submitter gives 145; stale version gives 409 and no change; already submitted gives 409; unknown id gives 404; caller without permission gives 403; missing Idempotency-Key gives 400; same key twice returns the same taskId; comment of 1001 characters gives 400 |

## Open items - none blocking

| ID | Item | Owner | Decision required |
|---|---|---|---|
| OI-1 | Whether the review task assignee should be returned | Product Owner | Confirm; it would be an additive optional field |
