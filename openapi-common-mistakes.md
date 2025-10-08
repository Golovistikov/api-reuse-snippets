# Typical OpenAPI Mistakes and How to Fix Them

Lately I’ve noticed the same issues repeating in automatically generated OpenAPI specs — whether from FastAPI, AI tools, or custom code.

Here’s a summary of the most common ones and how to fix them.

| Mistake | Example | Why it's wrong | How to fix |
|---------|---------|----------------|------------|
| String without pattern or length limit | `type: string` | Risk of DoS via huge strings | Add `pattern`, `minLength`, `maxLength` |
| Integer without limits | `type: integer` | Unlimited value range, missing validation | Add `minimum`, `maximum` |
| Array without item count limits | `type: array` | Potential large payload -> memory issues | Add `minItems`, `maxItems` |
| Missing property description | (none) | Unclear docs | Add `description` for every property |
| Non-camelCase parameters | `funding-provider` | Breaks client SDK generation | Use `fundingProvider` |
| Wrong URL naming | `/wallet/paymentMethods` | Hard to read, inconsistent | Use kebab-case for path segments |
| Missing version field | None in path/query/header | Clients can’t detect version | Add version param (`YYYY-MM-DD`) |
| Missing error responses | No `400`, `401`, `429`, `500` | Poor API guarantee | Always include them |
| Bad `operationId` format | `listPaymentMethods` | Hard to scan | Use `Entity_Action` |
| No common headers | Missing `RateLimit`, `CORS` headers | Missing useful info | Add via `components/headers` |
| 429 without `Retry-After` | (none) | Clients can’t retry properly | Add `Retry-After` header |
| Missing security schema | (none) | Undocumented authentication | Add `components.securitySchemes` |

## Naming Conventions typical issue

- Parameters - use camelCase  
  book-id -> bookId (works better in client SDKs, especially in any JS)
- Path segments/URLs - kebab-case for multi‑word segments  
  :ok:: /my-library/section-names/ (this improves readability, especially when links are automatically underlined)
  (Short single words can stay lowercase)
  :x:: /mylibrabry/section_names
- OperationId - {Entity}_{Action} format for clarity  
  :ok:: sectionNameList_Get
  :x:: get_section_by_name
- Version - must be passed in path param, query param, or header, in format YYYY-MM-DD
  Example: x-api-key: 2025-10-03

## Don’t forget to add responses

**Every response** must have:

- 401 - Unauthorized
- 400 (GET/DELETE with bad parameters) or 422 (validation errors for body)
- 500 - Internal Server Error
- default - unexpected errors
- 429 - Too Many Requests (rate limit exceeded)

## Don’t forget to add headers
RateLimit, Retry-After, Access-Control-Allow-Origin

## Example Snippets

See the [`snippets`](./snippets) folder for ready-to-use YAML blocks for headers, responses, and security schemas.
