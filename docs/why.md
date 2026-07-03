# Why pure Go

`go-ruby-oauth2/oauth2` reimplements the client core of Ruby's `oauth2` gem in
**pure Go, with cgo disabled**. The slice of the gem it covers is **deterministic
and interpreter-independent**: given a client config and params, the authorization
URL, the per-grant token-request spec, and the parse of a token/error response are
pure functions of those inputs — no live binding, no evaluation of arbitrary Ruby.
That is exactly the part that can — and should — live as a standalone Go library,
separate from the interpreter.

## Extracted for reuse, and reused by anyone

This library provides the deterministic OAuth2 client that both the interpreter
and its siblings need:

- any Go program can import `github.com/go-ruby-oauth2/oauth2` directly, with no
  Ruby runtime;
- the dependency runs the *other* way — `rbgo` binds this module as the OAuth2
  client for [go-embedded-ruby](https://github.com/go-embedded-ruby/ruby), rather
  than this module depending on the interpreter;
- [go-ruby-oidc](https://github.com/go-ruby-oidc/oidc) builds **on top of it** —
  reusing its authorization-URL and token-request construction rather than
  reimplementing OAuth2;
- the behaviour is pinned by a **differential oracle** against the reference
  `oauth2` gem, independent of any one consumer.

## The HTTP round-trip is a host seam

The gem lets Faraday perform the transport; this library mirrors that split. The
core builds a `Request` (method, URL, params, body, headers) and hands it to a
`RoundTripper`:

```go
type RoundTripper interface {
    RoundTrip(req *Request) (*Response, error)
}
```

So **the core never opens a socket** — a host wires the transport (to
[go-ruby-net-http](https://github.com/go-ruby-net-http/net-http) / faraday), and
tests supply an in-memory `RoundTripper`. Everything the library itself does — URL
building, token-request specs, response parsing — is therefore testable without a
network, which is what lets it hold 100% coverage on the ruby-free lanes alone.

## Why pure Go matters here

Because the library is CGO-free, it:

- cross-compiles to every Go target with no C toolchain, and links into a single
  static binary;
- has **no dependency on the Ruby runtime** — the dependency runs the other way;
- can be differentially tested against the `oauth2` gem wherever it is installed,
  while the cross-arch lanes (where the gem is absent) still validate the library.

See [Usage & API](api.md) for the surface and [Roadmap](roadmap.md) for what is a
host seam by design.
