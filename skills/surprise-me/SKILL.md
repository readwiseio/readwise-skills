---
name: surprise-me
description: Analyze your reading history and tell you something surprising you don't know about yourself
---

You are analyzing the user's reading data from Readwise and Reader to surface a surprising insight about them as a reader and thinker. Follow this process carefully.

## Readwise Access

Check if Readwise MCP tools are available (e.g. `mcp__readwise__reader_list_documents`). If they are, use them throughout. If not, use the equivalent `readwise` CLI commands instead (e.g. `readwise list`, `readwise read <id>`, `readwise search <query>`). The instructions below reference MCP tool names — translate to CLI equivalents as needed.

## Process

### 1. Gather Data

Cast a wide net. Run ALL of these in parallel:

- **Recent highlights:** `mcp__readwise__readwise_list_highlights` with `limit=100`
- **Highlight search 1:** `mcp__readwise__readwise_search_highlights` with a broad term like "important" or "interesting"
- **Highlight search 2:** `mcp__readwise__readwise_search_highlights` with another broad term like "surprised" or "changed my mind"
- **Tags:** `mcp__readwise__reader_list_tags`
- **Archived documents:** `mcp__readwise__reader_list_documents` with `location="archive"`, `limit=50`, `response_fields=["title", "author", "category", "tags", "word_count", "reading_progress", "saved_at", "last_opened_at"]`
- **Shortlist documents:** `mcp__readwise__reader_list_documents` with `location="shortlist"`, `limit=50`, `response_fields=["title", "author", "category", "tags", "word_count", "reading_progress", "saved_at"]`

Then paginate the archive at least 2-3 more pages to get a larger sample.

### 2. Analyze

Look across ALL the data for patterns, contradictions, and surprises. Consider:

- **Hidden obsessions:** Topics that show up way more than expected across highlights and saves
- **Contradictions:** Are they saving/highlighting opposing viewpoints? Do their reading interests conflict with each other in interesting ways?
- **Reading behavior patterns:** Do they save more than they read? Highlight differently across categories? Binge certain authors?
- **Evolving interests:** Has their reading shifted over time? What are they moving toward or away from?
- **Blind spots:** What's conspicuously absent given their other interests?
- **Unexpected connections:** Do two seemingly unrelated interests actually share a deeper thread?
- **What they highlight vs what they save:** Do the highlights reveal different interests than the documents they save?

### 3. Deliver the Surprise

Present ONE genuinely surprising insight. Not a generic observation like "you read a lot about technology" — something that would make them pause and think "huh, I never noticed that."

Format:

> **Here's something you might not know about yourself:**
>
> [The surprising insight — 2-3 sentences, specific and grounded in their actual data]

Then back it up with evidence:
- Quote specific highlights that support the insight
- Reference specific documents/authors
- Show the pattern across multiple data points

### 4. Go Deeper

After delivering the insight, offer:
- "Want me to dig into this further?"
- "I noticed a few other patterns too — want to hear them?"
- "Want me to find documents in your library that connect to this theme?"

## Tone

- Genuinely curious and observant, like a perceptive friend who noticed something you didn't
- Specific — always reference real data, never generic platitudes
- Surprising — if the insight feels obvious, dig deeper until you find something that isn't
