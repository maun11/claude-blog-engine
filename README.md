# claude-blog-engine

End-to-end blog generation for Claude Code: onboard your business, discover keyword opportunities, and write full SEO articles with images and publishing checklists.

> ### ⚡ Want to skip all the setup and get the results directly?
> Use my paid tool and get it done for you — **[useindexly.com](https://useindexly.com/)**

## Install

```bash
# 1. Clone the repo
git clone https://github.com/maun11/claude-blog-engine.git ~/claude-blog-engine

# 2. Run the install script
bash ~/claude-blog-engine/install.sh

# 3. Restart Claude Code (or start a new session)
```

That's it. The install script copies 4 skills to `~/.claude/skills/` so they're available in any project.

## Skills

| Skill | What it does |
|-------|-------------|
| `/blog-hub` | Dashboard — shows your status and what to do next |
| `/blog-onboard [url]` | Scrapes your website, builds business profile, finds 3 competitors |
| `/blog-topics [market]` | Keyword research, clustering, scoring, and 10 topic picks |
| `/blog-write [topic\|number]` | Full SEO article with images, schema, and publishing checklist |

## Quick Start

```
/blog-hub                              ← see where you are
/blog-onboard https://yoursite.com     ← one-time setup
/blog-topics                           ← find 10 topics (defaults to US)
/blog-write                            ← write the top-scored article
/blog-write 3                          ← write topic #3 from your pipeline
/blog-write "marketing attribution"    ← write a specific topic by keyword
```

## Example Output

Here's a real article written with this engine:

**[7 Best Cloud ERP Software Tools for Supply Chain and Finance Teams
](https://predflow.ai/blog/cloud-erp-software-supply-chain-finance)**

> *"A practical guide to attribution models for D2C performance marketers. Learn which models work, which lie, and how to get numbers you trust."*

Generated from keyword `marketing attribution models` — includes title, meta description, structured sections, images, and a publish-kit checklist with schema markup.

---

## API Keys

When you run `/blog-onboard` for the first time, it will automatically create a `.env` file in your project root with placeholders. Open it, fill in your keys, and reply to continue.

```
# Required
DATAFORSEO_LOGIN=your@email.com        # dataforseo.com — free trial
DATAFORSEO_PASSWORD=yourpassword
ANTHROPIC_API_KEY=sk-ant-...           # console.anthropic.com

# Optional — improve article quality
FIRECRAWL_API_KEY=fc-...               # firecrawl.dev — better page scraping
TAVILY_API_KEY=tvly-...                # tavily.com — deeper topic research
YOUTUBE_API_KEY=AIza...                # console.cloud.google.com — video insights
OPENAI_API_KEY=sk-...                  # platform.openai.com — DALL-E article images
```

Only `DATAFORSEO` + `ANTHROPIC` are required. Optional keys enhance `/blog-write` — it works fine without them.

You can also copy `.env.example` from this repo as a starting point.

See [docs/apis.md](docs/apis.md) for detailed setup and costs.

## How It Works

Three engines run sequentially. Each one feeds the next.

---

### Engine 1 — Onboarding (`/blog-onboard`)

Runs once. Builds your business profile and finds competitors.

```
Firecrawl / WebFetch          Claude Haiku                DataForSEO SERP
  Scrape website         →   Extract business profile  →   Find competitors
  /about, /pricing,          product type, ICP,             search "{product} software"
  /features, /product        differentiator, voice          filter to 3 direct competitors
```

Outputs: `.claude/blog-config.json` with business profile + 3 competitor domains.

---

### Engine 2 — Site Intelligence (`/blog-topics`)

Runs weekly. Builds your full keyword universe and picks 10 topics.

```
DataForSEO                    Claude Haiku                DataForSEO
  Your rankings (500)    +    Generate 30 seeds      →   Expand each seed (×30 parallel)
  Competitor keywords         based on ICP + pain         30 keywords per seed = ~900
  (3 × 200 = 600)             points + integrations

DataForSEO bulk KD  →  Merge + dedup (~2000)  →  Filter by volume / KD / ranking
                                                   ~300 actionable keywords remain

Claude Haiku (parallel batches)    Claude Sonnet             Scoring formula
  Classify funnel stage       →    Group into            →   opportunity_score =
  TOFU / MOFU / BOFU               6–10 clusters             log(volume) × difficulty × funnel
  50 keywords per call             one pillar per cluster    0–100 scale, 70+ = good

Claude Haiku
  Select 10 topics  →  Generate SEO titles  →  Save to pipeline
  one per cluster       one Haiku call for all
```

Outputs: `CONTENT-PLAN.md`, `.claude/blog-keywords.json`, `.claude/blog-clusters.json`, `exports/*.csv`

---

### Engine 3 — Content Engine (`/blog-write`)

Runs per article. Research → outline → full article → images → schema.

```
Step 1  Context Assembly
        Pull keyword record + cluster + business profile into one master context object

Step 2  Live Research (4 parallel)
  2a  DataForSEO advanced SERP  →  Firecrawl scrape top 3 articles
      extract H2 structures, PAA questions, avg word count
  2b  Tavily batch search (3 queries)
      recent news · expert opinion · common mistakes
  2c  YouTube Data API  →  transcript via Apify  →  Claude Haiku extract 2 insights

Step 3  SERP Gap Analysis — Claude Haiku
        What are top 3 articles missing? Best featured snippet opportunity?

Step 4  Outline Generation — Claude Sonnet
        Full section blueprint: H2s, H3s, word counts, image placements,
        product plug position, research assignments, CTA framing

Step 5  Article Generation — Claude Sonnet
        Full article in one shot following the outline exactly
        Image placeholders {{IMAGE_THUMBNAIL}} {{IMAGE_MID_ARTICLE}} embedded at spec'd positions

Step 6  Image Generation — Claude Haiku + DALL-E 3
        Haiku reads written content → generates DALL-E prompt + alt text
        Both images generated in parallel, downloaded immediately

Step 7  Image placeholder replacement (code only)

Step 8  Schema markup — Claude Haiku
        Article schema + FAQPage schema from FAQ section

Step 9  Meta assets — Claude Haiku
        meta_title · meta_description · social_excerpt
```

Outputs: `blog-posts/{date}-{slug}/article.md`, `publish-kit.md`, `images/`

---

### Shared State

All commands read/write `.claude/blog-config.json`. The pipeline tracks every queued, in-progress, and written topic — re-running `/blog-topics` always surfaces fresh keywords, never repeats.

### Output Files

| File | Purpose |
|------|---------|
| `CONTENT-PLAN.md` | Human-readable pipeline tracker — open anytime |
| `.claude/exports/*.csv` | Full keyword dataset — open in Excel or Google Sheets |
| `blog-posts/*/article.md` | Pure article, images embedded at correct positions |
| `blog-posts/*/publish-kit.md` | Publishing checklist — meta, schema JSON, SEO checks |
| `blog-posts/*/images/` | Thumbnail + mid-article images (requires `OPENAI_API_KEY`) |

## Market Targeting

```
/blog-topics            US (default)
/blog-topics uk         United Kingdom
/blog-topics in         India
/blog-topics au         Australia
/blog-topics ca         Canada
/blog-topics de         Germany
/blog-topics sg         Singapore
```

## Data Privacy

Everything is stored locally in your project. Only the API calls you configure touch external services. Add to `.gitignore`:

```
.claude/blog-config.json
.claude/blog-keywords.json
.claude/blog-clusters.json
.claude/exports/
blog-posts/
```

## Uninstall

```bash
rm -rf ~/.claude/skills/blog-hub
rm -rf ~/.claude/skills/blog-onboard
rm -rf ~/.claude/skills/blog-topics
rm -rf ~/.claude/skills/blog-write
rm -rf ~/.claude/blog-scripts
```

## Repo Structure

```
claude-blog-engine/                  ← this repo
├── .claude-plugin/
│   └── plugin.json                  ← plugin manifest
├── skills/
│   ├── blog-hub/SKILL.md            ← dashboard
│   ├── blog-onboard/SKILL.md        ← business profiling + competitors
│   ├── blog-topics/SKILL.md         ← keyword research + clustering
│   └── blog-write/SKILL.md          ← article generation
├── scripts/
│   └── topics_pipeline.py           ← deterministic data processing (12 subcommands)
├── docs/
│   └── apis.md                      ← API setup guide
├── install.sh
├── .env.example
└── README.md
```

## Generated Files (in your project)

```
your-project/
├── .claude/
│   ├── blog-config.json         ← business profile + pipeline
│   ├── blog-keywords.json       ← keyword data (after /blog-topics)
│   ├── blog-clusters.json       ← cluster map (after /blog-topics)
│   └── exports/
│       └── keywords-2026-04-10.csv
├── blog-posts/
│   └── 2026-04-10-your-topic/
│       ├── article.md
│       ├── publish-kit.md
│       └── images/
│           ├── thumbnail.png
│           └── mid-article.png
└── CONTENT-PLAN.md
```
