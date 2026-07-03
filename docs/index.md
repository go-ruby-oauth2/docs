# go-ruby-oauth2 documentation

**Ruby's `oauth2` gem — authorization URLs, every grant's token request, client-auth schemes, PKCE and response parsing — in pure Go, no cgo.**

`go-ruby-oauth2/oauth2` is a faithful, pure-Go (zero cgo) reimplementation of the
**deterministic protocol core** of Ruby's
[`oauth2`](https://gitlab.com/oauth-xx/oauth2) gem — the interpreter-independent
OAuth2 client. It matches the gem byte-for-byte. The module path is
`github.com/go-ruby-oauth2/oauth2`.

It is a **standalone, reusable** library importable by any Go program, the OAuth2
client bound into [go-embedded-ruby](https://github.com/go-embedded-ruby/ruby) by
`rbgo`, and the base that [go-ruby-oidc](https://github.com/go-ruby-oidc/oidc)
builds on. The dependency runs the other way: this library has **no dependency on
the Ruby runtime**, and it never opens a socket — the HTTP round-trip is a host
`RoundTripper` seam.

!!! success "Status: complete — gem byte-exact"
    A faithful pure-Go port of the `oauth2` gem's deterministic client protocol,
    validated by a **differential oracle** against the reference gem — authorize
    URLs, token-request specs, PKCE challenges and parsed responses compared
    byte-for-byte — at 100% coverage, `gofmt` + `go vet` clean, CI green across
    the six 64-bit Go targets.

## Quick taste

```go
client := oauth2.NewClient("myid", "mysecret", oauth2.Options{
    Site:         "https://provider.example.com",
    AuthorizeURL: "/oauth/authorize",
    TokenURL:     "/oauth/token",
})

url := client.AuthCode().AuthorizeURL(oauth2.Params{
    {"redirect_uri", "https://app/cb"}, {"scope", "read write"}, {"state", "xyz"},
})
// .../oauth/authorize?client_id=myid&redirect_uri=https%3A%2F%2Fapp%2Fcb&
//   response_type=code&scope=read%20write&state=xyz

req := client.AuthCode().GetTokenRequest("thecode", oauth2.Params{{"redirect_uri", "https://app/cb"}})
resp, _ := transport.RoundTrip(req)   // host seam (go-ruby-net-http / faraday)
tok, _ := client.ParseToken(resp)
```

## What it is — and isn't

The URL/param construction and response parsing are fully deterministic and need
**no interpreter**, so they live here as pure Go. The **HTTP round-trip itself is
a host seam**: the library builds a `Request` and a host turns it into a
`Response` via a `RoundTripper`, mirroring the gem where Faraday performs the
transport.

## Repositories

| Repo | What it is |
| --- | --- |
| [`oauth2`](https://github.com/go-ruby-oauth2/oauth2) | the library — the OAuth2 client core in pure Go |
| [`docs`](https://github.com/go-ruby-oauth2/docs) | this documentation site (MkDocs Material, versioned with mike) |
| [`go-ruby-oauth2.github.io`](https://github.com/go-ruby-oauth2/go-ruby-oauth2.github.io) | the organization landing page (Hugo) |
| [`brand`](https://github.com/go-ruby-oauth2/brand) | logo and brand assets |

## Principles

- **Pure Go, `CGO_ENABLED=0`** — trivial cross-compilation, a single static
  binary, no C toolchain.
- **Gem byte-exact.** Output matches the reference `oauth2` gem exactly, not
  approximately, validated by a differential oracle.
- **The transport is a host seam.** The core builds a `Request` and never dials;
  the host binds a `RoundTripper`, exactly as the gem uses Faraday.
- **Standalone & reusable.** No dependency on the Ruby runtime — the dependency
  runs the other way.
- **100% test coverage** is the target, enforced as a CI gate, across 6 arches.

## Where to go next

- [Why pure Go](why.md) — why the gem's client protocol is deterministic enough
  to live as a standalone, interpreter-independent Go library.
- [Usage & API](api.md) — the public surface and worked examples.
- [Roadmap](roadmap.md) — what is done and what is a host seam by design.

Source lives at [github.com/go-ruby-oauth2/oauth2](https://github.com/go-ruby-oauth2/oauth2).
