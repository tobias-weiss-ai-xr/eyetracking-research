<h1 align="center">
  <strong>Eyetracking Research Corpus</strong>
</h1>
<h3 align="center">Comprehensive collection of eyetracking and gaze analysis research papers</h3>

### 🔗 Links

- **License**: https://github.com/tobias-weiss-ai-xr/eyetracking-research/blob/main/LICENSE
- **CI**: https://github.com/tobias-weiss-ai-xr/eyetracking-research/actions/workflows/validate.yml
- **GitHub Pages**: https://tobias-weiss-ai-xr.github.io/eyetracking-research/

> 👁️ **Domain-specific:** This repository contains a curated collection of research papers
> focused on eyetracking technology, gaze analysis, visual attention, and related topics.

## What you get

| Domain | Focus |
|--------|-------|
| **Eyetracking** | Gaze tracking, eye movement analysis |
| **Vision** | Visual perception, attention mechanisms |
| **HCI** | Human-computer interaction with gaze |
| **Psychology** | Cognitive processes, visual attention |
| **AI/ML** | Gaze prediction, attention modeling |

## What you get

| Capability | How |
|------------|-----|
| 📄 **Curated corpus** | `papers.yaml` is the source of truth — one structured entry per paper |
| ✅ **Auto-validation** | `scripts/validate_papers.py` checks schema, duplicates, URL normalization, LaTeX artifacts |
| 🧾 **Auto-generated README** | `scripts/generate_readme.py` renders the paper list grouped by your taxonomy |
| 📊 **Statistics & trends** | `scripts/standard_stats.py` → `statistics.json` (momentum, gaps, bursts, venues, authors) |
| 🔍 **Literature review report** | `scripts/analysis/generate_reports.py` → `docs/research/literature_review.md` + `trends.md` |
| 🧭 **Topic planning** | `tools/topic_planner.py`, `tools/trend_scanner.py`, `tools/landscape_analyzer.py`, `tools/brief_generator.py` |
| 🔎 **New paper discovery** | `scripts/fetch/fetch_new_papers.py` (arXiv), `fetch_other_sources.py` (dblp/crossref/europepmc), `fetch_openalex_bulk.py` |
| 🐙 **GitHub repos discovery** | `scripts/fetch/fetch_github_repos.py` (optional, config-driven via `github_queries` in taxonomy.yaml) |
| 🦊 **GitLab projects discovery** | `scripts/fetch/fetch_gitlab_repos.py` (optional, config-driven via `gitlab_queries` in taxonomy.yaml) |
| 🏠 **Codeberg repos discovery** | `scripts/fetch/fetch_codeberg_repos.py` (optional, config-driven via `codeberg_queries` in taxonomy.yaml) |
| 📰 **News / intelligence digest** | `run_pipeline.py` — RSS/Atom + arXiv + GitHub releases + catalogs (e.g. CISA KEV) → scored, de-duplicated daily digest (`data/latest.md`/`.html`/`.json`) |
| 🖥️ **GitHub Pages site** | `docs/index.html` — searchable, filterable paper browser |
| 🤖 **Agentic workflow** | `AGENTS.md` + `config/taxonomy.yaml` make this repo agent-friendly by design |

## 🚀 Jump-start (5 steps)

```bash
# 1. Clone and rename
git clone https://github.com/tobias-weiss-ai-xr/skeleton-research.git my-topic-research
cd my-topic-research
git remote set-url origin https://github.com/<YOUR_ORG>/my-topic-research.git  # repoint to your fork
cd my-topic-research

# 2. Define your topic & taxonomy
#    Edit config/taxonomy.yaml: topic name, categories, subcategories, queries
vim config/taxonomy.yaml

# 3. Seed your corpus (start small — 5-10 papers is fine)
#    Either hand-curate papers.yaml, or auto-discover:
python3 scripts/fetch/fetch_new_papers.py --months 12 --dry-run   # preview arXiv hits
python3 scripts/fetch/fetch_new_papers.py --local                 # append to papers.yaml

# 4. Validate + generate
python3 scripts/validate_papers.py
python3 scripts/generate_readme.py
python3 scripts/standard_stats.py
python3 scripts/analysis/generate_reports.py

# 5. Commit & let CI keep it healthy
git add -A && git commit -m "bootstrap corpus for <YOUR TOPIC>"
git push
```

## 📖 How it works

```
config/taxonomy.yaml ──► papers.yaml ──► validate_papers.py
                          │   ▲              │
                          ▼   └── fetch_* ───┘
                   generate_readme.py ──► README.md paper list (auto)
                          │
                          ▼
                  standard_stats.py ──► statistics.json, docs/papers.json,
                                        README.md corpus statistics (auto)
                          │
                          ▼
              analysis/generate_reports.py ──► docs/research/*.md
```

The generated README sections (paper list + corpus statistics) are
**marker-delimited** (`<!-- BEGIN … -->` … `<!-- END … -->`) and are owned by
the pipeline: `generate_readme.py` and `standard_stats.py` regenerate exactly
their section on every run. Everything else in the README is user-owned prose
and is left untouched. If a repo drops a section entirely (e.g. the paper list
lives on the GitHub Pages site), the owning script skips it gracefully instead
of erroring.

- **Never edit the generated README sections by hand** — run the pipeline.
- The **taxonomy lives in one place** (`config/taxonomy.yaml`); every script reads it via `scripts/research_config.py`, which now validates the config up front so mistakes fail loudly.
- **CI (validate.yml)** runs on every push/PR and weekly to discover new papers. The `validate` job re-checks that all generated outputs are fresh (README, statistics, reports), and a `test` job runs the pytest suite.

## 🧪 Local pipeline (all in one)

```bash
make all          # validate → check freshness → generate → test
# …or run the raw steps:
python3 scripts/validate_papers.py && \
python3 scripts/generate_readme.py && \
python3 scripts/standard_stats.py && \
python3 scripts/analysis/generate_reports.py

# Freshness checks (non-destructive; exit 1 if stale) — used by CI
python3 scripts/generate_readme.py --check
python3 scripts/standard_stats.py --check
python3 scripts/analysis/generate_reports.py --check

# Unit tests
python3 -m pytest
```

## 📰 News / Intelligence Digest (`run_pipeline.py`)

A complementary pipeline that ingests **RSS/Atom feeds**, **arXiv queries**,
**GitHub release feeds**, and **structured catalogs** (e.g. CISA KEV), then
classifies + scores each item, de-duplicates against a seen-history, and renders
a daily digest (`data/latest.md`, `data/latest.html`, `data/raw.json`).

```bash
python3 run_pipeline.py --dry-run   # ingest + score only (no writes)
python3 run_pipeline.py              # full run → writes digest + marks seen
python3 run_pipeline.py --top 30     # smaller digest
```

Sources, weights and keyword classification live in `config/sources.yml`.

### Anti-saturation controls

Without guards, static catalogs (CISA KEV's 1,600+ entry list) and full-archive
feeds (Snyk's 1,600+ post history) recycle old items every run, and one
high-volume feed (e.g. a project blog) can crowd out everything else. Three
config knobs in `config/sources.yml` prevent this:

| Knob | Effect |
|------|--------|
| `recent_days` (per source) / `default_recent_days` | Drop items published/added older than N days. Applied to RSS + catalog sources; **opt in only on archive/catalog feeds** — normal blogs already return recent items, so leave the default at `0` to avoid starving low-frequency, high-signal feeds. |
| `max_items` (per source) | Hard cap on raw items kept from one source (e.g. 15 for an archive feed). |
| `max_source_share` (global) | Hard ceiling on how many digest slots a single source may occupy (default `0.40` → no source takes >40% of the digest). |

> Why `default_recent_days: 0`? Only archive/catalog feeds recycle. A blanket
> window would silently drop infrequent but high-signal feeds (e.g. Project
> Zero, which posts monthly). Set `recent_days` explicitly on the feeds that
> need it.

## 🔎 Discovery & utility scripts

Beyond the core pipeline, several scripts remain available for manual / scheduled use:

| Script | What it does |
|---|---|
| `scripts/fetch/fetch_new_papers.py` | arXiv discovery; `--create-pr` opens a weekly PR (used by CI) |
| `scripts/fetch/fetch_openalex_bulk.py` | OpenAlex bulk discovery per category (`--months`, `--local`) |
| `scripts/fetch/fetch_other_sources.py` | dblp / crossref / Europe PMC / Semantic Scholar discovery |
| `scripts/fetch/fetch_metadata.py` | backfill authors/abstracts/venues for existing arXiv papers |
| `scripts/fetch/saturate_papers.py` | expand queries & loop arXiv until corpus saturates |
| `scripts/fetch/fetch_github_repos.py` / `fetch_gitlab_repos.py` / `fetch_codeberg_repos.py` | discover topic-relevant code repos → `repos.yaml` |
| `scripts/fetch/search_arxiv_html.py` / `search_arxiv_offline.py` | alternate/ad-hoc arXiv search helpers |
| `scripts/export_bibtex.py` | write `paper/references.bib` from `papers.yaml` |
| `scripts/visualize_statistics.py` | visualise `statistics.json` |

The repo-discovery fetchers share rate-limit/backoff + relevance logic in `scripts/fetch/repos_common.py`.

## 🤖 Agentic workflow (AGENTS.md)

This repo is designed to be driven by coding agents (OpenCode, Claude Code, …):

- **Spec-style guardrails** in `AGENTS.md` — agents know the pipeline, never edit README, always re-validate.
- **One config file** to change → one re-run to verify (low context cost for agents).
- **Auto-validation** gives agents an objective pass/fail signal.
- **Weekly discovery** keeps the corpus fresh without human babysitting.

<!-- BEGIN PAPER LIST -->

## 📚 Paper list

- [📚 Methods & Architectures](#methods-&-architectures)
  - [Agentic](#agentic)
- [📚 Applications](#applications)
  - [Non-Agentic](#non-agentic)
- [📚 Evaluation & Benchmarks](#evaluation-&-benchmarks)
  - [Hybrid](#hybrid)
- [📚 Surveys & Taxonomies](#surveys-&-taxonomies)
  - [Non-Agentic](#non-agentic)
  - [Hybrid](#hybrid)

### Methods & Architectures

#### Agentic

##### 2026

- [2026] **Example Paper 2: An Agentic Method for Your Topic** [[paper](https://arxiv.org/abs/2603.00002)]

[⬆ Back to top](#paper-list)

### Applications

#### Non-Agentic

##### 2025

- [2025] **Example Paper 3: Application Study in Your Domain** [[paper](https://arxiv.org/abs/2511.00003)]

[⬆ Back to top](#paper-list)

### Evaluation & Benchmarks

#### Hybrid

##### 2025

- [2025] **Example Paper 4: An Evaluation Benchmark for Your Topic** [[paper](https://arxiv.org/abs/2508.00004)]

[⬆ Back to top](#paper-list)

### Surveys & Taxonomies

#### Non-Agentic

##### 2025

- [2025] **Example Paper 5: A Survey of Your Topic Across Domains** [[paper](https://arxiv.org/abs/2505.00005)]

[⬆ Back to top](#paper-list)

#### Hybrid

##### 2026

- [2026] **Example Paper 1: A Foundational Survey of Your Topic** [[paper](https://arxiv.org/abs/2601.00001)]

[⬆ Back to top](#paper-list)

<!-- END PAPER LIST -->

<!-- BEGIN CORPUS STATISTICS -->

## 📊 Corpus Statistics

**1184 papers** across **9 categories**.  
Sources: **arXiv** 1161 (98%).  

### Top categories

| Category | Papers | Recent | |
|----------|--------|--------|-|
| ml-ai | **353** | 111 | ████████████ |
| psychology | **210** | 80 | ███████░░░░░ |
| applications | **194** | 74 | ███████░░░░░ |
| algorithms | **163** | 54 | ██████░░░░░░ |
| hardware | **119** | 42 | ████░░░░░░░░ |
| vr-ar | **75** | 29 | ███░░░░░░░░░ |
| hci | **62** | 27 | ██░░░░░░░░░░ |
| survey | **6** | 1 | █░░░░░░░░░░░ |
| evaluation | **2** | 0 | █░░░░░░░░░░░ |

### By year

| Year | Papers | |
|------|--------|-|
| 2002 | 1 | █░░░░░░░░░░░ |
| 2004 | 1 | █░░░░░░░░░░░ |
| 2005 | 1 | █░░░░░░░░░░░ |
| 2006 | 1 | █░░░░░░░░░░░ |
| 2009 | 1 | █░░░░░░░░░░░ |
| 2012 | 3 | █░░░░░░░░░░░ |
| 2014 | 2 | █░░░░░░░░░░░ |
| 2015 | 3 | █░░░░░░░░░░░ |
| 2016 | 1 | █░░░░░░░░░░░ |
| 2017 | 3 | █░░░░░░░░░░░ |
| 2018 | 4 | █░░░░░░░░░░░ |
| 2019 | 3 | █░░░░░░░░░░░ |
| 2020 | 1 | █░░░░░░░░░░░ |
| 2021 | 2 | █░░░░░░░░░░░ |
| 2022 | 3 | █░░░░░░░░░░░ |
| 2023 | 86 | ██░░░░░░░░░░ |
| 2024 | 377 | ██████████░░ |
| 2025 | 437 | ████████████ |
| 2026 | 254 | ███████░░░░░ |

### Momentum (hottest categories)

| Category | Total | Rate | Recent | Score |
|----------|-------|------|--------|-------|
| Hci | 62 | 2.2/mo | 44% | 78 |
| Vr Ar | 75 | 2.4/mo | 39% | 60 |
| Psychology | 210 | 6.7/mo | 38% | 43 |
| Hardware | 119 | 3.5/mo | 35% | 31 |
| Applications | 194 | 6.2/mo | 38% | 28 |
| Algorithms | 163 | 4.5/mo | 33% | 23 |
| Ml Ai | 353 | 9.2/mo | 31% | 19 |
| Survey | 6 | 0.1/mo | 17% | 17 |
| Evaluation | 2 | 0.0/mo | 0% | 0 |

### Trending keywords

| Keyword | Papers | Burst |
|---------|--------|-------|
| oculomotor | 8 | 2.27 |
| gaze-based interface | 1 | 1.42 |
| saccade | 10 | 1.35 |
| scanpath | 12 | 1.31 |
| fixation | 31 | 1.22 |
| visual attention | 23 | 1.18 |
| foveated vision | 4 | 1.13 |
| transformer attention | 3 | 1.06 |

### Top venues

| Venue | Papers |
|-------|--------|
| CVPR 2026 | 3 |
| ECCV 2024 | 3 |
| CVPR 2024 | 2 |
| ICLR 2025 | 2 |
| TMLR | 1 |
| IROS 2024 | 1 |
| ICML 2026 | 1 |
| ICLR 2024 | 1 |
| CVPR 2024 Gaze workshop | 1 |
| CVPR 2025 | 1 |

### Research gaps (thinnest cells)

| Cell | Papers |
|------|--------|
| `ml-ai/gaze-estimation` | 1 |
| `vr-ar/foveated-rendering` | 1 |
| `psychology/visual-perception` | 1 |
| `applications/marketing` | 1 |
| `survey/state-of-the-art` | 1 |

*Generated 2026-08 by `scripts/standard_stats.py`.*

<!-- END CORPUS STATISTICS -->

## 📖 Citation

If you use this skeleton for a project, please cite:

```bibtex
@misc{skeleton-research,
  author = {Weiß, Tobias},
  title = {Research Corpus Skeleton: Data-Driven Agentic Literature Review},
  year = {2026},
  publisher = {GitHub},
  url = {https://github.com/tobias-weiss-ai-xr/skeleton-research}
}
```

## 📄 License

MIT — see [LICENSE](LICENSE).
