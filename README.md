# go-msadmin

[![CI](https://github.com/philband/go-msadmin/actions/workflows/ci.yml/badge.svg)](https://github.com/philband/go-msadmin/actions/workflows/ci.yml)

Small, dependency-free Go building blocks for talking to the **eventually
consistent, throttled Microsoft admin APIs** (Exchange Online / Purview, Teams,
Graph, ARM). Shared by [`go-exoscc`](https://github.com/philband/go-exoscc), the
Terraform providers built on [`tf-msadmin`](https://github.com/philband/tf-msadmin),
and any other client that needs the same behaviour.

```bash
go get github.com/philband/go-msadmin
```

## Packages

### `consistency` — retry until visible

These APIs are read-your-writes consistent only within a single backend session:
a create/update in one session is frequently not yet visible to a read from
another. `RetryUntil` polls a getter until the resource is present (and,
optionally, until it reflects a written value) before concluding it is absent.

```go
obj, ok, err := consistency.RetryUntil(ctx, consistency.Config{}, get, nil)
```

### `retry` — transient & rate-limit aware retries

Every service throttles with HTTP 429 (often with `Retry-After`) and returns
transient 5xx/network errors under load. `retry` centralises the policy: honour
`Retry-After`, otherwise exponential backoff with jitter, bounded attempts.

- `retry.Do` — retry an arbitrary operation with a caller-supplied classifier.
- `retry.Transport` — a drop-in `http.RoundTripper` that retries transient
  responses transparently (replays the body via `Request.GetBody`).

```go
hc := &http.Client{Transport: retry.NewTransport(nil, retry.Config{})}
```

## License

MIT — see [LICENSE](LICENSE). Not affiliated with or endorsed by Microsoft.
