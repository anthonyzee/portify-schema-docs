# Integration Audit Log (`INTLOG`)

The **Integration Audit Log** captures **append-only, non-transactional events** generated during integration runs such as order imports, inventory syncs, and configuration changes.

This log is designed for:
- Observability
- Troubleshooting
- Compliance & traceability
- UI audit timelines

Each log entry represents **one immutable event**.

---

## Resource Metadata

| Field | Value |
|-----|------|
| Resource Name | `integration_log` |
| Schema Name | `INTLOG` |
| Entity Type | `SCHEMA::default::integration_log` |
| Storage | DynamoDB |
| Nature | Append-only (non-transactional) |

---

## Key Design

| Key | Purpose |
|----|--------|
| **Partition Key (PK)** | `audit_id` – unique audit log identifier |
| **Sort Key (PK)** | `created_at` – chronological ordering |
| **Tenant Scope** | `pfy_cus_id` (implicit, multi-tenant isolation) |

---

## Attributes

### `audit_id`
- **Type:** `string`
- **Required:** Yes
- **Key:** PK
- **Generator:** `RUNNING_NO`
- **Source:** `COUNTER::integration_log`
- **Description:** Primary audit log identifier. Auto-generated running number ensuring uniqueness across audit entries.
- **Example:** `AUD-000001`

---

### `created_at`
- **Type:** `string`
- **Required:** Yes
- **Key:** PK
- **Description:** ISO-8601 timestamp when the audit event occurred. Used for time-based sorting and querying.
- **Example:** `2026-01-01T10:15:30.123Z`

---

### `session_id`
- **Type:** `string`
- **Description:** Logical execution or run identifier. Groups multiple audit logs belonging to the same integration run.
- **Example:** `RUN#20260101-1015`

---

### `request_id`
- **Type:** `string`
- **Description:** Correlation ID for tracing across API Gateway, Lambda, queues, and processors.
- **Example:** `req_8f9a2c`

---

### `integration_instance`
- **Type:** `string`
- **Description:** Identifier of the integration instance being executed (e.g. WooCommerce store, Shopify connector).
- **Example:** `INTINST#WOO#0001`

---

### `processor_instance`
- **Type:** `string`
- **Description:** Identifier of the processor instance that produced the log. Useful for debugging parallel or distributed executions.
- **Example:** `INTPROC#000123`

---

### `status`
- **Type:** `string`
- **Description:** Execution outcome of the event.
- **Valid Values:** `success`, `info`, `warning`, `error`
- **Example:** `success`

---

### `type`
- **Type:** `string`
- **Description:** High-level classification of the audit entry.
- **Valid Values:** `System`, `Status`, `Warning`, `Security`
- **Example:** `System`

---

### `action`
- **Type:** `string`
- **Description:** Business or system action being performed.
- **Common Values:** `Order Import`, `Inventory Sync`, `Customer Sync`, `Shipping Update`, `Run`, `Configured`, `Test`
- **Example:** `Order Import`

---

### `actor_type`
- **Type:** `string`
- **Description:** Source responsible for triggering the action.
- **Valid Values:** `system`, `user`, `processor`
- **Example:** `processor`

---

### `actor_id`
- **Type:** `string`
- **Description:** Identifier of the actor performing the action.
- **Examples:** `system`, `user_123`, `INTPROC#000123`

---

### `description`
- **Type:** `string`
- **Description:** Human-readable explanation of the event. Displayed directly in UI audit timelines.
- **Example:** `Started Session Order Import`

---

### `details`
- **Type:** `string`
- **Description:** Additional diagnostic or contextual information. Typically a stringified JSON payload.
- **Example:** `{ "processed": 100, "failed": 2, "duration_ms": 8421 }`

---

### `resource_type`
- **Type:** `string`
- **Description:** Type of business entity affected by the action.
- **Common Values:** `order`, `customer`, `product`, `inventory`, `shipment`
- **Example:** `order`

---

### `resource_id`
- **Type:** `string`
- **Description:** Identifier of the affected business resource (internal or external).
- **Example:** `ORDER#80599`

---

### `ref1`
- **Type:** `string`
- **Description:** Primary business reference, commonly invoice number.
- **Example:** `INV-100234`

---

### `ref2`
- **Type:** `string`
- **Description:** Secondary business reference such as purchase order.
- **Example:** `PO-77881`

---

### `ref3`
- **Type:** `string`
- **Description:** Tertiary reference for extended traceability (e.g. SRN, shipment reference).
- **Example:** `SRN-00991`

---

### `pfy_store_name`
- **Type:** `string`
- **Key:** FK
- **Description:** Human-readable store or tenant display name for UI and reporting.
- **Example:** `Manuscribe Store`

---

## Sample Audit Log Item

```json
{
  "pfy_cus_id": "manuscribe-store",
  "audit_id": "AUD-000104",
  "created_at": "2026-01-02T09:17:32.551Z",
  "session_id": "RUN#20260102-0915",
  "request_id": "req_c11e88",
  "integration_instance": "INTINST#WOO#0001",
  "processor_instance": "INTPROC#000045",
  "status": "error",
  "type": "System",
  "action": "Customer Sync",
  "actor_type": "processor",
  "actor_id": "INTPROC#000045",
  "description": "Customer sync failed due to API authentication error",
  "resource_type": "customer",
  "resource_id": "CUS#88912",
  "details": "{\"error_code\":401,\"message\":\"Invalid API credentials\"}",
  "pfy_store_name": "Manuscribe Store"
}
```

---

## Design Principles

- Append-only: audit logs are never updated or deleted (except via TTL)
- High-volume safe: isolated from transactional tables
- UI-ready: enums + descriptions support schema-driven dashboards
- Traceable: session_id + request_id enable full execution replay

