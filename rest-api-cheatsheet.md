# 🌐 REST API Design & HTTP Status Codes Cheatsheet

A concise reference guide for designing production-ready RESTful APIs, HTTP verbs, status codes, error models, security headers, and pagination.

---

## 📌 HTTP Verbs & Idempotency

| Verb | Usage | Idempotent | Safe | Typical Response |
|------|-------|------------|------|------------------|
| `GET` | Retrieve resource representation | Yes | Yes | `200 OK` |
| `POST` | Create a new resource / trigger action | No | No | `201 Created` / `200 OK` |
| `PUT` | Replace existing resource completely | Yes | No | `200 OK` / `204 No Content` |
| `PATCH` | Partial update of existing resource | No | No | `200 OK` |
| `DELETE` | Delete target resource | Yes | No | `204 No Content` / `200 OK` |
| `HEAD` | Fetch response headers only | Yes | Yes | `200 OK` |
| `OPTIONS` | Inspect supported HTTP methods (CORS) | Yes | Yes | `204 No Content` |

---

## 🚦 HTTP Status Codes Quick Reference

### 2xx Success
- `200 OK`: Request succeeded. Default for `GET`, `PUT`, `PATCH`.
- `201 Created`: Resource successfully created. Include `Location` header with URI.
- `202 Accepted`: Request accepted for asynchronous background processing.
- `204 No Content`: Action succeeded, no content payload returned (e.g., `DELETE`).

### 3xx Redirection
- `301 Moved Permanently`: Permanent redirect; client should update URL.
- `302 Found`: Temporary redirect.
- `304 Not Modified`: Cached representation is still valid (Conditional `GET`).

### 4xx Client Errors
- `400 Bad Request`: Invalid payload format, syntax error, or validation failure.
- `401 Unauthorized`: Authentication missing or invalid token.
- `403 Forbidden`: Authenticated user lacks permission for action.
- `404 Not Found`: Target resource URI does not exist.
- `405 Method Not Allowed`: Resource exists, but HTTP method not supported.
- `409 Conflict`: Resource state conflict (e.g., duplicate email registration).
- `415 Unsupported Media Type`: Payload format not supported (e.g., non-JSON `Content-Type`).
- `422 Unprocessable Entity`: Valid JSON syntax, but domain/business validation failed.
- `429 Too Many Requests`: Rate limit exceeded.

### 5xx Server Errors
- `500 Internal Server Error`: Unhandled server runtime exception.
- `502 Bad Gateway`: Invalid response from upstream microservice/proxy.
- `503 Service Unavailable`: Server overloaded or under scheduled maintenance.
- `504 Gateway Timeout`: Upstream dependency timed out responding.

---

## 🛠️ URI Naming Best Practices

```http
# ✅ Preferred (Plural nouns, lowercase, kebab-case)
GET /api/v1/users
GET /api/v1/users/42
GET /api/v1/users/42/orders
POST /api/v1/users/42/orders

# ❌ Avoid (Verbs in URIs, trailing slashes, camelCase)
GET /api/v1/getUsers
POST /api/v1/create_user/
GET /api/v1/userOrders
```

---

## ⚠️ Standard Error Response Schema (RFC 7807)

```json
{
  "type": "https://api.example.com/errors/invalid-payment",
  "title": "Unprocessable Entity",
  "status": 422,
  "detail": "Insufficient funds in account balance.",
  "instance": "/api/v1/payments/pay_9823",
  "invalid_params": [
    {
      "name": "amount",
      "reason": "Amount exceeds maximum credit limit"
    }
  ]
}
```

---

## 📊 Pagination & Filtering Standards

### Cursor Pagination (Recommended for large dynamic datasets)
```http
GET /api/v1/events?limit=25&starting_after=evt_9012
```

### Offset Pagination
```http
GET /api/v1/products?page=2&per_page=50&sort=-created_at&status=active
```

---

## 🛡️ Essential Security & Rate Limit Headers

### Security Headers
```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Content-Security-Policy: default-src 'self'
```

### Rate Limiting Headers
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 985
X-RateLimit-Reset: 1672531199
Retry-After: 60
```
