# ASK-LIST-CORE-ASKS - GET /api/v1/core-asks

| Field | Value |
|---|---|
| API ID | ASK-LIST-CORE-ASKS |
| Operation | GET /api/v1/core-asks - operationId listCoreAsks |
| Module | ASK |
| Purpose | Return one page of the Core ASKs the caller may view, newest first, optionally filtered by workflow status. Changes nothing. |
| Complexity | Medium - paging with a continuation token and a status filter, scoped by business unit |
| Sources | US-ASK-003; ADO 1859; FR1-FR8; BR1-BR5; AC1-AC7 |
| Standards | API 001, API 002, API 003, SEC 001, SEC 002, SEC 003, DATA 004, OBS 001, TEST 001 |
| Status | Approved |

## Settled decisions

| Item | Spec said | Master LLD said | Settled |
|---|---|---|---|
| Route | `GET /core-asks` | `GET /api/v1/asks` | `GET /api/v1/core-asks` - same path as the create operation, per ADR-DRT-API-001. GET and POST on one collection path is standard |
| Paging style | Continuation token | Continuation token or page number | **Continuation token**, as the spec states. Stable under inserts |
| Total count | Open question | Not stated | Not returned. Can be added later as an optional field without a breaking change |

## Contract summary

| Field | Value |
|---|---|
| Path parameters | None |
| Query parameters | `pageSize` - integer int32, optional, minimum 1, maximum 100 (default 25). `continuationToken` - string, optional, maxLength 512. `statusId` - integer int32, optional, one of 123, 127, 145 |
| Request body | None |
| Response schema | CoreAskListResponse - items (array of CoreAskSummary), pageSize, continuationToken |
| Success status | **200 OK**, including when items is empty |
| Idempotency | Not applicable - read-only |
| Concurrency | Not applicable - read-only |
| Sort | lastModifiedOn descending, then askId ascending. Fixed, not caller-selectable |

## Response fields - CoreAskListResponse

| Field | Type | Format | Required | Constraints | Description |
|---|---|---|---|---|---|
| items | array | | yes | items are CoreAskSummary | The ASKs on this page |
| pageSize | integer | int32 | yes | minimum 1, maximum 100 | The page size applied |
| continuationToken | string | | no | maxLength 512 | Pass back to get the next page. Absent on the last page |

## Response fields - CoreAskSummary

| Field | Type | Format | Required | Constraints | Description |
|---|---|---|---|---|---|
| askId | string | uuid | yes | | Identifier of the ASK |
| coreAskName | string | | yes | maxLength 200 | Name of the ASK |
| statusId | integer | int32 | yes | | Current workflow status - 123, 127 or 145 |
| version | integer | int32 | yes | minimum 1 | Current version number |
| lastModifiedOn | string | date-time | yes | | When the ASK was last changed, ISO 8601 UTC |

## Authorization

| Control | Value |
|---|---|
| Authentication | SEC 001 - Entra ID, OAuth 2.0 Authorization Code with PKCE |
| Permission | View ASK, applied per business unit (SEC 002) |
| Record scope | Only ASKs in business units where the caller holds View ASK |
| Audit | Not required for reads |

## Processing flow

1. Validate pageSize (1-100), statusId (123, 127 or 145) and the continuation token; otherwise return 400.
2. Resolve the current DRT actor. If the caller holds View ASK in no business unit, return 403.
3. Query ASKs in the caller's business units, apply the status filter, sort by lastModifiedOn descending then askId ascending.
4. Return 200 with up to pageSize items and a continuation token when more remain.

## Errors and tests

| Area | Content |
|---|---|
| Errors | 400 validation_failed - pageSize out of range, statusId not allowed, or bad continuation token; 401 unauthenticated; 403 access_denied - no View ASK in any business unit; 503 dependency_unavailable. Every non-2xx returns ProblemDetails |
| Tests | No parameters gives 200 and at most 25 items newest first; pageSize 5 gives 5 items and a token; the token returns the next 5 with no overlap; statusId 127 returns only 127; pageSize 0 and 101 give 400; statusId 999 gives 400; a tampered token gives 400; no matches gives 200 with empty items and no token; caller without permission gives 403 |

## Open items - none blocking

| ID | Item | Owner | Decision required |
|---|---|---|---|
| OI-1 | Whether to add a total count of matching ASKs | Product Owner | Confirm if needed; it would be an additive optional field |
