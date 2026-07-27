# AI Monetization Tracker

A self-hosted dashboard tracking the marginal signals of AI monetization, inspired by the format of [ai.castoramoney.com](https://ai.castoramoney.com/) and rebuilt from scratch in English:

- **Frontier Lab ARR** — live-ticking estimated ARR for Anthropic and OpenAI, modeled from publicly reported run-rate checkpoints (log-linear history + damped-growth extrapolation with an uncertainty fan)
- **Token usage & adoption** — OpenRouter daily tokens (per model family, lab share, top models), Vercel AI Gateway token/$-spend leaderboards, npm/PyPI SDK downloads
- **GPU rental prices** — Ornn Compute Price Index ($/GPU-hr, daily)
- **AI data-center buildout** — Epoch AI satellite/permit dataset + Google News feed

It is a **static site** (single `index.html` + ECharts from CDN + one generated `data/data.js`) refreshed by a **GitHub Actions** cron job. No server, no build step, no dependencies beyond Python 3 stdlib.

## Quick start (local)

```bash
python scripts/update_data.py   # generate data/data.js (works without any keys — see below)
python -m http.server 8000      # then open http://localhost:8000
```

Don't double-click `index.html` — browsers block `file://` data loading. Serve it as above.

On the first run without an OpenRouter key, the token-usage charts contain clearly flagged **SAMPLE data**; everything else (ARR model, GPU, data centers, Vercel, SDK, news) is real, fetched live.

## Put it on GitHub (with auto-refresh + free hosting)

```bash
cd ai-monetization-tracker
git init -b main
git add -A
git commit -m "AI monetization tracker"
gh repo create ai-monetization-tracker --public --source=. --push
```

(No GitHub CLI? Create an empty repo on github.com, then `git remote add origin <url> && git push -u origin main`.)

Then, in the repo settings on github.com:

1. **Actions secret** (optional but recommended): *Settings → Secrets and variables → Actions → New repository secret* → name `OPENROUTER_API_KEY`, value = a key from [openrouter.ai/keys](https://openrouter.ai/settings/keys) (free account works; the datasets API doesn't cost tokens).
2. **Run the workflow once**: *Actions → Update tracker data → Run workflow*. This replaces sample data with real OpenRouter data and commits it.
3. **GitHub Pages**: *Settings → Pages → Deploy from a branch → `main` / root*. Your dashboard is now live at `https://<you>.github.io/ai-monetization-tracker/` and refreshes daily (~13:17 UTC).

## Customizing

Everything lives in `config/tracker_config.json`:

- `arr.companies.*.checkpoints` — add newly reported run-rate figures as they're published; the model, Y/Y chip and live counter recalibrate automatically. `growth_damping` (how much of the recent growth pace the extrapolation assumes) and `fan_pct_per_month` (uncertainty band width) are tunable.
- `openrouter.watch_models` — the model families charted; `hero_model` gets its own panel.
- `sdk.npm` / `sdk.pypi` — packages tracked as adoption proxies.
- `news.query` / `news.pinned` — news feed query and manually pinned items.
- `signals.kol` — the manually curated quotes panel.

The front-end is one dependency-free `index.html`; colors and the retro window styling are CSS variables at the top of the file.

## Data sources & credits

| Section | Source | Access |
|---|---|---|
| Token usage | [OpenRouter Datasets API](https://openrouter.ai/docs/api/api-reference/datasets/get-rankings-daily) | API key (free) |
| Gateway share | [Vercel AI Gateway Leaderboards](https://vercel.com/ai-gateway/leaderboards/models) | public |
| SDK downloads | [npm API](https://api.npmjs.org) · [pypistats](https://pypistats.org) | public |
| GPU prices | [Ornn Compute Price Index](https://dashboard.ornnai.com/docs) | public |
| Data centers | [Epoch AI — AI Data Centers](https://epoch.ai/data/ai-data-centers) (CC-BY 4.0) | public |
| News | Google News RSS | public |
| ARR checkpoints | public press reports, editable in config | — |

ARR figures are unaudited run-rate estimates for research reference only — not investment advice. Token counts use each provider's own tokenizer and are not fully comparable across providers.
