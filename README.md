# Commit Galaxy 🚀

How far did your commits take you this year? Commit Galaxy turns a year of GitHub contributions into a rocket journey: launch from Earth, pass the clouds, the Kármán line, the ISS, the Moon, Mars and Jupiter, and with enough fuel reach Andromeda and the edge of the observable universe.

- **README card:** an animated SVG you can embed in your GitHub profile. It refreshes on its own.
- **Share page:** `commitsto.space/<username>` plays the journey and has one-click actions. Swap `github.com` with `commitsto.space` in any profile URL. Past years: `commitsto.space/<username>/2025`.
- **LinkedIn:** share the link (a 1200×630 PNG preview card is generated), or download an MP4 video and post it natively.

```md
[![Commit Galaxy](https://commitsto.space/card/alperen-selcuk.svg?year=2026)](https://commitsto.space/alperen-selcuk)
```

## How it works

```
                      ┌──────────────── Cloudflare Worker ────────────────┐
GitHub GraphQL  ───▶  │ getData() ── cached 4h (current year) / 7d (past) │
                      │      │                                            │
                      │      ├─▶ /card/:user.svg   animated SVG (SMIL+CSS)│──▶ GitHub README (camo)
                      │      ├─▶ /og/:user.png     resvg-wasm → PNG       │──▶ LinkedIn / X preview
                      │      ├─▶ /:user[/:year]    share page (OG tags)   │
                      │      └─▶ /api/data/:user   JSON                   │
                      └───────────────────────────────────────────────────┘
                                        │
                     browser (/app.js): same scene engine renders 1080×1080
                     frames → WebCodecs H.264 → mp4-muxer → .mp4 download
```

One scene engine (`src/scene.ts`) produces every output:

- `mode: "animated"` builds a self-playing SVG. It uses SMIL and CSS only, with no JavaScript, so it plays inside `<img>` on GitHub.
- `mode: "frame", t` builds the exact scene at time `t`. That gives frame-accurate video in the browser and the static OG image on the server.

The metric is the yearly **contribution total**: commits, PRs, issues and reviews, plus private contributions if the user enabled them on their profile.

| Contributions | Milestone | Contributions | Milestone |
|---:|---|---:|---|
| 5 | Clouds | 1,300 | Neptune |
| 25 | Stratosphere | 1,500 | Pluto & Kuiper Belt |
| 50 | Kármán Line | 1,800 | Interstellar Space |
| 100 | Space Station | 2,200 | Oort Cloud |
| 200 | The Moon | 2,700 | Alpha Centauri |
| 350 | Mars | 3,300 | Orion Nebula |
| 500 | Asteroid Belt | 4,000 | Milky Way Core |
| 700 | Jupiter | 5,000 | Andromeda |
| 900 | Saturn | 6,500 | Laniakea Supercluster |
| 1,100 | Uranus | 8,500 | Edge of the Universe |

Thresholds live in `MILESTONES` in `src/scene.ts`.

## Run locally

```bash
npm install
npm run dev            # http://localhost:8787 — .dev.vars has DEMO_FALLBACK=1 (fake data without a token)
```

Try `/demo?demo=1340` or `/card/demo.svg?demo=4200`; `?demo=N` fakes any total.

Put a real token in `.dev.vars` (`GITHUB_TOKEN=...`) to see real data.

`npm run preview -- <login> <total>` renders PNG frames of the scene into `out/`.

## Deploy (Cloudflare Workers)

```bash
npx wrangler login
npx wrangler secret put GITHUB_TOKEN   # fine-grained PAT, public read-only access is enough
npm run deploy
```

`wrangler.toml` binds `commitsto.space` and `www.commitsto.space` as custom domains, and `www` redirects to the apex. The `commitsto.space` zone must be added to the same Cloudflare account first (registrar nameservers → Cloudflare). Share links and the README snippet use whatever host serves the request.

**Limits:**

- The GraphQL API allows 5,000 points/hour per token, and every user/year result is cached at the edge.
- The Worker bundle is about 1 MB gzipped (resvg wasm + subsetted Inter), so it fits the free plan.

## Notes

- GitHub caches README images through camo. The card sends `Cache-Control: max-age=14400`, so expect a few hours of lag.
- Video export needs WebCodecs H.264: recent Chrome, Edge or Safari.
- `resvg-wasm` panics on off-canvas layers (element `opacity`, `clip-path`) and on text inside `opacity` groups. The world layer therefore uses `fill-opacity`/`stroke-opacity`, and frame mode avoids group opacity. Keep that in mind when adding art.
- Fonts: Inter (SIL OFL), subset to Latin + Turkish characters (`fonts/`).
# commitsto-space
# commitsto-space
