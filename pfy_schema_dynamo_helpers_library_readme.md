# PFY Schema Dynamo Helpers (Python)

Schema-driven helper functions for **PFY_TENANT_DATA** that standardize how schemas, keys, queries, and counters are handled in a multi-tenant DynamoDB design.

This library is designed for **schema-first**, **tenant-safe**, and **generic CRUD** implementations.

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
- OData-style filtering works without sacrificing DynamoDB access patterns

---

## Requirements

- Python 3.10+
- boto3
- DynamoDB **Table resource**
- Shared adapter:
  - `lib.dynamodb_odata_adapter.execute_dynamodb_query_from_odata`

---

## Core concepts

### tenant_data_table

All functions accept **`tenant_data_table`**, which must be a **boto3 DynamoDB Table resource**:

```python
import boto3

dynamodb = boto3.resource("dynamodb")
tenant_data_table = dynamodb.Table("PFY_TENANT_DATA")
```

This table stores:
- Schema definitions
- Tenant data records
- Counters (RUNNING_NO)

---

## Data model assumptions

### Schema records

Stored in the same table:

| Attribute | Value |
|--------|------|
| pfy_cus_id | `_DB` |
| pfy_entity_type | `SCHEMA::{schema_name}::{table_name}` |
| columns | JSON string |

### Tenant data records

| Attribute | Description |
|--------|------------|
| pfy_cus_id | Tenant identifier |
| pfy_entity_type | Schema-derived sort key |

### Running number counters

| Attribute | Value |
|--------|------|
| pfy_cus_id | Tenant id or `_DB` |
| pfy_entity_type | `COUNTER::<entity>` |
| current_value | Number |

---

## API Reference

### get_schema_for_resource

```python
get_schema_for_resource(
    tenant_data_table,
    table_name: str,
    schema_name: str = "default"
) -> dict
```

**Parameters**

| Name | Type | Description |
|----|----|------------|
| tenant_data_table | DynamoDB Table | boto3 DynamoDB table resource |
| table_name | str | Logical resource / table name |
| schema_name | str | Schema namespace (default: `default`) |

**Returns**
- Schema dictionary with parsed `columns: list[dict]`

---

### build_fix_key_from_schema

```python
build_fix_key_from_schema(
    schema: dict,
    key_values: dict,
    tenant_id: str,
    ignore_missing: bool = False
) -> str
```

**Parameters**

| Name | Type | Description |
|----|----|------------|
| schema | dict | Schema metadata |
| key_values | dict | Mapping of FK column names to values |
| tenant_id | str | Tenant prefix |
| ignore_missing | bool | Skip missing FK fields if True |

**Returns**
- Fixed FK key string

---

### build_sort_key_from_schema

```python
build_sort_key_from_schema(
    schema: dict,
    key_values: dict,
    ignore_missing: bool = False
) -> str
```

**Parameters**

| Name | Type | Description |
|----|----|------------|
| schema | dict | Schema metadata |
| key_values | dict | Mapping of PK column names to values |
| ignore_missing | bool | Allow partial PK key generation |

**Returns**
- Sort key string

---

### get_single_item_from_schema

```python
get_single_item_from_schema(
    tenant_data_table,
    schema: dict,
    tenant_id: str,
    key_values: dict
) -> dict | None
```

**Parameters**

| Name | Type | Description |
|----|----|------------|
| tenant_data_table | DynamoDB Table | boto3 DynamoDB table resource |
| schema | dict | Parsed schema |
| tenant_id | str | Tenant partition key |
| key_values | dict | PK values used to build sort key |

**Returns**
- Item dict or `None`

---

### list_items_from_schema

```python
list_items_from_schema(
    tenant_data_table,
    schema: dict,
    tenant_id: str,
    filter_query: str | None = None,
    top: int = 500,
    skip: int = 0,
    inlinecount: bool = False,
    select: str | None = None
) -> list[dict]
```

**Parameters**

| Name | Type | Description |
|----|----|------------|
| tenant_data_table | DynamoDB Table | boto3 DynamoDB table resource |
| schema | dict | Parsed schema |
| tenant_id | str | Tenant partition key |
| filter_query | str | OData filter string |
| top | int | Max items |
| skip | int | Pagination offset |
| inlinecount | bool | Include total count |
| select | str | Comma-separated projection |

**Returns**
- List of items

---

### get_next_running_no

```python
get_next_running_no(
    tenant_data_table,
    pfy_cus_id: str,
    entity: str
) -> int
```

**Parameters**

| Name | Type | Description |
|----|----|------------|
| tenant_data_table | DynamoDB Table | boto3 DynamoDB table resource |
| pfy_cus_id | str | Tenant or `_DB` |
| entity | str | Counter name |

**Returns**
- Incremented running number

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
