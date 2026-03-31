# 5. Understanding HTTP for Backend Engineers — Where It All Starts

## 1. Core Principles of HTTP

HTTP (Hypertext Transfer Protocol) is an application-layer protocol (Layer 7 in the OSI model) used by clients and servers to communicate. It is built on two fundamental ideas:

- **Statelessness:** The server retains no memory of past interactions. Every request is entirely self-contained and must include all the necessary information (like authentication tokens or cookies) for the server to process it.
  - *Benefits:* This simplifies server architecture and improves scalability, because a single server doesn't need to keep track of user sessions, and a server crash won't destroy a client's state.
- **Client-Server Model:** Communication is always initiated by the client (e.g., a web browser) to request resources or actions, and the server waits for these requests to process and respond.

---

## 2. Transport Protocol & HTTP Versions

HTTP relies on a reliable, connection-based transport protocol, almost universally **TCP (Transmission Control Protocol)**. Over the years, HTTP has evolved to improve how these TCP connections are handled:

| Version | Key Feature |
|---|---|
| **HTTP 1.0** | Opened a new TCP connection for every single request/response — highly inefficient |
| **HTTP 1.1** | Introduced *persistent connections* (`keep-alive`) as the default |
| **HTTP 2.0** | Multiplexing, binary framing, header compression, and server push |
| **HTTP 3.0** | Replaced TCP with QUIC (built over UDP) for faster connections and no head-of-line blocking |

---

## 3. Anatomy of HTTP Messages

Client-server communication happens via structured text messages.

**Request Message (Client → Server):**
- Request Method (e.g., `GET`, `POST`)
- Resource URL
- HTTP version
- Host domain
- Headers
- Blank line
- Optional Request Body

**Response Message (Server → Client):**
- HTTP version
- Status Code (e.g., `200`)
- Status Value (e.g., `OK`)
- Headers
- Blank line
- Response Body

---

## 4. HTTP Headers

Headers are key-value pairs that act as metadata for the package being transmitted, allowing the system to be highly extensible and act as a "remote control" to dictate server behavior.

- **Request Headers:** Sent by the client to provide context (e.g., `User-Agent` identifies the browser, `Authorization` sends credentials).
- **General Headers:** Apply to both requests and responses (e.g., `Date`, `Connection`, `Cache-Control`).
- **Representation Headers:** Describe the message body (e.g., `Content-Type` for media format like JSON/HTML, `Content-Length` for byte size, `Content-Encoding` for gzip compression).
- **Security Headers:** Protect against attacks (e.g., `Strict-Transport-Security` forces HTTPS, `Content-Security-Policy` prevents cross-site scripting, `Set-Cookie` with HTTP-only flags).

---

## 5. HTTP Methods and Idempotency

Methods define the semantic *intent* of the client's request.

| Method | Description |
|---|---|
| `GET` | Fetches data from the server without modifying anything |
| `POST` | Submits new data to the server (includes a request body) |
| `PATCH` | Partially updates an existing resource |
| `PUT` | Completely replaces an existing resource with the provided body |
| `DELETE` | Removes a resource |
| `OPTIONS` | Inquires about the server's capabilities (used heavily in CORS) |

**Idempotency** is a crucial concept here:

- **Idempotent Methods:** Can be executed multiple times and yield the exact same result on the server state (e.g., `GET`, `PUT`, `DELETE`).
- **Non-Idempotent Methods:** Running them multiple times creates different results (e.g., submitting a `POST` request twice creates two separate resources).

---

## 6. Cross-Origin Resource Sharing (CORS)

Browsers enforce a **Same-Origin Policy**, blocking web apps from making requests to different domains (origins). CORS is a security mechanism to bypass this safely.

**Simple Requests** (usually `GET` or `POST` with standard headers/content types):
1. The browser automatically adds an `Origin` header.
2. If the server allows the request, it replies with `Access-Control-Allow-Origin` containing the client's domain (or a `*` wildcard).
3. If the header is missing, the browser blocks the response.

**Pre-flight Requests** — triggered when a request uses a non-simple method (`PUT`/`DELETE`), requires authorization headers, or uses `application/json`:
1. The browser first fires an `OPTIONS` request asking if the route supports the intended method and headers.
2. The server replies with `204 No Content`, explicitly listing allowed origins, methods, headers, and a `max-age` to cache this configuration.
3. If successful, the browser then sends the actual original request.

---

## 7. Standardized Status Codes

Status codes are three-digit numbers that act as a universal language to indicate the outcome of a request.

### 1xx — Informational
Indicates headers received; client can proceed.
- `100 Continue` — used for large uploads

### 2xx — Success
- `200 OK` — successful operation
- `201 Created` — usually follows a `POST` request
- `204 No Content` — successful, but no body to return (used in `OPTIONS` or `DELETE`)

### 3xx — Redirection
- `301 Moved Permanently` — the resource has a new URL
- `302 Found / Temporary Redirect` — temporarily forward to a new route
- `304 Not Modified` — tells the client to use its locally cached version

### 4xx — Client Errors
- `400 Bad Request` — invalid data format sent by client
- `401 Unauthorized` — missing or invalid authentication token
- `403 Forbidden` — authenticated, but lacks necessary permissions
- `404 Not Found` — incorrect URL or deleted resource
- `405 Method Not Allowed` — using the wrong method for a route
- `409 Conflict` — business logic violation (e.g., duplicate username)
- `429 Too Many Requests` — client has hit rate limits

### 5xx — Server Errors
- `500 Internal Server Error` — an unhandled exception crashed the server
- `501 Not Implemented` — feature not yet supported
- `502 Bad Gateway` / `504 Gateway Timeout` — proxies or load balancers failing to reach upstream servers
- `503 Service Unavailable` — server down or under maintenance

---

## 8. HTTP Caching

Caching reuses previously downloaded responses to save bandwidth and load times.

1. When a client first fetches a resource, the server responds with the payload alongside three headers: `Cache-Control` (sets max duration), `ETag` (a unique hash of the payload), and `Last-Modified`.
2. On subsequent requests, the client sends conditional headers: `If-None-Match` (carrying the ETag) or `If-Modified-Since`.
3. If the data on the server hasn't changed, the server sends an empty `304 Not Modified` response, instructing the browser to use its cached copy. If it has changed, it sends `200 OK` with new data and a new ETag.

---

## 9. Content Negotiation and Compression

Clients and servers can negotiate the best format to exchange data.

- The client sends preferences via:
  - `Accept` — e.g., `application/json` vs `application/xml`
  - `Accept-Language` — e.g., `en` vs `es`
  - `Accept-Encoding` — e.g., `gzip`
- The server responds with the appropriate format.

**Compression:** By negotiating an encoding like `gzip`, a server can drastically compress text responses (e.g., shrinking a 26 MB JSON payload down to 3.8 MB) to save massive amounts of network bandwidth.

---

## 10. Handling Large Data Transfers

- **Large Client Uploads (Images/Video):** Standard JSON is unsuitable for binary data. Instead, clients use a `multipart/form-data` request, which breaks the file into chunks separated by a unique string delimiter defined in the `boundary` header.
- **Large Server Downloads:** To prevent timing out, the server streams the file in chunks using `Content-Type: text/event-stream` and `Connection: keep-alive`. The browser continually appends these chunks until the transfer finishes.

---

## 11. Security (SSL/TLS & HTTPS)

- **TLS (Transport Layer Security):** The modern, secure replacement for the outdated SSL protocol.
- It encrypts data in transit to prevent interception (eavesdropping) or tampering, utilizing certificates to verify the server's identity.
- **HTTPS:** The standard HTTP protocol wrapped inside a secure TLS connection.