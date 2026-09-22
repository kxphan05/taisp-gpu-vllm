# Talk to Qwen

A single static page (`index.html`, no build step) that sends chat requests
straight from your browser to a vLLM OpenAI-compatible server, e.g.

```
http://100.76.19.104:8000/v1/chat/completions
```

Open the page, click the ⚙ icon, set your endpoint URL / model, and chat.
Settings and chat history are stored in your browser's `localStorage` only —
nothing goes through a backend, so there's nothing to host besides this
static file.

## Deploying to GitHub Pages

1. Push this repo to GitHub.
2. Repo Settings → Pages → Source: deploy from branch → pick `main` (or
   whichever branch) and `/ (root)`.
3. Your page will be live at `https://<username>.github.io/<repo>/`.

## Important: this will not work out of the box

GitHub Pages serves your site over **HTTPS**. Your vLLM server in the curl
example is plain **HTTP**, and the IP (`100.76.19.104`) looks like a
Tailscale address, reachable only from devices on your tailnet. Two problems
follow:

### 1. Mixed content blocking
Browsers refuse to let an HTTPS page call an HTTP endpoint by default. When
you open the page over `https://...github.io`, the fetch to
`http://100.76.19.104:8000/...` will likely be blocked silently or with a
console error like "Mixed Content: ... was blocked".

Options, roughly in order of effort:
- **Easiest**: click the padlock/shield icon in the browser address bar and
  allow "insecure content" / "unsafe scripts" for this one site. Fine for
  personal use on a machine you control.
- **Better**: put a TLS reverse proxy (e.g. Caddy, or Tailscale's own HTTPS
  via `tailscale serve`) in front of vLLM so the endpoint is
  `https://100.76.19.104:8443/v1/chat/completions` or a proper HTTPS
  hostname. Then there's no mixed-content issue at all.

### 2. CORS
Even over HTTPS, the vLLM server needs to explicitly allow cross-origin
requests from `https://<username>.github.io`, or the browser will block the
response with a CORS error.

vLLM's OpenAI server supports CORS flags. Start it with something like:

```
vllm serve Qwen/Qwen2.5-7B-Instruct \
  --allowed-origins '["https://<username>.github.io"]' \
  ...your other flags
```

(Use `'["*"]'` to allow any origin — simplest for personal/tailnet-only use
since the IP isn't reachable from the public internet anyway.)

### 3. Reachability
Since `100.76.19.104` is a Tailscale IP, the page will only be able to reach
your model from a device that's also on your tailnet (or via `tailscale
serve`/Funnel if you want it public). That's expected — this isn't a public
API, just a browser UI for your own server.

## Files

- `index.html` — the whole app (HTML/CSS/JS, no dependencies).
