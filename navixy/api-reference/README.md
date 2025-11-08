# Navixy SQL Execution API Reference

This document provides comprehensive API reference for the Navixy SQL execution endpoint used by the dashboard format.

## Overview

The SQL Execution API provides a secure, multi-tenant interface for executing SQL queries with prepared statements, parameter binding, and comprehensive safety controls.

## Base URL

```
POST /api/v1/sql/run
```

## Authentication

All requests require authentication via API token or session cookie.

### Headers
```
Authorization: Bearer <api_token>
Content-Type: application/json
```

## Request Format

### Request Body

```json
{
  "dialect": "postgresql",
  "statement": "SELECT event_type AS category, COUNT(*)::int AS value FROM events WHERE tenant_id = ${tenant_id} AND ts >= ${__from} AND ts < ${__to} GROUP BY 1 ORDER BY 2 DESC LIMIT ${limit}",
  "params": {
    "tenant_id": "8a5e4d6c-1234-5678-9abc-def012345678",
    "from": "2025-01-24T10:00:00Z",
    "to": "2025-01-25T10:00:00Z",
    "limit": 20
  },
  "limits": {
    "timeout_ms": 8000,
    "max_rows": 5000
  },
  "read_only": true
}
```

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `dialect` | string | Yes | SQL dialect (currently only "postgresql") |
| `statement` | string | Yes | SQL query with named parameters |
| `params` | object | Yes | Parameter values keyed by parameter name |
| `limits` | object | No | Execution limits |
| `read_only` | boolean | No | Enforce read-only queries (default: true) |

#### Limits Object

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `timeout_ms` | integer | 10000 | Maximum execution time in milliseconds |
| `max_rows` | integer | 10000 | Maximum rows to return |

## Response Format

### Success Response

```json
{
  "columns": [
    { "name": "category", "type": "string" },
    { "name": "value", "type": "number" }
  ],
  "rows": [
    ["Harsh Braking", 312],
    ["Speeding", 271],
    ["Idling", 156]
  ],
  "stats": {
    "row_count": 3,
    "elapsed_ms": 128,
    "query_hash": "a1b2c3d4e5f6..."
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `columns` | array | Column metadata |
| `rows` | array | Query result rows |
| `stats` | object | Execution statistics |

#### Columns Array

Each column object contains:
- `name`: Column name
- `type`: Data type (string, number, boolean, timestamp, json)

#### Stats Object

| Field | Type | Description |
|-------|------|-------------|
| `row_count` | integer | Number of rows returned |
| `elapsed_ms` | integer | Query execution time |
| `query_hash` | string | Hash of the executed query |

## Error Responses

### SQL Error

```json
{
  "error": {
    "type": "sql_error",
    "code": "INVALID_SYNTAX",
    "message": "Syntax error in SQL statement",
    "error_id": "err_12345",
    "details": {
      "position": 45,
      "hint": "Check your SQL syntax"
    }
  }
}
```

### Validation Error

```json
{
  "error": {
    "type": "validation_error",
    "code": "INVALID_PARAMETER",
    "message": "Parameter 'limit' must be between 1 and 1000",
    "error_id": "err_12346",
    "details": {
      "parameter": "limit",
      "value": 2000,
      "constraints": { "min": 1, "max": 1000 }
    }
  }
}
```

### Security Error

```json
{
  "error": {
    "type": "security_error",
    "code": "FORBIDDEN_OPERATION",
    "message": "DML operations are not allowed",
    "error_id": "err_12347",
    "details": {
      "operation": "INSERT",
      "reason": "read_only_mode"
    }
  }
}
```

### Rate Limit Error

```json
{
  "error": {
    "type": "rate_limit_error",
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "error_id": "err_12348",
    "details": {
      "limit": 100,
      "window": "1m",
      "retry_after": 30
    }
  }
}
```

## Error Codes

### SQL Errors
- `INVALID_SYNTAX` - SQL syntax error
- `INVALID_COLUMN` - Reference to non-existent column
- `INVALID_TABLE` - Reference to non-existent table
- `INVALID_SCHEMA` - Reference to non-allowed schema

### Validation Errors
- `INVALID_PARAMETER` - Parameter validation failed
- `MISSING_PARAMETER` - Required parameter not provided
- `INVALID_TYPE` - Parameter type mismatch

### Security Errors
- `FORBIDDEN_OPERATION` - Operation not allowed
- `SCHEMA_ACCESS_DENIED` - Access to schema denied
- `TENANT_MISMATCH` - Tenant ID mismatch

### Resource Errors
- `TIMEOUT_EXCEEDED` - Query execution timeout
- `ROW_LIMIT_EXCEEDED` - Too many rows returned
- `MEMORY_LIMIT_EXCEEDED` - Memory limit exceeded

### Rate Limiting
- `RATE_LIMIT_EXCEEDED` - Request rate limit exceeded
- `QUOTA_EXCEEDED` - Daily quota exceeded

## Parameter Types

### Supported Types

| Type | Description | Example |
|------|-------------|---------|
| `uuid` | UUID string | `"8a5e4d6c-1234-5678-9abc-def012345678"` |
| `int` | Integer | `42` |
| `numeric` | Decimal number | `3.14159` |
| `text` | String | `"sample text"` |
| `timestamptz` | Timestamp with timezone | `"2025-01-24T10:00:00Z"` |
| `bool` | Boolean | `true` |
| `json` | JSON object/array | `{"key": "value"}` |
| `text[]` | Array of strings | `["item1", "item2"]` |
| `uuid[]` | Array of UUIDs | `["uuid1", "uuid2"]` |

### Type Conversion

The API automatically converts parameter values to the specified SQL types:

- **Time formats**: Supports epoch milliseconds and ISO 8601 strings
- **Arrays**: Converts JSON arrays to PostgreSQL array types
- **Numbers**: Handles string-to-number conversion
- **Booleans**: Converts string representations ("true"/"false") to boolean

## Security Features

### Prepared Statements
All queries use prepared statements with parameter binding to prevent SQL injection.

### Read-Only Mode
When `read_only: true` (default), the API rejects:
- INSERT, UPDATE, DELETE statements
- DDL operations (CREATE, DROP, ALTER)
- Stored procedure calls
- COPY operations

### Schema Isolation
- Queries are restricted to allowed schemas
- Automatic `SET LOCAL ROLE` and `SET LOCAL SEARCH_PATH`
- Tenant isolation enforced at the database level

### Audit Logging
Every query execution is logged with:
- Dashboard UID and panel ID
- Query hash and parameters
- Execution time and row count
- User and tenant information

## Rate Limiting

### Limits
- **Per User**: 100 requests per minute
- **Per Dashboard**: 50 requests per minute
- **Per Panel**: 20 requests per minute

### Headers
Rate limit information is included in response headers:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1643123456
```

## Caching

### Cache Keys
Cache keys are generated from:
- Dashboard UID
- Panel ID
- Parameter hash
- Schema version

### TTL
- **Live data**: 30 seconds
- **Historical data**: 5 minutes
- **Static data**: 1 hour

## Examples

### Simple Count Query

```bash
curl -X POST /api/v1/sql/run \
  -H "Authorization: Bearer your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "dialect": "postgresql",
    "statement": "SELECT COUNT(*)::int AS value FROM events WHERE tenant_id = ${tenant_id}",
    "params": {
      "tenant_id": "8a5e4d6c-1234-5678-9abc-def012345678"
    }
  }'
```

### Time Series Query

```bash
curl -X POST /api/v1/sql/run \
  -H "Authorization: Bearer your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "dialect": "postgresql",
    "statement": "SELECT DATE_TRUNC(\"hour\", ts) AS time, COUNT(*)::int AS value FROM events WHERE tenant_id = ${tenant_id} AND ts BETWEEN ${__from} AND ${__to} GROUP BY 1 ORDER BY 1",
    "params": {
      "tenant_id": "8a5e4d6c-1234-5678-9abc-def012345678",
      "from": "2025-01-24T00:00:00Z",
      "to": "2025-01-25T00:00:00Z"
    },
    "limits": {
      "timeout_ms": 5000,
      "max_rows": 1000
    }
  }'
```

### Parameterized Query with Limits

```bash
curl -X POST /api/v1/sql/run \
  -H "Authorization: Bearer your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "dialect": "postgresql",
    "statement": "SELECT category, COUNT(*)::int AS value FROM events WHERE tenant_id = ${tenant_id} AND category IN (${categories}) GROUP BY 1 LIMIT ${limit}",
    "params": {
      "tenant_id": "8a5e4d6c-1234-5678-9abc-def012345678",
      "categories": ["alarm", "warning", "info"],
      "limit": 10
    },
    "limits": {
      "timeout_ms": 3000,
      "max_rows": 100
    }
  }'
```

## SDK Examples

### JavaScript/TypeScript

```typescript
interface SQLRequest {
  dialect: string;
  statement: string;
  params: Record<string, any>;
  limits?: {
    timeout_ms?: number;
    max_rows?: number;
  };
  read_only?: boolean;
}

interface SQLResponse {
  columns: Array<{ name: string; type: string }>;
  rows: any[][];
  stats: {
    row_count: number;
    elapsed_ms: number;
    query_hash: string;
  };
}

async function executeSQL(request: SQLRequest): Promise<SQLResponse> {
  const response = await fetch('/api/v1/sql/run', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${apiToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(request)
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(`SQL execution failed: ${error.error.message}`);
  }

  return response.json();
}
```

### Python

```python
import requests
from typing import Dict, Any, List

def execute_sql(
    statement: str,
    params: Dict[str, Any],
    timeout_ms: int = 10000,
    max_rows: int = 10000
) -> Dict[str, Any]:
    url = "https://api.example.com/api/v1/sql/run"
    headers = {
        "Authorization": f"Bearer {api_token}",
        "Content-Type": "application/json"
    }
    
    payload = {
        "dialect": "postgresql",
        "statement": statement,
        "params": params,
        "limits": {
            "timeout_ms": timeout_ms,
            "max_rows": max_rows
        },
        "read_only": True
    }
    
    response = requests.post(url, json=payload, headers=headers)
    response.raise_for_status()
    
    return response.json()
```

## Best Practices

### Query Optimization
- Use appropriate indexes
- Limit result sets with WHERE clauses
- Use LIMIT for large datasets
- Avoid SELECT * when possible

### Parameter Binding
- Always use named parameters
- Validate parameter types
- Set appropriate limits
- Use prepared statements

### Error Handling
- Handle all error types gracefully
- Log errors for debugging
- Provide user-friendly error messages
- Implement retry logic for transient errors

### Performance
- Use appropriate timeouts
- Implement caching where possible
- Monitor query performance
- Optimize for common use cases
