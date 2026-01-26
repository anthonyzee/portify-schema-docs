# PFY Schema Dynamo Helpers (Python)

Schema-driven helper functions for **PFY_TENANT_DATA** to:

- Fetch resource schemas (stored as `SCHEMA::{schema}::{table}`)
- Build consistent keys (FK “fix key” + PK-based sort key)
- Read a single item by schema-derived key
- List items using **OData filter syntax** via a shared adapter
- Generate running numbers via atomic DynamoDB counters

---

## Suggested repository name

**pfy-schema-dynamo-helpers**  
Alternatives:
- tenant-data-schema-kit  
- schema-keys-odata-dynamo  
- pfy-tenantdata-toolkit

---

## Why this exists

When resources are schema-driven and stored in DynamoDB:

- Key logic stays consistent and centralized
- CRUD APIs become generic and reusable
- Query behavior is predictable and tenant-safe
- OData-style filtering works without sacrificing DynamoDB performance patterns

---

## Requirements

- Python 3.10+
- boto3
- DynamoDB Table resource (`tenant_data_table`)
- Shared adapter:
  - `lib.dynamodb_odata_adapter.execute_dynamodb_query_from_odata`

---

## Data model assumptions

### Schema records

Stored in the same table (`PFY_TENANT_DATA`):

- `pfy_cus_id = "_DB"`
- `pfy_entity_type = "SCHEMA::{schema_name}::{table_name}"`
- `columns` stored as JSON string

### Tenant data records

- `pfy_cus_id = "<tenant_id>"`
- `pfy_entity_type = "<SORT_KEY>"`

### Running number counters

- `pfy_cus_id = "<tenant_id or _DB>"`
- `pfy_entity_type = "COUNTER::<entity>"`
- Attribute: `current_value` (number)

---

## API Reference

### get_schema_for_resource(...)

Fetches schema metadata and parses the `columns` JSON into a Python list.

Raises:
- Schema not found
- Invalid JSON in `columns`

---

### build_fix_key_from_schema(...)

Builds a fixed FK-based key using:

`tenant_id::fk_value_1::fk_value_2`

Ordered by column `index`.

---

### build_sort_key_from_schema(...)

Builds a PK-based sort key:

`<SCHEMA_PREFIX>::pk1::pk2`

Supports partial key generation when `ignore_missing=True`.

---

### get_single_item_from_schema(...)

Fetches a single DynamoDB item using schema-derived PK + SK.

Returns:
- Item dict or `None`

---

### list_items_from_schema(...)

Lists items using OData filter syntax.

Automatically:
- Extracts PK filters
- Builds exact or prefix sort key conditions
- Applies remaining filters via adapter
- Supports `$top`, `$skip`, `$inlinecount`, `$select`

---

### get_next_running_no(...)

Atomically increments a DynamoDB counter using `ADD`.

Returns:
- Updated `current_value`

---

## Recommended structure

```
lib/
  pfy_schema_dynamo_helpers.py
  dynamodb_odata_adapter.py
```

---

## License

Internal / private use.
