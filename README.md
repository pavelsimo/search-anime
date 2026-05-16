# search-anime skill

A Claude Code skill that wraps the `ani` CLI. Ask for anime or manga in plain English and it runs the right command.

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

<details>
<summary>Claude Code</summary>

**Project-level** (one project):
```bash
cp SKILL.md .claude/commands/search-anime.md
```

**Global** (all projects):
```bash
cp SKILL.md ~/.claude/commands/search-anime.md
```

Invoke with `/search-anime`.

</details>

<details>
<summary>OpenAI Codex</summary>

Append `SKILL.md` contents to `AGENTS.md` (project) or `~/.codex/instructions.md` (global).

Invoke: `codex "search for trending anime"`

</details>

<details>
<summary>GitHub Copilot</summary>

Append `SKILL.md` contents to `.github/copilot-instructions.md`.

Invoke: open Copilot Chat and describe the task — "show me trending anime" or "search for dark psychological thrillers".

</details>

## Contributing

Open an issue or pull request. Keep commits atomic.

## License

MIT
