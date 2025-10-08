# Typical OpenAPI Mistakes and How to Fix Them

Lately I’ve noticed the same issues repeating in automatically generated OpenAPI specs — whether from FastAPI, AI tools, or custom code.

Here’s a summary of the most common ones and how to fix them.

| Mistake | Example | Why it's wrong | How to fix |
|---------|---------|----------------|------------|
| String without pattern or length limit | 'type: string' | Risk of DoS via huge strings | Add 'pattern', 'minLength', 'maxLength' |
| Integer without limits | 'type: integer' | Unlimited value range, missing validation | Add 'minimum', 'maximum' |
| Array without item count limits | 'type: array' | Potential large payload -> memory issues | Add 'minItems', 'maxItems' |
| Missing property description | | Unclear, poor docs | Add 'description' for every property |
| Non‑camelCase parameters | 'book-Id' | Breaks client SDK generation | Change to camelCase ('bookId'). This works best in client SDKs, especially in any JS |
| URL naming incorrect | '/mylibrary/section_names' | Inconsistent -> hard to read | Use kebab-case for multi‑word path segments (`/my-library/section-names/`). This improves readability, especially when links are automatically underlined|
| Missing version field | None in path/query/header | Clients can’t know spec version | Add version param 'YYYY-MM-DD': `x-api-key: 2025-10-03` |
| Missing required error responses | No '400', '401, '500', 'default' | Poor API guarantee | Always include them |
| OperationId wrong format | 'get_section_by_name' | Hard to scan | Use '{Entity}_{Action}' format for clarity ('sectionNameList_Get')|
| No common headers in responses | No 'RateLimit'/'Access-Control-Allow-Origin' | CORS & rate info missing | Add from 'components/headers' |
| 429 without Retry-After | | No info on when to retry | Add 'Retry-After' header |
| Missing security schema | No components.securitySchemes | API lacks documented authentication -> unsafe and unclear for consumers	Define in components.securitySchemes, e.g.: oauth2, apiKey, etc.|

## Snippets

### Required Headers in All Responses

Add these universally in all responses via 'components/headers':

```yaml
'200':
   description: Successful Response
      headers:
         RateLimit:
            $ref: '#/components/headers/RateLimit'
         Access-Control-Allow-Origin:
            $ref: '#/components/headers/Access-Control-Allow-Origin'
```

For 429, also include:

```yaml
'429'
   Retry-After:
      $ref: '#/components/headers/RetryAfter'
```

How to describe in the components section of OpenAPI

```yaml
  headers:
    Access-Control-Allow-Origin:
      description: CORS support
      schema:
        type: string
        pattern: "^(?!\\s*$).+"
        maxLength: 1024
        minLength: 1
        example: '*'
    RateLimit:
      description: Rate limit
      schema:
        type: integer
        format: int64
        maximum: 2147483647
        minimum: 0
        example: 1000
    RetryAfter:
      description: >
        If the rate limit is exceeded, it indicates how many seconds you should wait before retrying
      schema:
        type: integer
        format: int32
        maximum: 3600
        minimum: 1
```

### Don’t forget to add security schema

```yaml
components:
  securitySchemes:
    oauthAuthCode:
      type: oauth2
      description: >
        A bearer token in the format of a JWS and conforms to RFC8725
      flows:
        authorizationCode:
          authorizationUrl: "https://login.microsoftonline.com/{your-tenant-id}/oauth2/v2.0/authorize"
          tokenUrl: "https://login.microsoftonline.com/{your-tenant-id}/oauth2/v2.0/token"
          refreshUrl: "https://login.microsoftonline.com/{your-tenant-id}/oauth2/v2.0/token"
          scopes:
            "api://my-librabry/Books.Read": Allows to get a book from my librabry
            offline_access: Access to refresh tokens
    apiKeyHeader:
      description: API Key needed to access the endpoints
      type: apiKey
      name: Ocp-Apim-Subscription-Key
      in: header
```

### Don’t forget to add versions

```yaml
parameters:
  - name: x-api-version
    in: header
    required: true
    schema:
      $ref: '#/components/schemas/APIVersion'
```

```yaml
components:
  schemas:
    APIVersion:
      type: string
      description: API version in format YYYY-MM-DD
      pattern: "^[0-9]{4}-[0-9]{2}-[0-9]{2}$"
      example: "2025-10-03"
```

