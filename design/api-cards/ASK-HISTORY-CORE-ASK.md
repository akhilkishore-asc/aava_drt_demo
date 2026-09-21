# ASK-HISTORY-CORE-ASK - GET /api/v1/core-asks/{id}/history

| Field | Value |
|---|---|
| API ID | ASK-HISTORY-CORE-ASK |
| Operation | GET /api/v1/core-asks/{id}/history - operationId listCoreAskHistory |
| Module | ASK |
| Purpose | Return the history of one Core ASK, newest first, one page at a time. Changes nothing. |
| Complexity | Simple - a paged read of audit records for one ASK |
| Sources | US-ASK-005; ADO 1861; FR1-FR5; BR1-BR6; AC1-AC5 |
| Standards | API 001, API 002, API 003, SEC 001, SEC 002, SEC 003, DATA 004, OBS 001, TEST 001 |
| Status | Approved |

## Settled decisions

| Item | Spec said | Master LLD said | Settled |
|---|---|---|---|
| Route | `GET /core-asks/{id}/history` | `GET /api/v1/asks/{id}/audit` | `GET /api/v1/core-asks/{id}/history` - a child collection under the collection settled in ADR-DRT-API-001 |
| Paging style | Continuation token | Continuation token or page number | **Continuation token**, the same as the list endpoint (ADO 1859) |

## Contract summary

| Field | Value |
|---|---|
| Path parameter | `id` - string, format uuid, required. The askId |
| Query parameters | `pageSize` - integer int32, optional, minimum 1, maximum 100 (default 25). `continuationToken` - string, optional, maxLength 512 |
| Request body | None |
| Response schema | CoreAskHistoryResponse - items (array of CoreAskHistoryEntry), pageSize, continuationToken |
| Success status | **200 OK** |
| Idempotency | Not applicable - read-only |
| Concurrency | Not applicable - read-only |
| Sort | performedOn descending, then auditId ascending. Fixed |

## Response fields - CoreAskHistoryResponse

| Field | Type | Format | Required | Constraints | Description |
|---|---|---|---|---|---|
| items | array | | yes | items are CoreAskHistoryEntry | The history entries on this page |
| pageSize | integer | int32 | yes | minimum 1, maximum 100 | The page size applied |
| continuationToken | string | | no | maxLength 512 | Pass back to get the next page. Absent on the last page |

## Response fields - CoreAskHistoryEntry

| Field | Type | Format | Required | Constraints | Description |
|---|---|---|---|---|---|
| auditId | string | uuid | yes | | Identifier of the audit record |
| action | string | | yes | enum Created, Saved, Submitted | What happened |
| fromStatusId | integer | int32 | no | | Status before the action. Absent on Created |
| toStatusId | integer | int32 | yes | | Status after the action |
| version | integer | int32 | yes | minimum 1 | ASK version after the action |
| performedBy | string | uuid | yes | | Internal identifier of the user who acted |
| performedOn | string | date-time | yes | | When it happened, ISO 8601 UTC |

## Authorization

| Control | Value |
|---|---|
| Authentication | SEC 001 - Entra ID, OAuth 2.0 Authorization Code with PKCE |
| Permission | View ASK for the ASK's business unit (SEC 002) |
| Record scope | Route id is not authorization evidence (SEC 003) |
| Audit | Not required for reads |

## Processing flow

1. Validate id, pageSize and the continuation token; otherwise return 400.
2. Load the ASK; if none exists, return 404.
3. Check View ASK for its business unit; otherwise return 403.
4. Read its audit records, sorted by performedOn descending then auditId ascending.
5. Return 200 with up to pageSize entries and a continuation token when more remain.

## Errors and tests

| Area | Content |
|---|---|
| Errors | 400 validation_failed - bad id, pageSize out of range, or bad continuation token; 401 unauthenticated; 403 access_denied; 404 not_found; 503 dependency_unavailable. Every non-2xx returns ProblemDetails |
| Tests | Existing ASK gives 200 newest first; the Created entry has no fromStatusId; pageSize 2 gives 2 entries and a token; the token returns the next entries with no overlap; unknown id gives 404; pageSize 0 and 101 give 400; caller without permission gives 403 |

## Open items - none blocking

| ID | Item | Owner | Decision required |
|---|---|---|---|
| OI-1 | Whether field-level changes should be returned later | Product Owner | Confirm; it would be an additive optional field |
