# Talk to Qwen

A single static page (`index.html`, no build step, just a text box) that
streams chat responses straight from your browser to a vLLM
OpenAI-compatible server via SSE. The endpoint and model are hardcoded in
`index.html`:

```js
const ENDPOINT = "https://taisp-ws-001.tail519d90.ts.net/v1/chat/completions";
const MODEL = "Qwen/Qwen2.5-7B-Instruct";
```

The endpoint is served over HTTPS via `tailscale serve --bg 8000` running on the
taisp server, which fronts vLLM's plain-HTTP port 8000 with a TLS cert on the
tailnet's `.ts.net` hostname. This avoids browser mixed-content blocking
entirely (no more "allow insecure content" workarounds needed). Only devices
on the same tailnet can reach it, same as before.

Edit those two lines if your server address or model changes. Chat history
is kept in your browser's `localStorage` only — nothing goes through a
backend, so there's nothing to host besides this static file.

## Deploying to GitHub Pages

1. Push this repo to GitHub.
2. Repo Settings → Pages → Source: deploy from branch → pick `main` (or
   whichever branch) and `/ (root)`.
3. Your page will be live at `https://<username>.github.io/<repo>/`.

## Server-side requirements

Two things have to be true on the vLLM side for the page to work:

### 1. CORS
vLLM must explicitly allow requests from `https://<username>.github.io`, or
the browser blocks the response with a CORS error. Start vLLM with:

```
vllm serve Qwen/Qwen2.5-7B-Instruct \
  --allowed-origins '["https://<username>.github.io"]' \
  ...your other flags
```

(Use `'["*"]'` to allow any origin — simplest for personal/tailnet-only use
since the server isn't reachable from the public internet anyway.)

### 2. HTTPS (to avoid mixed-content blocking)
GitHub Pages serves this site over HTTPS, and browsers block an HTTPS page
from calling a plain-HTTP endpoint. That's why the endpoint above is a
`tailscale serve`-fronted `.ts.net` HTTPS URL instead of the raw
`http://<tailscale-ip>:8000` address:

```bash
tailscale serve --bg 8000
```

run on the machine hosting vLLM. This terminates TLS via Tailscale and
forwards to `localhost:8000`. Only devices on the same tailnet can reach it
(same reachability as the raw IP — just with a valid HTTPS cert now).

## Files

- `index.html` — the whole app (HTML/CSS/JS, no dependencies).
