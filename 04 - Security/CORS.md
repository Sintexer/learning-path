---
flashcards:
  q-iexc: { nid: 1795356732341, hash: 97ensc23, sync: ievttamt }
  q-gwzr: { nid: 1795356732367, hash: a34gtcik, sync: kfgqptfa }
  q-62ic: { nid: 1795356732392, hash: fy7in9sx, sync: qsyfwj68 }
  q-v43q: { nid: 1795356732426, hash: zgkaidvt, sync: a42zu5ek }
---

**CORS (Cross-Origin Resource Sharing)** is a browser-enforced [[HTTP]] header mechanism that selectively relaxes the **Same-Origin Policy (SOP)**, allowing a web app on Origin A to read responses from an [[API]] on Origin B.

What three components define a browser Origin?::Scheme (protocol), host (domain), and port.
^q-iexc

## Simple vs Preflight Requests

- **Simple requests** (`GET`, `HEAD`, or `POST` with standard `Content-Type` like `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain` and no custom headers): Browser sends the request directly and checks `Access-Control-Allow-Origin` before exposing the response to JavaScript.
- **Preflight requests** (triggered by `PUT`, `DELETE`, `PATCH`, `Content-Type: application/json`, or custom headers like `Authorization`): Browser first sends an `OPTIONS` request with `Origin`, `Access-Control-Request-Method`, and `Access-Control-Request-Headers`. The actual request is sent only if the server responds with matching `Access-Control-Allow-*` headers.

## Key Headers

- `Access-Control-Allow-Origin`: Exact origin (e.g., `https://app.example.com`) or `*`.
- `Access-Control-Allow-Credentials: true`: Required for cookies or TLS client certs to be sent cross-origin.
- `Access-Control-Allow-Methods` / `Access-Control-Allow-Headers`: Permitted methods and headers for preflighted requests.
- `Access-Control-Max-Age`: Caches preflight results in the browser to avoid extra `OPTIONS` round-trips.

When `Access-Control-Allow-Credentials` is `true`, `Access-Control-Allow-Origin` ==cannot be the wildcard `*` and must match an exact allowlisted origin==.
^q-gwzr

> [!warning] Failure modes & trade-offs
> - **CORS is NOT a server-side security boundary or CSRF defense**: Only browsers enforce CORS (`curl`, scripts, and proxies ignore it). Worse, a simple cross-origin `POST` with cookies still executes on your server before the browser hides the response. Defend against CSRF with `SameSite=Lax/Strict` cookies, CSRF tokens, or non-cookie `Authorization` headers.
> - **Reflecting arbitrary `Origin` headers**: Echoing the request's `Origin` header alongside `Access-Control-Allow-Credentials: true` effectively disables SOP for every malicious site on the internet. Always validate against an explicit allowlist.
> - **Cache poisoning without `Vary: Origin`**: Returning a dynamic `Access-Control-Allow-Origin` without `Vary: Origin` causes [[CDN]] or [[Reverse proxy]] caches to serve Origin A's header to Origin B, breaking clients intermittently.

Always send `Vary: Origin` when dynamically setting `Access-Control-Allow-Origin` so CDNs and reverse proxies do not cache one origin's CORS header for another. #card-reminder
^q-62ic

## Why is CORS NOT a defense against CSRF? #card

1. CORS is enforced only by browsers on **reading responses** for simple requests—a malicious site can still trigger a simple cross-origin `POST` form submission carrying the victim's cookies, executing the state change on the backend before the browser blocks JS from reading the response.
2. Non-browser clients (`curl`, bots) ignore CORS completely.
3. Defend against CSRF with `SameSite` cookies, anti-CSRF tokens, or `Authorization` headers (which are never attached automatically by browsers).
^q-v43q

## Related

- [[HTTP]]
- [[HTTPS]]
- [[API Gateway]]
- [[Reverse proxy]]
- [[CDN]]
- [[Authentication]]
