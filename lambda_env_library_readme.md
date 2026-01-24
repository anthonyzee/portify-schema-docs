# DynamoDB & Lambda Environment Utilities

A small, production-ready utility library for **AWS Lambda**, **FastAPI local testing**, and **DynamoDB Local** usage.

This module standardizes:
- DynamoDB initialization (AWS vs Local)
- API Gateway–compatible response objects
- Float-to-Decimal conversion for DynamoDB safety
- Conditional debug logging

---

## 📥 Import Path

```python
from lib.lambda_env import (
    initialize_dynamodb,
    create_response_object,
    convert_floats_to_decimal,
    debug_message
)
```

---

## 📦 Installation

```bash
pip install boto3
```

Python 3.9+ recommended.

---

## 🔧 Environment Variables

| Variable | Description | Default |
|--------|-------------|---------|
| USE_LOCAL_DYNAMODB | Enable DynamoDB Local | false |
| AWS_DEFAULT_REGION | AWS region | us-west-2 |
| AWS_ACCESS_KEY_ID | AWS access key (local allowed) | dummy |
| AWS_SECRET_ACCESS_KEY | AWS secret key (local allowed) | dummy |
| DEBUG | Enable debug logging | false |

---

## Function Reference

---

### initialize_dynamodb()

Initializes and returns both a DynamoDB **client** and **resource**, automatically switching between **AWS DynamoDB** and **DynamoDB Local** depending on environment configuration.

#### Parameters
None

#### Returns
(boto3.client, boto3.resource)

Tuple containing:
- DynamoDB low-level client
- DynamoDB high-level resource

#### Modes

**AWS mode (USE_LOCAL_DYNAMODB=false)**
- Uses default AWS credential provider chain
- Connects to AWS DynamoDB

**Local mode (USE_LOCAL_DYNAMODB=true)**
- Endpoint: http://localhost:8000
- Region: AWS_DEFAULT_REGION (fallback us-west-2)
- Dummy credentials allowed

#### Example
```python
client, resource = initialize_dynamodb()
```

---

### create_response_object(status_code, body, enable_cors=True)

Creates an API Gateway / Lambda proxy compatible response object.

#### Parameters
status_code (int)  
HTTP status code

body (str)  
Response body string (usually json.dumps)

enable_cors (bool, default True)  
Attach permissive CORS headers

#### Returns
dict

```json
{
  "statusCode": 200,
  "headers": {...},
  "body": "..."
}
```

#### Example
```python
return create_response_object(
    200,
    json.dumps({"status": "ok"})
)
```

---

### convert_floats_to_decimal(obj)

Recursively converts all Python float values into Decimal for DynamoDB compatibility.

#### Parameters
obj (any)  
JSON-like structure (dict, list, nested)

#### Returns
any  
Same structure with float → Decimal(str(float))

#### Conversion Rules
- list → iterate
- dict → iterate values
- float → Decimal
- others → unchanged

#### Example
```python
payload = convert_floats_to_decimal(payload)
```

---

### debug_message(event)

Conditionally prints Lambda event and parsed request body when DEBUG=true.

#### Parameters
event (dict)  
AWS Lambda event payload

#### Environment Variable
DEBUG=true

#### Behavior
- Prints full event JSON
- Parses and prints event["body"]

No-op when DEBUG=false.

#### Example
```python
debug_message(event)
```

---

## Recommended Project Structure

```text
project/
├── lib/
│   ├── __init__.py
│   └── lambda_env.py
├── lambda_function.py
├── requirements.txt
└── README.md
```

---

## Best Practices

- Always run convert_floats_to_decimal before DynamoDB writes
- Keep DEBUG disabled in production
- Use DynamoDB Local for integration testing
- Standardize Lambda responses using create_response_object

---

## License

MIT
