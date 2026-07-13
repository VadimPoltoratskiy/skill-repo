# Fastify Best Practices — Anti-Patterns

Apply these rules to every changed file under `server/src/modules/*/routes.ts` and `server/src/app.ts`.

## CRITICAL

- **Handler bypasses schema validation** — every route must declare a Zod `body`/`params`/`querystring` schema via `fastify-type-provider-zod`. Never call `Schema.parse(req.body)` manually inside a handler; invalid input must be rejected 422 before the handler runs.
- **Business logic inside a route handler** — handlers must parse → delegate to service → return. Any `if/else` that decides business outcomes or any DB call inside a handler is a violation.
- **Uncaught async throw leaks 500 with stack trace** — all async handlers must be wrapped or use Fastify's built-in async error propagation. Never swallow errors with an empty `catch {}`.

## HIGH

- **Plugin registers routes without a prefix** — every feature plugin registered in `app.ts` must pass a `prefix` option. Unprefixed routes collide across modules.
- **SSE route not exempted from rate limiting** — long-lived streaming routes must be listed in the rate-limit exclusion list; otherwise the connection is closed after the global request cap.
- **Secrets or config read inside a handler** — config must be resolved at startup via `loadConfig()` and injected. Reading `process.env` or file I/O inside a request handler adds latency on every call.
- **Reply sent after return** — calling `reply.send()` and then returning a value (or vice-versa) produces a double-send error. Use `return reply.send(data)` or `return data` consistently.

## MEDIUM

- **Logging sensitive fields** — `req.log.info` must not log passwords, tokens, or PII. Use Pino's `redact` option or omit the field.
- **Hardcoded port/host in `listen()`** — port and host must come from config, not literals.