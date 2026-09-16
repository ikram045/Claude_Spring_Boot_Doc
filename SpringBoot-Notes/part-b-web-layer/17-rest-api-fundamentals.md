[← Back to Contents](../README.md) · Part B — Web Layer (Spring MVC / REST)

---

# 17. REST API Fundamentals

## 17.1 What is REST?

**Simple words:**
REST is a **set of rules for designing web APIs** so that different systems can talk to each other in a predictable way. It says: treat everything as a *resource* (an order, a customer), give each resource a URL, and use standard HTTP methods to act on it.

**Technical wording:**
REST (REpresentational State Transfer) is an architectural style defining constraints for distributed hypermedia systems: client-server separation, statelessness, cacheability, a uniform interface, layered system, and optionally code-on-demand.

## 17.2 The REST constraints that matter daily

| Constraint | What it means in practice |
|---|---|
| **Client–Server** | Frontend and backend evolve independently |
| **Stateless** | **Every request carries everything needed.** The server keeps no session between requests. This is what allows horizontal scaling — any instance can serve any request. |
| **Cacheable** | Responses declare whether they can be cached |
| **Uniform Interface** | Resources have URIs; standard methods act on them |
| **Layered System** | Client can't tell if it's talking to the server or a load balancer/gateway |

**Statelessness is the one to emphasise to a TL:** it is why JWT (self-contained token) suits microservices better than server-side sessions — no sticky sessions, no shared session store required, and any pod can handle any request.

## 17.3 HTTP methods

| Method | Purpose | Idempotent? | Safe? | Request body | Typical success |
|---|---|---|---|---|---|
| **GET** | Read a resource | Yes | Yes | No | 200 OK |
| **POST** | Create a resource | **No** | No | Yes | 201 Created |
| **PUT** | Full replace/update | **Yes** | No | Yes | 200 OK / 204 |
| **PATCH** | Partial update | No (usually) | No | Yes | 200 OK |
| **DELETE** | Remove a resource | **Yes** | No | No | 204 No Content |

**Definitions you must get right in interviews:**
- **Safe** = does not modify server state. Only GET (and HEAD/OPTIONS).
- **Idempotent** = calling it N times has the **same effect** as calling it once. GET, PUT, DELETE are idempotent; POST is not.

**Why idempotency matters in production:** if a client times out and retries, an idempotent call is harmless. A POST retry can create a **duplicate order**. This is why payment APIs require an **idempotency key** header — a real, practical concern worth raising with a TL.

**PUT vs PATCH — the classic question:**

```http
PUT /api/orders/1          →  replaces the ENTIRE resource
{ "customerId": "C1", "amount": 500, "status": "CONFIRMED" }
   Any field you omit is set to null/default.

PATCH /api/orders/1        →  updates ONLY the given fields
{ "status": "SHIPPED" }
   Everything else is left untouched.
```

## 17.4 URL design rules

| Rule | Bad | Good |
|---|---|---|
| Use **nouns**, not verbs | `/getOrder`, `/createOrder` | `/orders` |
| Use **plural** resource names | `/order/1` | `/orders/1` |
| The HTTP method is the verb | `POST /createOrder` | `POST /orders` |
| Hierarchy for relations | `/getOrderItems?orderId=1` | `/orders/1/items` |
| lowercase with hyphens | `/orderItems`, `/order_items` | `/order-items` |
| Version the API | `/orders` | `/api/v1/orders` |
| Filters as query params | `/orders/status/pending` | `/orders?status=PENDING` |

**A complete, well-designed resource:**

```
GET    /api/v1/orders                 list orders (paginated, filterable)
GET    /api/v1/orders/{id}            get one order
POST   /api/v1/orders                 create an order
PUT    /api/v1/orders/{id}            replace an order
PATCH  /api/v1/orders/{id}            update part of an order
DELETE /api/v1/orders/{id}            delete an order
GET    /api/v1/orders/{id}/items      items of that order (sub-resource)
GET    /api/v1/orders?status=PENDING&page=0&size=20&sort=createdAt,desc
```

## 17.5 HTTP status codes (know these cold)

### 2xx — Success
| Code | Name | Use when |
|---|---|---|
| **200** | OK | Successful GET / PUT / PATCH |
| **201** | Created | **POST created a resource** — include a `Location` header |
| **202** | Accepted | Async processing started, not yet complete |
| **204** | No Content | **DELETE succeeded**, nothing to return |

### 4xx — Client's fault
| Code | Name | Use when |
|---|---|---|
| **400** | Bad Request | Validation failed, malformed JSON |
| **401** | Unauthorized | **Not authenticated** — no/invalid token |
| **403** | Forbidden | **Authenticated but not allowed** |
| **404** | Not Found | Resource doesn't exist |
| **405** | Method Not Allowed | Wrong HTTP method for that URL |
| **409** | Conflict | Duplicate resource, optimistic-lock failure |
| **415** | Unsupported Media Type | Wrong `Content-Type` |
| **422** | Unprocessable Entity | Syntactically fine, semantically invalid |
| **429** | Too Many Requests | Rate limit exceeded |

### 5xx — Server's fault
| Code | Name | Use when |
|---|---|---|
| **500** | Internal Server Error | Unhandled exception — a bug |
| **502** | Bad Gateway | Upstream service returned garbage |
| **503** | Service Unavailable | Down / overloaded / circuit breaker open |
| **504** | Gateway Timeout | Upstream timed out |

**The single most important distinction — 401 vs 403:**
- **401 Unauthorized** = *"I don't know who you are."* Missing or invalid credentials. **Log in.**
- **403 Forbidden** = *"I know who you are, and you're not allowed."* Valid token, insufficient role. **Logging in again won't help.**

(The name "Unauthorized" for 401 is a historical misnomer — it actually means unauthenticated.)

**Second most important — 4xx vs 5xx:** 4xx means the client must change something. 5xx means **we** have a problem. Returning 500 for a validation failure is a common bug: it makes your error dashboards alert on the customer's typo.

## 17.6 HTTP headers worth knowing

| Header | Direction | Purpose |
|---|---|---|
| `Content-Type` | Both | Format of **this** message's body (`application/json`) |
| `Accept` | Request | Format the client **wants** back |
| `Authorization` | Request | `Bearer <jwt>` or `Basic <base64>` |
| `Location` | Response | URI of the newly created resource (with 201) |
| `Cache-Control` | Response | Caching policy |
| `ETag` / `If-None-Match` | Both | Conditional requests, optimistic concurrency |
| `X-Correlation-Id` | Both | Request tracing across microservices |

## 17.7 TL Questions

**Q: What makes an API RESTful?**
A: Resources identified by URIs, standard HTTP methods as the verbs, statelessness, appropriate status codes, and a uniform, predictable interface. In practice: nouns in URLs, correct methods, correct status codes, no server-side session state.

**Q: Why does statelessness matter for us?**
A: It allows horizontal scaling. Any instance can serve any request because no session is held in memory, so we can add pods behind a load balancer without sticky sessions or a shared session store. It's also the reason we use JWT rather than server sessions.

**Q: PUT or PATCH for our update endpoint?**
A: PUT if the client sends the complete resource and we replace it; PATCH if they send only changed fields. PUT is idempotent, which makes retries safe. We use PUT for full updates and PATCH for partial ones like a status change.

**Q: What status code should a POST return?**
A: 201 Created, with a `Location` header pointing at the new resource. 200 is acceptable but 201 is more precise and tells the client a resource was created.

**Q: What's the difference between 401 and 403?**
A: 401 means we don't know who you are — the token is missing or invalid. 403 means we know exactly who you are but your role doesn't permit this action. Logging in again fixes 401, not 403.

**Q: Why do we version the API?**
A: So we can make breaking changes without breaking existing clients. Mobile apps in particular can't be force-upgraded, so `/api/v1` and `/api/v2` have to coexist for a period.

**Q: What happens if the client retries a failed POST?**
A: It could create a duplicate, because POST isn't idempotent. For critical operations like payments we accept an idempotency key header and return the original result if the key repeats.

## 17.8 Interview Questions

**Beginner — What does REST stand for?**
Representational State Transfer.

**Beginner — Which methods are idempotent?**
GET, PUT, DELETE, HEAD, OPTIONS. POST is not; PATCH usually is not.

**Intermediate — Difference between safe and idempotent?**
Safe = no state change at all (GET). Idempotent = repeating it produces the same final state (PUT, DELETE). All safe methods are idempotent, but not vice versa — DELETE changes state yet is idempotent.

**Intermediate — Is DELETE really idempotent if the second call returns 404?**
Yes. Idempotency concerns the **server state**, not the status code. After the first DELETE the resource is gone; further DELETEs leave it gone. The differing response code doesn't break idempotency.

**Intermediate — REST vs SOAP?**
REST: HTTP-native, usually JSON, lightweight, stateless, no formal contract required. SOAP: XML envelope, strict WSDL contract, transport-independent, built-in WS-Security and standardised transactions. SOAP still appears in banking and legacy enterprise integration.

**Advanced — What is HATEOAS and do we use it?**
Hypermedia As The Engine Of Application State — responses include links to the next available actions, so a client discovers the API rather than hardcoding URLs. It is REST's highest maturity level (Richardson Level 3), but most corporate APIs stop at Level 2 (resources + verbs + status codes) because of the added complexity and limited client support.

**Advanced — How do you version an API?**
URI versioning (`/api/v1/orders`) — simplest and most visible, most common. Header versioning (`Accept: application/vnd.company.v1+json`) — cleaner URLs, purer REST, harder to test. Query param (`?version=1`) — easy but untidy. Most teams choose URI versioning for its clarity.

## 17.9 Quick Revision

1. REST = resources + URIs + standard HTTP methods + status codes + statelessness.
2. Nouns, plural, lowercase-hyphenated, versioned: `/api/v1/orders/{id}/items`.
3. GET read, POST create, PUT replace, PATCH partial, DELETE remove.
4. Idempotent: GET, PUT, DELETE. **Not** POST.
5. 200 OK, 201 Created (+`Location`), 204 No Content.
6. 400 validation, **401 not authenticated**, **403 not allowed**, 404 missing, 409 conflict.
7. 4xx = client's problem, 5xx = our problem. Never return 500 for validation.
8. Stateless → horizontal scaling → JWT over sessions.
9. Filters and paging go in query params, not the path.

## 17.10 TL Explanation (speak this)

> "We design the API around resources rather than actions — so `POST /api/v1/orders` instead of `/createOrder`, with the HTTP method carrying the verb. Everything is stateless, meaning each request carries its own JWT and no session lives on the server, which is what lets us scale horizontally without sticky sessions. We're careful with status codes: 201 with a `Location` header on create, 204 on delete, 400 for validation, and 401 versus 403 distinguished properly — 401 means we don't know who you are, 403 means you're known but not permitted. And we version the URL from day one so mobile clients don't break when we change contracts."

---

[← 16. HandlerMapping and HandlerAdapter](16-handlermapping-and-handleradapter.md) | [Contents](../README.md) | [18. Controllers →](18-controllers.md)
