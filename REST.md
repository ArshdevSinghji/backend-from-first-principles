# Complete REST API Design

## 1. What Is REST? (History and Definition)

- The origin: In the 1990s, Tim Berners-Lee invented the World Wide Web. As it grew exponentially, it faced a scalability crisis. In 2000, Roy Fielding proposed a standardized architectural style to solve this, named REST (Representational State Transfer).
- Breaking down the name:
  - Representational: Resources on the web can have multiple representations based on who is asking. For an API client, it might be JSON; for a browser, HTML.
  - State: The current condition or attributes of a specific resource, such as the items and total price in a shopping cart.
  - Transfer: Moving these representations between client and server using standard HTTP methods.

## 2. The 6 Constraints of REST Architecture

- To be truly RESTful and highly scalable, a system should follow these constraints proposed by Roy Fielding:
  1. Client-Server: A strict separation of concerns where clients handle UI and servers handle data and business logic.
  2. Uniform Interface: A standardized way for all components to communicate.
  3. Layered System: Architecture is composed of hierarchical layers, such as load balancers or proxies.
  4. Cache: Responses explicitly declare whether they are cacheable.
  5. Stateless: The server keeps no memory of past requests; each request contains everything needed.
  6. Code on Demand (optional): The server can extend client behavior by sending executable code.

## 3. Anatomy of a RESTful Route

- When designing API URLs, follow standard hierarchical conventions.
- The structure: `https://api.example.com/v1/books` includes scheme (`https`), API subdomain (`api.`), version (`v1`), and resource path (`books`).
- Always use plural nouns for resources, such as `/books` or `/organizations`, even when fetching one item (`/books/123`).
- Formatting: avoid spaces and underscores in URLs. Convert readable slugs to lowercase with hyphens, such as `/books/harry-potter`.
- Hierarchical nesting: `/organizations/123/projects` means projects that belong to organization `123`.

## 4. HTTP Methods and Idempotency

- Idempotency means applying the same operation multiple times leads to the same final side-effects as applying it once.
- GET (idempotent): fetches data without changing server state.
- PATCH vs PUT (idempotent): both are updates where repeating the same payload leads to the same final state.
  - PATCH: updates selected fields.
  - PUT: replaces the full resource representation.
- DELETE (idempotent): first call removes the resource; repeated calls do not create additional side-effects.
- POST (non-idempotent): repeated calls can create multiple new resources.

## 5. Custom Actions (Exception to CRUD)

- Some actions do not fit CRUD cleanly, such as cloning a project or archiving an organization.
- Rule: when the action does not map to standard CRUD, model it as a POST.
- Route design: append the action verb to a specific resource route, such as `POST /projects/123/clone` or `POST /organizations/5/archive`.

## 6. Designing Robust List APIs (GET)

- List endpoints should support pagination, sorting, and filtering.

1. Pagination: return chunks of data instead of huge payloads.
   - `data`: objects for the current page.
   - `total`: total records in storage.
   - `page`: current page number.
   - `totalPages`: total page count.
2. Sorting: support query parameters such as `sortBy=name` and `sortOrder=ascending`.
3. Filtering: support query parameters such as `?status=active&name=org`.

## 7. Handling HTTP Status Codes Properly

- `200 OK`: successful fetch, update, or custom action.
- `201 Created`: successful resource creation through POST.
- `204 No Content`: successful delete with empty response body.
- 404 (Not Found) rule:
  - Return `404` when a specific resource ID does not exist, such as `GET /users/999`.
  - For list endpoints with no matches, return `200 OK` with an empty array `[]`, not `404`.

## 8. Golden Rules and Best Practices for API Engineers

- Extract nouns from UI: identify entities users interact with (projects, users, tasks), then model those as resources.
- Implement sane defaults: if optional fields are missing, apply defaults such as limit `10`, sort by `created_at` descending, or status `active`.
- Keep consistency: use one naming convention (for example, `camelCase`) and avoid mixing field names like `description` vs `desc`.
- Provide interactive documentation: use Swagger/OpenAPI for discoverability and testing.
