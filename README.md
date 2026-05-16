# search-anime skill

Search anime and manga by natural-language query using the ani CLI and AniList.

## Usage

```
/search-anime                                # trending digest (anime + manga)
/search-anime trending [--manga]             # what's trending right now
/search-anime popular [--season S --year Y]  # popular this (or a past) season
/search-anime upcoming                       # season preview
/search-anime top                            # top-rated of all time
/search-anime search <natural query>         # NL search with genre/score/year/format filters
/search-anime info <title or AniList id>     # full detail card with streaming links
/search-anime digest                         # full weekly briefing
/search-anime gems [--manga]                 # high-score, lower-profile titles
/search-anime binge                          # completed long-run series
```

**Natural search examples**

```
/search-anime search something like attack on titan
/search-anime search dark psychological thriller finished
/search-anime search action anime from 2023 with score above 85
/search-anime search mecha sci-fi from the 90s
/search-anime search isekai comedy
```

## What you get

Every mode produces beautifully formatted output with score emojis (🔥 ⭐ 👍), genre pills, and airing countdowns. The `info` command goes deeper:

- Synopsis, studio, top tags
- **Streaming links** — Crunchyroll, HIDIVE, and others pulled live from AniList
- **Community recommendations** — top 5 fan-voted "if you liked this" suggestions with scores
- Related works (sequels, prequels, adaptations)

**Requires:** [`ani`](https://github.com/pavelsimo/ani) on your PATH.

## Installation

```bash
npx skills@latest add pavelsimo/search-anime
```

## Contributing

Open an issue or pull request. Keep commits atomic.

## License

MIT
