---
name: search-anime
description: natural-language anime and manga assistant powered by the ani CLI and AniList — browse trending, discover hidden gems, search by vibe, and find where to stream each title
trigger: /search-anime
---

# search-anime skill

A Claude Code skill that gives anime fans a natural-language interface on top of the `ani` CLI tool. Ask in plain English and get beautifully formatted results: trending digests, hidden gem hunts, season previews, smart similarity searches, and rich detail cards — including community recommendations and streaming links (Crunchyroll, HIDIVE, and more) — all backed by AniList's live GraphQL API.

## features

- **natural language → command translation** — "what dark psychological anime finished in 2023?" becomes the right `ani search` invocation automatically
- **trending digest** — aggregates trending anime + manga into a curated summary with scores, genre pills, airing countdowns, and a pick-of-the-week
- **streaming links** — every `info` deep-dive surfaces clickable links to Crunchyroll, HIDIVE, and other providers; digest mode includes streaming info for the top pick
- **community recommendations** — the `info` card shows the top 5 community-voted "if you liked this, watch that" suggestions, each with score and format
- **hidden gems** — cross-references top-50 by score with popularity; surfaces high-score titles most fans haven't discovered yet
- **season preview** — combines upcoming + currently popular into a "what to watch this season" guide with airing schedules
- **smart similarity search** — "like Attack on Titan" looks up AoT's genres, then searches for similar titles with matching filters
- **manga mode** — every mode works for manga via `--manga`; trending, gems, search, and info all work the same way

## usage

```
/search-anime                                # default: trending digest (anime + manga)
/search-anime trending                       # currently trending anime
/search-anime trending --manga               # currently trending manga
/search-anime popular                        # popular this season
/search-anime popular --season winter --year 2024
/search-anime upcoming                       # season preview: upcoming + current popular
/search-anime top                            # top-rated of all time
/search-anime all-time                       # most-watched of all time
/search-anime search <natural query>         # smart NL search
/search-anime info <title or AniList id>     # full detail card with streaming links
/search-anime digest                         # full weekly briefing
/search-anime gems                           # hidden gems: high score, lower popularity
/search-anime gems --manga                   # hidden gem manga
/search-anime binge                          # completed series worth binging (24+ eps)
```

**natural search examples**

```
/search-anime search action anime from 2023 with score above 85
/search-anime search something like attack on titan
/search-anime search dark psychological thriller finished
/search-anime search slice of life romance with high ratings
/search-anime search isekai comedy
/search-anime search mecha sci-fi from the 90s
```

## workflow

### 0. prerequisite check

Before any command, verify `ani` is installed:
```bash
which ani
```
If not found, stop and tell the user:
> `ani` is not installed. Install it from https://github.com/pavelsimo/ani — `go install github.com/pavelsimo/ani@latest` or grab a release binary.

### 1. mode detection

Parse the first argument:

| Argument | Mode |
|---|---|
| (none) or `digest` | digest |
| `trending` | trending |
| `popular` | popular |
| `upcoming` | preview |
| `top` | top |
| `all-time` | all-time |
| `search <query>` | search |
| `info <name-or-id>` | info |
| `gems` | gems |
| `binge` | binge |

`--manga` flag anywhere → append `--type manga` to all `ani` calls for that invocation. `--lang english` → append `--lang english` to all calls. Derive the current season from today's month: Jan–Mar = winter, Apr–Jun = spring, Jul–Sep = summer, Oct–Dec = fall.

### 2. trending mode

```bash
ani trending --json --per-page 20
# with --manga:
ani trending --json --per-page 20 --type manga
```

Parse the JSON array. Present as a markdown table with columns: `#`, title (bold), score (with emoji), genre pills (backtick-wrapped), format, airing status. For RELEASING with `nextAiringEpisode`, show `Airing (ep N in Xd Yh)` — convert `timeUntilAiring` seconds to days/hours. For FINISHED, show `Finished (N eps)`. For NOT_YET_RELEASED, show `Upcoming`. Score emoji: 90+ → 🔥, 80–89 → ⭐, 70–79 → 👍, 60–69 → 📊, 0–59 → number only, 0 → `—`.

After the table: "Pick of the Week" — 2-sentence note on the highest-scored currently airing title. Footer: `💡 Run /search-anime info <title> to see streaming links (Crunchyroll, HIDIVE, etc.)`

### 3. popular mode

```bash
ani popular --json --per-page 20
# with options:
ani popular --json --per-page 20 --season winter --year 2024
```

Same table format. Title: `## Popular This Season — Spring 2026` (or specified season/year). Add "Season Snapshot": count genres across all results, note the dominant 2–3 genres. Footer with streaming tip.

### 4. preview mode (upcoming)

Run two commands:
```bash
ani upcoming --json --per-page 10
ani popular --json --per-page 10
```

Merge results, deduplicate by `id`. Present in two sections:
- **Now Airing (Worth Starting)** — popular titles currently airing, sorted by score
- **Coming Soon** — upcoming titles; show `startDate` as `YYYY-MM-DD` if available, otherwise `Date TBD`; show `—` for score if `averageScore = 0`

### 5. top mode

```bash
ani top --json --limit 20
```

Trophy-list format (not a table):
```
🥇 **Title** — 🔥 91 · 28 eps · `Adventure` `Drama` `Fantasy`
🥈 **Title** — 🔥 91 · Movie · `Action` `Comedy`
🥉 **Title** — 🔥 90 · 64 eps · `Action` `Adventure` `Drama`
4. **Title** — 🔥 90 · …
```
Medals for 1–3; numbers for 4+. Footer with streaming tip.

### 6. all-time mode

```bash
ani all-time --json --per-page 20
```

Table format. Footnote: "Popularity reflects total AniList members — not the same as score."

### 7. search mode (natural language)

**Step A — extract structured filters** from the query:

| Signal | Filter |
|---|---|
| Genre name (see genre table) | `--genre <name>` (repeatable) |
| "score above N" / "min score N" / "rated N+" | `--min-score N` |
| Year digits 19xx–20xx or "in YYYY" / "from YYYY" | `--year YYYY` |
| Season word + year context | `--season <s>` |
| "tv show" / "series" | `--format tv` |
| "movie" | `--format movie` |
| "ova" | `--format ova` |
| "ona" / "web" | `--format ona` |
| "finished" / "completed" | `--status finished` |
| "airing" / "ongoing" | `--status airing` |
| "upcoming" / "announced" | `--status upcoming` |

For genres: use loose matching — "psychological thriller" → `--genre Psychological --genre Thriller`; "romcom" → `--genre Romance --genre Comedy`; "dark fantasy" → `--genre Fantasy`.

**Step B — similarity flow** ("like X" / "similar to X" / "reminds me of X"):
1. Extract the reference title
2. `ani search "<reference title>" --json --per-page 1`
3. If found: extract its `genres` array; build a new search using those genres + any other extracted filters + `--min-score 75`; exclude the reference title from results
4. If not found: tell the user and ask them to confirm the title or provide an AniList ID

**Step C — build and run the command**:
```bash
ani search "<remaining query terms>" [--genre X] [--year Y] [--season S] [--format F] [--status T] [--min-score N] --json --per-page 10
```
If no query terms remain after extraction and no filters were found either, ask the user to clarify.

**Step D — present results**:
```
## Search Results — "dark psychological thriller finished"
*Interpreted as: genres Psychological + Thriller, status finished*

| # | Title | Score | Genres | Format | Year |
|---|-------|-------|--------|--------|------|
| 1 | **Monster** | ⭐ 87 | `Mystery` `Psychological` `Thriller` | TV | 2004 |
```

Always show the interpreted filters in italics so the user can correct misinterpretations. On zero results: suggest removing one filter at a time.

### 8. info mode

**Step A — resolve identifier**:
- Positive integer → use as AniList ID directly
- Title string → `ani search "<title>" --json --per-page 1`; take the top result's `id`; if ambiguous, show a mini disambiguation list and ask the user to pick

**Step B — fetch details**:
```bash
ani info <id> --json
```

**Step C — present deep-dive card**:
```markdown
## Frieren: Beyond Journey's End
*葬送のフリーレン · Sousou no Frieren*

**Type:** TV · 28 episodes · Finished
**Score:** 🔥 91 / 100 · **Popularity:** 428,587 members
**Season:** Fall 2023 · **Studio:** Madhouse
**Genres:** `Adventure` `Drama` `Fantasy`
**Tags:** Elf · Time Skip · Magic · Redemption *(top 5 non-spoiler tags by rank)*

### Synopsis
<description — already plain text since asHtml: false in the query; truncate to ~300 chars with "…">

### Where to Watch
[Crunchyroll](https://crunchyroll.com/...) · [HIDIVE](https://hidive.com/...)
```
Filter `externalLinks` to `type = "STREAMING"`, render each as `[site](url)` joined by ` · `. If none found: "No streaming links on AniList — check [Livechart](https://www.livechart.me) or [AniList](https://anilist.co/anime/<id>)."

```markdown
### Community Recommendations
If you liked this, fans also recommend:
1. **Mushishi** — ⭐ 87 · TV · Finished *(342 votes)*
2. **Made in Abyss** — 🔥 90 · TV · Finished *(298 votes)*
```
The `recommendations` array is already sorted `RATING_DESC` by the query (up to 10 entries). Filter out entries where `mediaRecommendation` is null. Show top 5. Format: `N. **[title.english or title.romaji]** — [score emoji] [score] · [format] · [status] *([rating] votes)*`. Skip entries with `averageScore = 0`.

```markdown
### Relations
- **Sequel:** Frieren Part 2 (TV · Not Yet Released)
- **Source:** Frieren (Manga · Releasing)
```
Show each `relationType` + `node.title.romaji or english` + `(type · format · status)`.

### 9. digest mode

Announce: "Fetching your weekly anime digest…" then run:
```bash
ani trending --json --per-page 5
ani trending --json --per-page 5 --type manga
ani popular --json --per-page 5
ani upcoming --json --per-page 5
ani info <top_trending_id> --json   # for Pick of the Week streaming links
```

Present:
```
# Anime & Manga Weekly Digest — Week of [current date]

## Trending Anime
[top 5 table]

## Trending Manga
[top 5 table]

## Popular This Season ([Season Year])
[top 5 table — deduplicated against trending]

## Coming Soon
[top 5 upcoming — show startDate or "TBD"]

---
## This Week's Pick
**[highest-scored airing title]**
[1–2 sentence editorial note on why it stands out]
**Where to Watch:** [Crunchyroll](url) · [HIDIVE](url)

## Genre Pulse
This season is dominated by: `Fantasy` `Action`
*(most common genres across trending + popular)*
```

Deduplicate: if a title appears in both trending and popular, show it only in trending.

### 10. gems mode

```bash
ani top --json --limit 50
# with --manga:
ani top --json --limit 50 --type manga
```

Filter client-side: `averageScore >= 85` AND `popularity < 150000`. Sort: score desc, then popularity asc (equal scores → prefer less popular). Show top 10:

```
## Hidden Gems — High Score, Lower Profile
*Titles that scored 85+ but fly under the radar*

| # | Title | Score | Popularity | Format | Genres |
|---|-------|-------|-----------|--------|--------|
| 1 | **3-gatsu no Lion** | ⭐ 89 | 126K | TV | `Drama` `Slice of Life` |
```

Add "Gem of the Day" callout: the single title with the best score-to-popularity ratio, 2-sentence editorial note.

### 11. binge mode

```bash
ani search --status finished --min-score 80 --format tv --json --per-page 20
```

Filter client-side: keep only entries where `episodes >= 24`. Sort by `averageScore` desc. Present:

```
## Binge-Worthy Completed Series
*Long-run finished anime — safe to start without waiting for more episodes.*

| Title | Score | Episodes | Genres |
|-------|-------|----------|--------|
| **Hunter x Hunter (2011)** | ⭐ 89 | 148 | `Action` `Adventure` `Fantasy` |
```

### 12. error handling

| Situation | Response |
|---|---|
| `ani` not on PATH | Stop. Show install instructions: https://github.com/pavelsimo/ani |
| Empty JSON array `[]` | "No results found. Try [specific filter to loosen]." |
| Invalid season name | List valid options: `winter`, `spring`, `summer`, `fall` |
| Invalid format | List valid options: `tv`, `movie`, `ova`, `ona`, `special` |
| `info` — title search returns no results | Ask user to confirm spelling or provide the AniList ID |
| `nextAiringEpisode` is null but status RELEASING | Show `Airing · schedule TBD` |
| `averageScore = 0` | Show `—` |
| `mediaRecommendation` is null | Skip that recommendation entry silently |
| Network or API error | Surface the raw error message; suggest retrying |

## best practices

- **always use `--json`** — never parse the human-readable table output
- **always show interpreted filters** after NL search so the user can correct them
- **deduplicate by `id`** in digest and preview modes — never show the same title twice
- **cap per-page at 50** — that is the AniList API maximum
- **convert seconds to human time** — `timeUntilAiring` is in seconds; always display as "Xd Yh"
- **derive the current season dynamically** from today's month; never hardcode it
- **null-guard recommendations** — `mediaRecommendation` can be null if the entry was deleted; skip silently
- **streaming links are detail-only** — `externalLinks` only comes from `ani info`, not from list commands; surface a tip in list modes to run `/search-anime info <title>` for streaming links

## genre reference

Genre names must be passed to `--genre` in exact Title Case:

| Genre | Common pairings | Natural language aliases |
|-------|----------------|------------------------|
| Action | Adventure, Fantasy, Sci-Fi | "shonen", "fight", "battle" |
| Adventure | Fantasy, Action | "quest", "journey" |
| Comedy | Slice of Life, Romance | "funny", "humor", "lighthearted" |
| Drama | Psychological, Romance | "emotional", "tearjerker" |
| Ecchi | Comedy, Romance | — |
| Fantasy | Adventure, Action, Drama | "magic", "isekai" (also use as search term) |
| Horror | Psychological, Thriller, Supernatural | "scary", "creepy" |
| Mahou Shoujo | Fantasy, Comedy | "magical girl" |
| Mecha | Sci-Fi, Action, Drama | "robot", "mech" |
| Music | Slice of Life, Drama | "idol", "band" |
| Mystery | Psychological, Thriller, Drama | "detective", "whodunit" |
| Psychological | Mystery, Thriller, Drama, Horror | "mind game", "dark", "disturbing" |
| Romance | Comedy, Drama, Slice of Life | "love story", "romcom" |
| Sci-Fi | Action, Mecha, Drama | "space", "futuristic", "space opera" |
| Slice of Life | Comedy, Drama, Romance | "moe", "calm", "everyday" |
| Sports | Drama, Comedy | "competition" |
| Supernatural | Fantasy, Action, Horror | "ghost", "demon", "spirit" |
| Thriller | Psychological, Mystery, Horror | "suspense", "tension" |

## format reference

| Value | Meaning |
|---|---|
| `tv` | Seasonal television series |
| `movie` | Feature film |
| `ova` | Original Video Animation (direct-to-video) |
| `ona` | Original Net Animation (streaming-first) |
| `special` | Short or bonus episode |

Pass to `ani search --format <value>` always lowercase.

## score display guide

| Range | Emoji | Label |
|---|---|---|
| 90–100 | 🔥 | Exceptional |
| 80–89 | ⭐ | Great |
| 70–79 | 👍 | Good |
| 60–69 | 📊 | Average |
| 0–59 | *(number only)* | Below average |
| 0 (unscored) | `—` | No score yet |
