# Performance

`go-ruby-oauth2/oauth2` is the pure-Go library that
[`rbgo`](https://github.com/go-embedded-ruby/ruby) binds for the OAuth2 client
protocol. This page records how the module's performance is characterised, as
part of the ecosystem-wide per-module parity work.

## What dominates the cost

An OAuth2 client operation is **string-construction / parsing work**, not a heavy
compute loop, and the network hop is **outside** this library:

- building an authorization URL is a sorted, percent-encoded query-string
  assembly; building a token request spec is a small map / form-encode;
- parsing a response is a content-type dispatch plus a JSON or form decode into an
  ordered `Map`;
- the actual HTTP round-trip goes through the host's `RoundTripper` — its latency
  is the transport's and the provider's, not the module's.

So the meaningful figure is the **CPU cost of one authorize-URL build**, one
**token-request spec build**, and one **response parse**, with the transport
mocked out — pure, allocation-light string work.

## Reference-parity framing

The parity question for this module is narrow: the gem's equivalent work is Ruby
string formatting and `URI`/`JSON` handling; this library does the same in Go's
`strings` / `sort` / `encoding/json`. There is no interpreter dispatch on the hot
path — a built URL is a pure function of `(client, params)`. Correctness is pinned
byte-for-byte by the differential oracle; the performance question is simply
whether the Go assembly of the same bytes is at least as cheap.

!!! note "No fabricated numbers"
    This page deliberately does **not** publish a benchmark table: doing so
    honestly requires a committed, reproducible harness (a Go driver plus the gem
    workload) measured on a named host, exactly as the numeric modules do. That
    harness is the tracked follow-up for this module. When it lands, the measured
    `ns/op` for authorize-URL build, token-request build and response parse will
    be recorded here, with the host, dates and method stated — never estimated.

## Reproducing locally

The library's unit benchmarks run with the standard toolchain against a mocked
`RoundTripper`, so no network or provider is required:

```sh
go test -run=^$ -bench=. -benchmem ./...
```
