# Integration Transform Library (Python)

A small, schema-driven transformation utility used in Portify/Manuscribe pipelines to:

- Read nested values from source JSON safely
- Resolve mapping expressions (path lookup, defaults, concatenation, literals)
- Merge values into a nested output dict using dot/bracket paths
- Support per-line mappings (`is_line=true` with `{{i}}` placeholders)
- Execute controlled operator scripts (`run_exec`) to compute derived fields
- Parse AWS API Gateway Lambda event bodies safely

---

## Constants

### `SPECIAL_LITERALS: dict[str, str]`

Lookup table for reserved literal tokens used inside `{...}` during concatenation.

| Key | Value |
|---|---|
| `space` | `" "` |
| `comma` | `","` |
| `hash` | `"#"` |

**Example**

```text
"SKU{hash}&sku" → "SKU#ABC123"
```

---

## Function Reference

---

## `get_nested_value(data, path_with_default)`

Safely retrieves a nested value from dictionaries/lists using dot and bracket notation.

### Parameters
- **data** (`dict | list`)  
  Source JSON object.
- **path_with_default** (`str | None`)  
  Examples:
  - `a.b.c`
  - `items[0].sku`
  - `items[0].sku|UNKNOWN`

### Returns
- Resolved value if found
- Fallback value after `|` if lookup fails
- `None` if not found and no fallback is provided

### Examples

```python
get_nested_value({"a": {"b": 1}}, "a.b")            # 1
get_nested_value({"a": {}}, "a.b|0")                 # "0"
get_nested_value({"items": [{"sku": "X"}]}, "items[0].sku")
get_nested_value({"items": []}, "items[0].sku|NA")
```

---

## `resolve_target_value(integration_source, raw_value)`

Resolves mapping expressions into final values.

### Supported Expression Types

#### 1. Pass-through
Non-string values (`int`, `float`, `bool`, `dict`, `list`) are returned as-is.

#### 2. Forced literal string (`=` prefix)

```python
"=00123" → "00123"
```

#### 3. Concatenation (`&`)

- Parts wrapped in `{}` resolve from `SPECIAL_LITERALS`
- Other parts are treated as source paths

```python
"SKU{hash}&sku" → "SKU#ABC123"
```

#### 4. Direct path lookup

Falls back to `get_nested_value(integration_source, raw_value)`.

### Parameters
- **integration_source** (`dict`)
- **raw_value** (`Any`)

### Returns
- Resolved value

---

## `merge_nested_dict(base, path, value)`

Writes a value into a nested structure using dot and bracket paths.

### Parameters
- **base** (`dict`)  
  Target output object (mutated in-place).
- **path** (`str`)  
  Example: `variants[0].sku`
- **value** (`Any`)

### Behavior
- Automatically creates missing dicts/lists
- Extends lists as required

### Example

```python
out = {}
merge_nested_dict(out, "variants[0].sku", "ABC")
merge_nested_dict(out, "variants[0].price", 9.99)

# Result:
# {"variants": [{"sku": "ABC", "price": 9.99}]}
```

---

## `process_line_mapping(final_output, integration_source, item, is_exec=False)`

Processes per-line mappings or exec scripts using `{{i}}` placeholders.

### Parameters
- **final_output** (`dict`)
- **integration_source** (`dict`)
- **item** (`dict`)  
  Mapping/operator definition
- **is_exec** (`bool`, default `False`)

### Required Fields

**Mapping mode (`is_exec=False`)**
- `line_field`
- `target_map_path`
- `target_value`

**Exec mode (`is_exec=True`)**
- `line_field`
- `target_map_path`
- `run_exec`

### Exec Environment
- Locals: `integration_source`, `i`, `result`
- Globals: `re`, `datetime`, `timedelta`
- `__builtins__` disabled

---

## `apply_run_execs_from_items(final_output, integration_source, operator_items)`

Executes operator scripts to compute derived fields.

### Parameters
- **final_output** (`dict`)
- **integration_source** (`dict`)
- **operator_items** (`list[dict]`)

### Operator Types
- Non-line operators (`is_line=false`)
- Line operators (`is_line=true`)

---

## `transform_item(integration_source, mapping_items, operator_items=None)`

Core reusable transformation entry point.

### Parameters
- **integration_source** (`dict`)  
  Source integration JSON
- **mapping_items** (`list[dict]`)  
  Mapping definitions
- **operator_items** (`list[dict] | None`)  
  Optional operator definitions

### Returns
```python
(final_output: dict, target_ecommerce_platform: str | None)
```

### Example

```python
final_output, platform = transform_item(
    integration_source=woo_product,
    mapping_items=mappings,
    operator_items=operators
)
```

---

## `_parse_body(event)`

Safely parses API Gateway Lambda request bodies.

### Parameters
- **event** (`dict`)

### Returns
- `dict` if JSON body is valid
- `None` if missing or invalid

### Example

```python
payload = _parse_body(event)
if payload is None:
    return {"statusCode": 400, "body": "Invalid JSON"}
```

---

## Mapping Specification Examples

### Simple Mapping

```python
{
  "is_active": True,
  "is_line": False,
  "target_map_path": "title",
  "target_value": "name"
}
```

### Fallback Default

```python
{
  "is_active": True,
  "is_line": False,
  "target_map_path": "vendor",
  "target_value": "store.name|Unknown Vendor"
}
```

### Concatenation

```python
{
  "is_active": True,
  "is_line": False,
  "target_map_path": "handle",
  "target_value": "name&{hash}&id"
}
```

### Line Mapping (`{{i}}`)

```python
{
  "is_active": True,
  "is_line": True,
  "line_field": "variations",
  "target_map_path": "variants[{{i}}].sku",
  "target_value": "variations[{{i}}].sku"
}
```

### Operator Exec

```python
{
  "is_active": True,
  "is_line": False,
  "target_map_path": "meta.slug",
  "run_exec": "result = re.sub(r'\\s+', '-', integration_source.get('name','').lower())"
}
```

---

## Security Notes

- `exec` is sandboxed with empty `__builtins__`
- Only limited modules are exposed
- Operator definitions should be tenant-controlled and validated

---

## License

Internal Manuscribe / Portify utility library.

