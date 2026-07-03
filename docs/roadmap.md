# Roadmap

`go-ruby-oauth2/oauth2` is grown **test-first**, each capability differential-tested
against the reference `oauth2` gem rather than built in isolation. The
deterministic, interpreter-independent client core is **complete**.

| Stage | What | Status |
| --- | --- | --- |
| Authorization URLs | `AuthCode().AuthorizeURL(...)` — client_id / `response_type=code` defaults, sorted keys, RFC 3986 percent-encoding, byte-faithful to the gem. | **Done** |
| Every grant | Authorization-code, client-credentials, resource-owner password, JWT-bearer assertion and refresh-token — each builds the token request spec. | **Done** |
| Client authentication | `basic_auth` (default), `request_body`, `tls_client_auth` (RFC 8705), over `POST` (form body) or `GET` (query params). | **Done** |
| Response parsing | Content-type dispatch into an `AccessToken`; promoted fields plus residual provider params in response order. | **Done** |
| AccessToken & PKCE | Expiry derivation, refresh-request builder, `to_hash`, resource-request injection (header/query/body, `Bearer`/`MAC`/custom); `CodeChallenge` for PKCE `S256` / `plain`. | **Done** |
| Differential oracle & coverage | Authorize URLs, token-request specs, PKCE challenges and parsed responses diffed byte-for-byte against the gem; 100% coverage on the ruby-free lanes, green across 6 arches. | **Done** |

## Documented out-of-scope boundaries

These are **deliberate**, recorded so the module's surface is unambiguous:

- **The HTTP transport is a host seam.** The library builds a `Request` and parses
  a `Response`; the actual round-trip is the host's `RoundTripper` — exactly as the
  gem delegates to Faraday. This is why `rbgo` binds the transport rather than the
  reverse.
- **Deterministic protocol logic only.** URL/param construction and response
  parsing need no interpreter; anything requiring a live Ruby binding belongs in
  the consumer.
- **Reference is the `oauth2` gem.** Byte-for-byte conformance targets the gem's
  observable output.

See [Usage & API](api.md) for the surface and [Why pure Go](why.md) for the
deterministic / transport split.
