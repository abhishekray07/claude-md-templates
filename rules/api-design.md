---
description: API design standards for route handlers and endpoints
paths:
  - "src/api/**"
  - "src/routes/**"
  - "app/api/**"
---

# API Design

- Every endpoint must validate inputs at the boundary. Never trust raw request data.
- Return consistent error responses: `{ "error": { "code": "...", "message": "..." } }`.
- Use appropriate HTTP status codes: 201 for creation, 404 for not found, 422 for validation errors.
- Keep route handlers thin. Extract business logic into service functions.
- All endpoints must have request/response type definitions.
- Log errors with context (request ID, user ID, endpoint) — never log sensitive data (tokens, passwords).
- Add rate limiting to any public-facing endpoint.

<!--
CUSTOMIZATION NOTES (this comment is stripped from context, it's for you):

Adjust the paths globs to match your project structure:
- FastAPI: src/api/**, src/routers/**
- Express: src/routes/**, src/controllers/**
- Next.js: app/api/**, pages/api/**
-->
