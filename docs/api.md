# Usage & API

The public API lives at the module root (`github.com/go-ruby-oauth2/oauth2`). It
is **gem-shaped but Go-idiomatic**: the surface mirrors `OAuth2::Client` /
`AccessToken` / `Response` / `Error`, while following Go conventions — value
types, no global state, and an explicit transport seam.

!!! success "Status: implemented"
    The library is built and importable as `github.com/go-ruby-oauth2/oauth2`,
    bound into `rbgo` as the OAuth2 client and reused by go-ruby-oidc; see
    [Roadmap](roadmap.md).

## Install

```sh
go get github.com/go-ruby-oauth2/oauth2
```

## The transport seam — `RoundTripper`

The library builds a `Request`; a host performs the HTTP round-trip:

```go
type RoundTripper interface {
    RoundTrip(req *Request) (*Response, error)
}
```

A host binds it to go-ruby-net-http / faraday; tests supply an in-memory mock.
Nothing in this package opens a connection — exactly as the gem lets Faraday do
the transport.

## Worked example

```go
client := oauth2.NewClient("myid", "mysecret", oauth2.Options{
    Site:         "https://provider.example.com",
    AuthorizeURL: "/oauth/authorize",
    TokenURL:     "/oauth/token",
})

// 1. Authorization URL to redirect the user-agent to.
url := client.AuthCode().AuthorizeURL(oauth2.Params{
    {"redirect_uri", "https://app/cb"}, {"scope", "read write"}, {"state", "xyz"},
})

// 2. Build the token request for the returned code, run it through the host
//    transport (the seam), then parse the response into an AccessToken.
req := client.AuthCode().GetTokenRequest("thecode", oauth2.Params{{"redirect_uri", "https://app/cb"}})
// req.Method == "POST", req.Headers has Authorization: Basic base64(id:secret)
resp, _ := transport.RoundTrip(req)
tok, _ := client.ParseToken(resp)
fmt.Println(tok.Token, tok.ExpiresAt, tok.Expired())

// 3. Inject the token into a resource request.
headers, _, _ := tok.ApplyTo(nil, nil, nil) // Authorization: Bearer <token>
_ = headers
```

## PKCE

```go
verifier := "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk"
challenge := oauth2.CodeChallenge(verifier, oauth2.PKCES256)

url := client.AuthCode().AuthorizeURL(oauth2.Params{
    {"redirect_uri", "https://app/cb"},
    {"code_challenge", challenge},
    {"code_challenge_method", "S256"},
})
```

## gem ↔ package map

| gem | this package |
| --- | --- |
| `OAuth2::Client.new(id, secret, ...)` | `NewClient(id, secret, Options{...})` |
| `client.auth_code.authorize_url(...)` | `client.AuthCode().AuthorizeURL(params)` |
| `client.<grant>.get_token(...)` | `client.<Grant>().GetTokenRequest(...)` + `client.ParseToken(resp)` |
| `OAuth2::AccessToken` | `*AccessToken` |
| `OAuth2::Response#parsed` | `(*Response).Parsed()` |
| `OAuth2::Error` | `*Error` |
| Faraday transport | `RoundTripper` (host seam) |

## Surface

```go
func NewClient(id, secret string, opts Options) *Client
func (c *Client) AuthorizeURL() string
func (c *Client) TokenURL() string
func (c *Client) ParseToken(resp *Response) (*AccessToken, error)
func (c *Client) ParseRefreshToken(resp *Response, prevRefresh string) (*AccessToken, error)

func (c *Client) AuthCode() AuthCode
func (c *Client) ClientCredentials() ClientCredentials
func (c *Client) Password() Password
func (c *Client) Assertion() Assertion
func (c *Client) Refresh() Refresh

func (s AuthCode) AuthorizeURL(params Params) string
func (s AuthCode) GetTokenRequest(code string, extra Params) *Request
// ... one GetTokenRequest per grant strategy

type AccessToken struct {
    Token, RefreshToken string
    ExpiresAt           int64
    Params              *Map
    Mode
    HeaderFormat, ParamName string
}
func (t *AccessToken) Expires() bool
func (t *AccessToken) Expired() bool
func (t *AccessToken) RefreshRequest(extra ...Param) (*Request, error)
func (t *AccessToken) ApplyTo(headers, params, body *Map) (h, p, b *Map)
func (t *AccessToken) ToHash() *Map

type Request struct { Method, URL string; Params, Body, Headers *Map }
func (r *Request) EncodedBody() string
func (r *Request) FullURL() string

type Response struct { Status int; Headers *Map; Body string }
func (r *Response) Parsed() map[string]any
func (r *Response) ContentType() string

type Error struct { Response *Response; Code, Description string }
func (e *Error) Error() string
func (e *Error) FirstLine() string

func CodeChallenge(verifier string, method PKCEMethod) string // PKCE S256 / plain

type RoundTripper interface { RoundTrip(*Request) (*Response, error) }
```

## gem conformance

A **differential oracle** runs authorize URLs, token-request specs (each grant ×
auth-scheme × method), PKCE challenges, and parsed token/error responses through
both the reference `oauth2` gem and this library and compares them
**byte-for-byte**. The oracle scripts `$stdout.binmode` and skip themselves where
the gem is absent, so the qemu cross-arch and Windows lanes still pass the
coverage gate on the ruby-free tests alone.
