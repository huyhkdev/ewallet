# REST API guidelines

- Base path `/api/v1`. Resources are plural nouns: `/wallets`, `/transfers`, `/topups`. `/me` for the current user's resources.
- JSON uses camelCase. Amounts are strings with currency: `{"amount": "12.50", "currency": "USD"}`.
- Timestamps are ISO-8601 in UTC with `Z`: `2026-10-08T07:56:27Z`. Store `timestamptz`.
- IDs exposed to clients are UUIDs (never sequential database ids).
- Status codes: `200` read/update, `201` + `Location` on create, `204` no body, `400` malformed, `401` not authenticated, `403` not allowed, `404` not found (also when the resource exists but belongs to someone else), `409` state conflict, `422` business rule violated, `429` rate limited, `5xx` our fault.
- Errors: RFC 9457 `application/problem+json` with a stable `code` (EW-020).
- Money-moving `POST` endpoints require `Idempotency-Key` (UUID). Semantics in ADR-0009.
- Pagination: cursor-based for statements and lists that grow forever (`?cursor=&size=`, response has `nextCursor`). Max `size` 100.
- Never break a published contract within `v1`: adding optional fields is fine; renaming or removing is a new version.
