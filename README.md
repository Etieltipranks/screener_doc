# Stock Screener API Documentation

## Overview

The Stock Screener API is a powerful tool for filtering and searching through stock data stored in MongoDB. It provides flexible filtering capabilities with support for various data types and operations, along with pagination and sorting functionality.

## Endpoint

```
POST /api/screener
```

## Request Format

### Headers
```
Content-Type: application/json
```

### Request Body Structure

```json
{
  "page": 1,
  "pageSize": 50,
  "sort": {
    "id": "fieldName",
    "dir": "asc|desc"
  },
  "filter": [
    {
      "id": "fieldName",
      "op": "operation",
      "value": "value"
    }
  ]
}
```

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | integer | No | 1 | Page number (minimum: 1) |
| `pageSize` | integer | No | 50 | Number of items per page (1-1000) |
| `sort` | object | No | `{"id": "_id", "dir": "desc"}` | Sorting configuration |
| `filter` | array | No | `[]` | Array of filter conditions |
| `project` | array | No | `[]` | Array of project conditions |

### Sort Object (optional)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Field name to sort by |
| `dir` | string | No | Sort direction: "asc" or "desc" (default: "asc") |

### Filter Object (optional)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Field name to filter on |
| `op` | string | Yes | Operation type (see operations below) |
| `value` | any | Yes | Value(s) for the operation |

### Project Object (optional)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Field name to filter on |
| `op` | string | Yes | Operation type (see operations below) |
| `value` | any | Yes | Value(s) for the operation |

## Supported Operations

| Operation | Description | Value Type | Example |
|-----------|-------------|------------|---------|
| `eq` | Equals | any | `{"id": "ticker", "op": "eq", "value": "AAPL"}` |
| `gte` | Greater than or equal | number | `{"id": "ebitda", "op": "gte", "value": 2168000000}` |
| `lte` | Less than or equal | number | `{"id": "revenueTtm", "op": "lte", "value": 13099000000}` |
| `gt` | Greater than | number | `{"id": "marketCap.daily.marketCap", "op": "gt", "value": 100000000}` |
| `lt` | Less than | number | `{"id": "pe", "op": "lt", "value": 20}` |
| `range` | Between (inclusive) | array | `{"id": "roe", "op": "range", "value": [50, 100]}` |
| `in` | In list | array | `{"id": "company.sector", "op": "in", "value": ["financial", "basicmaterials"]}` |
| `nin` | In list | array | `{"id": "company.sector", "op": "in", "value": ["technology", "basicmaterials"]}` |

## Response Format

```json
{
  "data": [...],
  "totalCount": 1500,
  "page": 1,
  "pageSize": 50,
  "totalPages": 30
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `data` | array | Array of stock documents matching the criteria |
| `totalCount` | integer | Total number of records matching the filters |
| `page` | integer | Current page number |
| `pageSize` | integer | Number of items per page |
| `totalPages` | integer | Total number of pages |

## Examples

### 1. Basic Usage - Get All Stocks (Paginated)

```bash
curl -X POST "{base_url}/api/screener" \
  -H "Content-Type: application/json" \
  -d '{
    "page": 1,
    "pageSize": 20,
    "sort": {
      "id": "_id",
      "dir": "desc"
    },
    "filter": [],
    "project": []
  }'
```

### 2. Filter by Price Range

```bash
curl -X POST "{base_url}/api/screener" \
  -H "Content-Type: application/json" \
  -d '{
    "page": 1,
    "pageSize": 10,
    "sort": {
      "id": "pe",
      "dir": "asc"
    },
    "filter": [
      {
        "id": "pe",
        "op": "range",
        "value": [35, 41]
      }
    ]
  }'
```

### 3. Filter by Market Cap and Sector

```bash
curl -X POST "{base_url}/api/screener" \
  -H "Content-Type: application/json" \
  -d '{
    "page": 1,
    "pageSize": 15,
    "sort": {
      "id": "marketCap.daily.marketCap",
      "dir": "desc"
    },
    "filter": [
      {
        "id": "marketCap.daily.marketCap",
        "op": "gte",
        "value": 100000000000
      },
      {
        "id": "company.sector",
        "op": "in",
        "value": ["consumerdefensive","technology"]
      }
    ]
  }'
```

### 4. Filter by Market Cap and Sector and Project by PE, EPS, FCF TTM

```bash
curl -X POST "{base_url}/api/screener" \
  -H "Content-Type: application/json" \
  -d '{
    "page": 1,
    "pageSize": 15,
    "sort": {
      "id": "marketCap.daily.marketCap",
      "dir": "desc"
    },
    "filter": [
      {
        "id": "marketCap.daily.marketCap",
        "op": "gte",
        "value": 100000000000
      },
      {
        "id": "company.sector",
        "op": "in",
        "value": ["consumerdefensive","technology"]
      }
    ],
    "project": [
      "fcfTtm",
      "eps",
      "pe"
    ]
  }'
```


## HTTP Status Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 400 | Bad Request (invalid JSON, missing required fields) |
| 500 | Internal Server Error |

## Error Response Format

```json
"Error message describing what went wrong"
```

## Notes

- All filters are combined with AND logic
- Case-sensitive string matching for equality operations
- Numeric values can be integers or decimals
- Empty filter arrays return all records (with pagination)
- Default sort is by `_id` in descending order
