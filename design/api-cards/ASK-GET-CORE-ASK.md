# ASK-GET-CORE-ASK - GET /api/v1/core-asks/{id}

| Field | Value |
|---|---|
| API ID | ASK-GET-CORE-ASK |
| Operation | GET /api/v1/core-asks/{id} - operationId getCoreAsk |
| Module | ASK |
| Purpose | Return one Core ASK by its identifier: the current version, its business data and its workflow status. Changes nothing. |
| Complexity | Simple - single read, no workflow change |
| Sources | US-ASK-002; ADO 1858; FR1-FR7; BR1-BR5; AC1-AC5 |
| Standards | API 001, API 002, API 003, SEC 001, SEC 002, SEC 003, DATA 004, OBS 001, TEST 001 |
| Status | Approved |

## Settled decisions

| Item | Spec said | Master LLD said | Settled |
|---|---|---|---|
| Route | `GET /core-asks/{id}` | `GET /api/v1/asks/{id}` | `GET /api/v1/core-asks/{id}` - same prefix and collection name as ADR-DRT-API-001 for the create operation |
| Caller without permission | 403 (Assumption 1) | Not stated | **403** - the spec is explicit and the Master LLD is silent, so there is nothing to reconcile |

## Contract summary

| Field | Value |
|---|---|
| Path parameter | `id` - string, format uuid, required. The askId |
| Query parameters | None |
| Request body | None |
| Response schema | CoreAskResponse: askId, askDetailId, version, statusId, coreAskDetails, createdOn, lastModifiedOn |
| Success status | **200 OK** |
| Idempotency | Not applicable - read-only |
| Concurrency | Not applicable - read-only |

## Response fields - CoreAskResponse

| Field | Type | Format | Required | Constraints | Description |
|---|---|---|---|---|---|
| askId | string | uuid | yes | | Identifier of the ASK |
| askDetailId | string | uuid | yes | | Identifier of the current version's detail record |
| version | integer | int32 | yes | minimum 1 | Current version number |
| statusId | integer | int32 | yes | | Current workflow status: 123, 127 or 145 |
| coreAskDetails | object | | yes | | Business data of the current version (fields below) |
| createdOn | string | date-time | yes | | When the ASK was created, ISO 8601 UTC |
| lastModifiedOn | string | date-time | yes | | When the ASK was last changed, ISO 8601 UTC |

## Response fields - coreAskDetails

Same fields and types as the create operation's coreAskDetails (ASK-POST-CORE-ASKS).

| Field | Type | Format | Required | Constraints |
|---|---|---|---|---|
| coreAskName | string | | yes | maxLength 200 |
| dppGroupId | string | uuid | yes | |
| needReasonId | string | uuid | yes | |
| generalSpecialityNeedId | string | uuid | yes | |
| generalSpecialityNeedComment | string | | no | |
| levelNeedId | string | uuid | yes | |
| pml | string | | no | maxLength 99 |
| projectedStartDate | string | date-time | yes | |
| endDate | string | date-time | no | |
| outgoingResource | string | | no | |
| employeeId | string | uuid | no | |
| headCountAmount | integer | int32 | yes | |
| fteAmount | number | double | yes | |
| rolePostingId | string | uuid | no | |
| numberofresources | integer | int32 | no | |
| titlingCategory | string | | yes | |
| transitionalCoach | string | | no | maxLength 99 |
| roleSummary | string | | yes | |
| roleResponsibility | string | | yes | |
| roleQualification | string | | yes | |

## Authorization

| Control | Value |
|---|---|
| Authentication | SEC 001 - Entra ID, OAuth 2.0 Authorization Code with PKCE |
| Permission | View ASK, scoped to the record's business unit (SEC 002) |
| Record scope | Route ID is not authorization evidence (SEC 003) |
| Audit | Not required for reads |

## Processing flow

1. Validate that id is a GUID; otherwise return 400.
2. Resolve the current DRT actor and check View ASK for the record's business unit; otherwise return 403.
3. Load the ASK and its current version; if none exists, return 404.
4. Return 200 with CoreAskResponse.

## Errors and tests

| Area | Content |
|---|---|
| Errors | 400 validation_failed - id is not a GUID; 401 unauthenticated; 403 access_denied - no View ASK permission; 404 not_found - no ASK with that id; 503 dependency_unavailable. Every non-2xx returns ProblemDetails |
| Tests | Existing id gives 200 with all fields; unknown id gives 404; non-GUID id gives 400; caller without permission gives 403; unauthenticated gives 401; a GET changes no data |

## Open items - none blocking

| ID | Item | Owner | Decision required |
|---|---|---|---|
| OI-1 | Whether a caller without permission should get 404 instead of 403 to hide the record's existence | Security | Confirm 403 or change to 404 |
