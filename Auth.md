# Authentication and Authorization for Backend Engineers

## 1. Core Definitions

- Authentication answers who you are.
- Authorization answers what you are allowed to do.
- Authentication happens first, then authorization decisions are applied on top.

## 2. Why This Matters in Backend Systems

- HTTP is stateless, so identity must be attached to each request through a token, cookie, or session identifier.
- Backends must balance security, developer ergonomics, and scalability.
- A secure identity layer is foundational for every protected route and business action.

## 3. Common Authentication Building Blocks

### Sessions

- The server creates a session after login and stores it in a fast store such as Redis.
- The client gets a session identifier, usually via a cookie.
- Each new request carries that identifier, and the server loads the corresponding session data.

### JWT (JSON Web Token)

- JWT enables stateless authentication by carrying claims inside the token.
- A typical token has three parts: header, payload, and signature.
- The server verifies the signature before trusting claims like subject, role, or expiry.

### Cookies

- Cookies let servers persist small pieces of data in the browser.
- HttpOnly cookies reduce token theft through browser JavaScript access.
- Secure and SameSite flags are important to reduce transport and CSRF risks.

## 4. Main Authentication Approaches

### Stateful Authentication

- Flow: user logs in, server stores session, browser sends session id on later requests, server validates via session store.
- Strength: easy server-side revocation and centralized control.
- Tradeoff: session storage and synchronization overhead at scale.

### Stateless Authentication

- Flow: user logs in, server signs JWT, client sends token on later requests, server verifies token cryptographically.
- Strength: excellent horizontal scalability and low lookup overhead.
- Tradeoff: revocation is hard before token expiry unless extra infrastructure is added.

### API Key Authentication

- Used for service-to-service or automated programmatic access.
- Client sends a generated key in request headers.
- Best practice is scoped keys, rotation, and strict usage monitoring.

### OAuth 2.0 and OpenID Connect

- OAuth 2.0 handles delegated authorization through scoped access tokens.
- OpenID Connect adds identity with an ID token and standard user profile claims.
- This powers sign-in flows like Sign in with Google.

## 5. Authorization with RBAC

- Role-Based Access Control maps users to roles and roles to permissions.
- Example roles: User, Moderator, Admin.
- During request processing, backend checks role and permission before protected actions.
- Unauthorized actions return 403 Forbidden.

## 6. Security Best Practices

- Use generic authentication error responses to avoid account enumeration.
- Hash passwords with a modern adaptive algorithm such as Argon2 or bcrypt.
- Compare secrets in constant time to reduce timing side channels.
- Enforce short token lifetimes and support refresh-token rotation where needed.
- Rate-limit login and sensitive endpoints to slow brute-force attempts.
- Log authentication events for alerting and incident investigation.

## 7. End-to-End Request Mental Model

1. User submits credentials to login endpoint.
2. Backend verifies credentials and creates session or token.
3. Client stores credential artifact and sends it with future requests.
4. Backend validates identity artifact on each protected request.
5. Backend checks authorization rules for the requested action.
6. Backend returns resource or rejects with 401 or 403 based on failure type.
