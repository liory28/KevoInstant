# Kevo Instant Approval — self-hosted deploy (GitHub + Netlify)

This folder is a complete, deployable copy of Kevo Instant Approval: the same chat-based
Non-QM pre-approval prototype that runs inside claude.ai, wired to talk to a real Anthropic
API key on your own domain instead of the claude.ai `sample` capability.

```
index.html                    the whole app (single self-contained file)
netlify/functions/chat.js     Netlify Function that holds the Anthropic API key server-side
netlify.toml                  routes /api/chat -> the function above
package.json
.env.example                  template for the ANTHROPIC_API_KEY variable
```

## Deploy steps (GitHub + Netlify)

1. **Unzip** this folder on your computer — `index.html`, `netlify.toml`, etc. should sit at
   the top level (not nested inside another folder).
2. **Create a GitHub repo** (e.g. `kevo-instant-approval`) and push this folder's contents to
   it — either via GitHub's "Add file -> Upload files" web UI, or via git from a terminal:
   ```
   git init
   git add .
   git commit -m "Kevo Instant Approval"
   git branch -M main
   git remote add origin <your repo URL>
   git push -u origin main
   ```
3. **Get an Anthropic API key** at console.anthropic.com (billed separately from any
   claude.ai subscription).
4. **Import the repo into Netlify**:
   - Go to app.netlify.com -> **Add new site -> Import an existing project**.
   - Choose GitHub, authorize if asked, and pick your `kevo-instant-approval` repo.
   - Netlify reads `netlify.toml` automatically, so the build settings (publish directory
     `.`, functions directory `netlify/functions`) are already set — you don't need to type
     anything into the build command / publish directory fields. Click **Deploy site**.
5. **Set the environment variable** — once the site exists, go to **Site configuration ->
   Environment variables -> Add a variable**, and add `ANTHROPIC_API_KEY` (paste the key from
   step 3). Optionally also add `ANTHROPIC_MODEL` if you want to pin a specific model.
6. **Redeploy** so the function picks up the new variable — **Deploys** tab -> **Trigger
   deploy -> Deploy site** (any push to `main` also triggers this automatically going
   forward).
7. **Open the site** — Netlify gives you a working `https://<random-name>.netlify.app` URL.
   Confirm the ask-bar and document upload work (they call this real backend now, not the
   claude.ai capability) — if you see a "server_misconfigured" message, double check step 5
   and that you redeployed after adding the variable.
8. **Point your real domain at it (optional)** — **Domain management -> Add a domain**, type
   your subdomain (e.g. `apply.kevomortgage.com`), and follow Netlify's DNS instructions (a
   CNAME to your `.netlify.app` address, or delegate the domain to Netlify DNS).
9. **Rename the site (optional)** — **Site configuration -> Change site name** swaps the
   random `<random-name>.netlify.app` for something like `kevo-instant-approval.netlify.app`
   before you point a custom domain at it, if you'd like a nicer link to share in the
   meantime.
10. **Re-deploying later** — push new commits to the same GitHub repo (or drag-and-drop
    updated files in the Netlify UI) and Netlify redeploys automatically within a minute or
    two. No new Netlify site needed.

## Notes

- The chat list in the nav rail is saved to each visitor's own browser (`localStorage`) —
  it is not shared across devices or synced anywhere, and nothing is sent to Kevo's servers
  beyond the live-AI calls themselves.
- `netlify/functions/chat.js`'s rate limiting is a light per-IP deterrent only, not
  production-grade abuse protection — worth hardening (Netlify's own rate limiting add-ons,
  or a Redis-backed one via Upstash) before pointing real borrower traffic at this beyond the
  broker network.
- Local testing: `npm install -g netlify-cli` once, then `netlify dev` from this folder runs
  the site and the function together on `localhost:8888` (it will prompt you to log in / link
  the site, and reads `ANTHROPIC_API_KEY` from a local `.env` file — copy `.env.example` to
  `.env` and fill in your key for local runs only; never commit `.env`).
- This is a prototype: property data, credit pulls, and bank connections are all simulated.
  See the in-app footer disclaimer and the project notes for the full "what's real vs.
  simulated" breakdown.
