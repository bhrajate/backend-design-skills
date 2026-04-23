# API Design Principles — Reference

---

## REST API Golden Rules

### 1. Resource-Oriented URLs
URLs represent nouns (resources), not verbs (actions).

```
✅ Good:
GET    /orders           → list orders
GET    /orders/123       → get order 123
POST   /orders           → create order
PUT    /orders/123       → replace order 123
PATCH  /orders/123       → partial update
DELETE /orders/123       → delete order

❌ Bad (RPC-style):
POST /getOrder
POST /createOrder
POST /deleteOrder
```

### 2. Use HTTP Methods Correctly
| Method | Idempotent? | Safe? | Use for |
|--------|-------------|-------|---------|
| GET | Yes | Yes | Read |
| POST | No | No | Create |
| PUT | Yes | No | Replace |
| PATCH | No | No | Partial update |
| DELETE | Yes | No | Delete |

### 3. Meaningful HTTP Status Codes
```
200 OK            → successful GET/PATCH
201 Created       → successful POST (include Location header)
204 No Content    → successful DELETE
400 Bad Request   → client sent invalid data
401 Unauthorized  → not authenticated
403 Forbidden     → authenticated but not authorized
404 Not Found     → resource doesn't exist
409 Conflict      → state conflict (e.g. duplicate)
422 Unprocessable → validation failed
429 Too Many Requests → rate limited
500 Internal Error → server bug
```

### 4. Consistent Error Response Format
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": [
      { "field": "email", "issue": "Invalid email format" }
    ]
  }
}
```

### 5. Versioning
- URL versioning: `/v1/orders` (most common, explicit)
- Header versioning: `Accept: application/vnd.api+json;version=2`
- Never break existing clients — add fields, don't remove

### 6. Pagination
```json
GET /orders?page=2&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "has_next": true
  }
}
```
Or use cursor-based pagination for large/real-time datasets.

### 7. Filtering, Sorting, Fields
```
GET /orders?status=pending&sort=-created_at&fields=id,status,total
```

---

## API Security Checklist

- [ ] Authentication on all non-public endpoints (JWT, OAuth2)
- [ ] Authorization checks (user can only see their own data)
- [ ] Rate limiting per user/IP
- [ ] Input validation and sanitization
- [ ] HTTPS only
- [ ] No sensitive data in URLs (use body or headers)
- [ ] CORS configured correctly
- [ ] Audit logging for sensitive operations

---

## gRPC vs REST

| | REST | gRPC |
|--|------|------|
| Protocol | HTTP/1.1 + JSON | HTTP/2 + Protobuf |
| Performance | Good | Excellent |
| Browser support | Native | Needs proxy |
| Contract | OpenAPI (optional) | .proto (required) |
| Streaming | Limited | Native |
| Use case | Public APIs, simple services | Internal microservices |

---

## API Design Smell List

| Smell | Example | Fix |
|-------|---------|-----|
| Verb in URL | `POST /createOrder` | `POST /orders` |
| Exposing DB structure | `/user_accounts/123` | `/users/123` |
| Inconsistent naming | `userId` vs `user_id` vs `UserID` | Pick one convention |
| Returning 200 for errors | `200 OK { "error": "not found" }` | Use proper status codes |
| No pagination | Returns 100k records | Add pagination |
| Breaking changes | Remove field from response | Version the API |