# Static site example

A plain-HTML folder served through portless — no package.json or framework needed.

## Run it

```bash
cd examples/static-site
portless demo npx -y serve .
# -> https://demo.localhost
```

Open https://demo.localhost in your browser. The proxy is HTTPS on port 443
(no certificate warning, because portless installs its own local CA on first run).
`npx -y serve` honors the injected `PORT`, so no `-l` flag is needed.

Stop it with `Ctrl+C`; the route is cleaned up automatically. See `portless list`
to confirm, or `portless hosts sync` if the hostname does not resolve (Safari).
