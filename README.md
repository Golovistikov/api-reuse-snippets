# api-reuse-snippets
A living collection of practical API design snippets and notes — made to copy, not to impress

Small reusable pieces for OpenAPI and API design. Based on real-world mistakes and patterns I often see

I keep this repo alive as part of my weekly learning habit.  Feel free to contribute
If something here saved you a few minutes — that’s already success.

The goal is simple: help teams create cleaner, safer, and more consistent API

## Content

- [`openapi-common-mistakes.md`](./openapi-common-mistakes.md) – a living list of common OpenAPI issues and how to fix them  
- More to come later (naming conventions, validation patterns, response formats)

## Usage

These snippets are meant to be reused directly inside your OpenAPI files or code generators.

Example:

```yaml
components:
  headers:
    $ref: './snippets/common-headers.yaml'
  responses:
    $ref: './snippets/common-responses.yaml'
