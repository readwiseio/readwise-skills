---
name: highlight-graph
description: Visualize your highlights and their connections in an interactive 2D graph
metadata:
  openclaw:
    requires:
      bins: ["readwise"]
---

You are building an interactive 2D force-graph visualization of the user's Readwise highlights, showing how ideas connect across books, articles, and other sources. Think Obsidian's graph view, but for highlights.

## Readwise Access

Check if Readwise MCP tools are available (e.g. `mcp__readwise__readwise_list_highlights`). If they are, use them throughout. If not, use the equivalent `readwise` CLI commands instead.

## Process

### Step 1: Fetch Highlights

Open with:

> **Highlight Graph** · Readwise
>
> I'll pull your recent highlights, find connections between them, and build a graph you can explore. Give me a moment.

Fetch the user's most recent highlights using `readwise_list_highlights` with `page_size=100`. Fetch 2 pages (200 highlights) for a good starting graph. Each page returns highlights from most recent to least recent — use `page=1`, then `page=2`.

### Step 2: Prepare Highlight Data

Parse the API responses and build a JSON array of highlights. Each highlight needs:

```json
{
  "id": "12345",
  "text": "The actual highlight text...",
  "note": "User's note if any",
  "book_id": 58741401,
  "source_title": "The Goal",
  "source_author": "Eliyahu Goldratt",
  "url": "https://..."
}
```

**Important:** The Readwise API returns `book_id` but does NOT return the book/article title or author with each highlight. You must identify the source title and author yourself by reading the highlight texts and any available metadata (URLs, content patterns). Group highlights by `book_id` and infer the source from context. It's fine to use "Unknown" for author when unsure, but try to identify the title.

Write this array to a temp file: `/tmp/highlights.json`

### Step 3: Initial Render (No Connections)

Write an empty connections file and run the build script to give the user something to look at immediately:

```bash
echo '[]' > /tmp/connections.json
python3 SKILL_DIR/build_graph.py --highlights /tmp/highlights.json --connections /tmp/connections.json --output highlight-graph.html
open highlight-graph.html
```

Tell the user:

> Graph is open with **{N} highlights** across **{N} sources**. Finding connections between ideas now...

### Step 4: Find Cross-Source Connections

Launch **parallel subagents** (3-5 agents) to find semantic connections between highlights from *different* sources. Each agent should analyze a batch of highlights and return connections.

**Batching strategy:**
1. Group highlights by source (book_id).
2. Split sources into batches of ~8 sources each.
3. For each batch pair (including within-batch), launch an agent with the highlight texts from those sources.
4. Each agent should find 5-10 genuine conceptual connections — shared themes, ideas, or language across different sources.

Each agent should return a JSON array of connections:

```json
[
  {
    "a_id": "12345",
    "b_id": "67890",
    "label": "Feedback loops",
    "why": "Both highlights discuss how tight feedback loops improve quality"
  }
]
```

**Quality over quantity.** Only create connections when the link is real and would be interesting. 15-30 total cross-source connections for 200 highlights is ideal.

### Step 5: Rebuild with Connections

Merge all agent results into a single connections JSON array, write to `/tmp/connections.json`, and re-run the build script:

```bash
python3 SKILL_DIR/build_graph.py --highlights /tmp/highlights.json --connections /tmp/connections.json --output highlight-graph.html
open highlight-graph.html
```

Present a summary:

> Built a graph of **{N} highlights** across **{N} sources**, with **{N} connections** between ideas.
>
> A few interesting connections I found:
> - "{highlight A snippet}" ↔ "{highlight B snippet}" — *{connection label}*
> - ...
>
> The graph is open in your browser. Want to add more highlights?

### Step 6: Iterate

- **More highlights:** Fetch additional pages (`page=3`, `page=4`, etc.), re-run source identification, find new connections, rebuild.
- **Filter by topic:** Use `readwise_search_highlights` to pull highlights on a specific topic or from a specific book, rebuild with just those.

## The Build Script

`build_graph.py` (in this skill's directory) handles all the visualization logic. It takes two JSON files and outputs a self-contained HTML file:

```
python3 build_graph.py --highlights highlights.json --connections connections.json --output output.html
```

**highlights.json:** Array of `{id, text, note, book_id, source_title, source_author, url}`
**connections.json:** Array of `{a_id, b_id, label, why}`

The script handles:
- Deduplication and filtering of highlights
- Color assignment per source (rose-pine palette)
- Same-source full pairwise connectivity (all highlights from same book linked)
- Cross-source semantic connections (dashed purple edges with particles)
- Interactive sidebar panel with connected ideas and same-book highlights
- Label overlap prevention
- Screen-relative text scaling (consistent size at any zoom level)
- Source legend with click-to-filter
- Hover isolation and selection persistence

The output is a single HTML file using [force-graph](https://github.com/vasturiano/force-graph) from CDN. No server needed — just open in a browser.

## SKILL_DIR

Replace `SKILL_DIR` in commands above with the actual path to this skill's directory (where `build_graph.py` lives).
