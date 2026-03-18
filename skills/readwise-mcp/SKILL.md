---
name: readwise-mcp
description: How to use the Readwise MCP tools — access highlights, documents, and your entire reading library via MCP
---

# Readwise MCP

Use the Readwise MCP tools to access the user's Readwise highlights and Reader documents. Readwise has two products:

- **Readwise** — highlights from books, articles, podcasts, and more. Includes daily review and spaced repetition.
- **Reader** — a read-later app for saving and reading articles, PDFs, EPUBs, RSS feeds, emails, tweets, and videos.

## Setup

Add the Readwise MCP server to your client's configuration:

```json
{
  "readwise": {
    "type": "http",
    "url": "https://mcp2.readwise.io/mcp"
  }
}
```

The server handles authentication via OAuth — the user will be prompted to authorize on first use.

## Tool Reference

All tools are prefixed with `mcp__readwise__`. Each tool name maps directly to a Readwise or Reader API action.

## Reader Tools

### Searching documents

```
reader_search_documents(query="spaced repetition")
```

Hybrid search (semantic + keyword) across all saved documents. Combine with filters to narrow results:

```
# Search articles saved for later
reader_search_documents(query="machine learning", category_in=["article"], location_in=["later", "shortlist"])

# Search by author
reader_search_documents(query="AI agents", author_search="Simon Willison")

# Search within a date range
reader_search_documents(query="transformers", published_date_gt="2024-01-01")

# Search by tags
reader_search_documents(query="productivity", tags_in=["research"])
```

Other filters: `title_search`, `summary_search`, `note_search`, `url_search`, `source_search`, `document_id`, `limit` (default 20, max 100).

### Browsing documents

```
# List 10 most recent inbox items with minimal fields
reader_list_documents(location="new", limit=10, response_fields=["title", "author", "summary", "word_count", "category", "saved_at"])

# Archived articles with a specific tag
reader_list_documents(location="archive", tag=["research"], category="article")

# Unseen inbox items
reader_list_documents(location="new", seen=false)

# RSS feed items
reader_list_documents(location="feed", limit=20, response_fields=["title", "author", "summary", "site_name"])

# Get a specific document by ID
reader_list_documents(id="<document_id>")

# Paginate through results
reader_list_documents(location="later", limit=10, page_cursor="<cursor_from_previous_response>")
```

Locations: `new` (inbox), `later`, `shortlist`, `archive`, `feed`. When the user says "inbox", use `new`. Only use `feed` when the user explicitly asks about RSS/feeds.

Use `response_fields` to limit returned data and save tokens. The `id` field is always included. Available fields: `url`, `title`, `author`, `source`, `category`, `location`, `tags`, `site_name`, `word_count`, `reading_time`, `created_at`, `updated_at`, `published_date`, `summary`, `image_url`, `content`, `source_url`, `notes`, `parent_id`, `reading_progress`, `first_opened_at`, `last_opened_at`, `saved_at`, `last_moved_at`, `html_content`, `is_deleted`.

Tip: unseen documents have `first_opened_at=null`. Mark as seen via `reader_bulk_edit_document_metadata`.

### Reading and highlighting

```
# Get full document details including Markdown content
reader_get_document_details(document_id="<id>")

# Get all highlights on a document
reader_get_document_highlights(document_id="<id>")

# Create a highlight — html_content must match the document's HTML exactly
# Get the HTML via reader_list_documents with response_fields=["html_content"]
reader_create_highlight(document_id="<id>", html_content="<p>The exact passage to highlight</p>")

# Highlight with a note and tags
reader_create_highlight(document_id="<id>", html_content="<p>Key insight</p>", note="Connects to spaced repetition", tags=["review", "concept"])
```

### Saving documents

```
# Save a URL — Reader scrapes it automatically
reader_create_document(url="https://example.com/article")

# Save with metadata and tags
reader_create_document(url="https://example.com", title="Great Article", tags=["research", "ai"], notes="Recommended by Alice")

# Save raw Markdown content (provide a unique URL as identifier)
reader_create_document(title="Meeting Notes", markdown="# Notes from today...", url="https://me.com#notes-march-2025")
```

### Organizing

```
# Move documents between locations (max 50 per call)
reader_move_documents(document_ids=["<id1>", "<id2>"], location="archive")

# Bulk mark documents as seen
reader_bulk_edit_document_metadata(documents=[{"document_id": "<id>", "seen": true}])

# Bulk update metadata
reader_bulk_edit_document_metadata(documents=[{"document_id": "<id>", "title": "Better Title", "tags": ["ai", "research"]}])
```

Note: `reader_move_documents` and `reader_bulk_edit_document_metadata` share a rate limit of 20 calls/minute. Batch document IDs into fewer calls.

### Tags

```
# List all tags
reader_list_tags()

# Add tags to a document
reader_add_tags_to_document(document_id="<id>", tag_names=["important", "research"])

# Remove tags from a document
reader_remove_tags_from_document(document_id="<id>", tag_names=["old-tag"])

# Add tags to a highlight
reader_add_tags_to_highlight(document_id="<id>", highlight_document_id="<hid>", tag_names=["concept"])

# Remove tags from a highlight
reader_remove_tags_from_highlight(document_id="<id>", highlight_document_id="<hid>", tag_names=["old-tag"])

# Set notes on a highlight (pass null to clear)
reader_set_highlight_notes(document_id="<id>", highlight_document_id="<hid>", notes="My updated note")
```

### Exporting

```
# Export all documents as a ZIP of Markdown files (async — returns export_id)
reader_export_documents()

# Delta export — only docs updated since last export
reader_export_documents(since_updated="2024-01-01T00:00:00Z")

# Poll for completion
reader_get_export_documents_status(export_id="<id>")
```

Poll `reader_get_export_documents_status` until `status` is `"completed"`, then use the `download_url`.

## Readwise Tools

### Searching highlights

```
# Semantic search across all highlights
readwise_search_highlights(vector_search_term="learning techniques")

# Search with full-text filter on a specific field
readwise_search_highlights(vector_search_term="memory", full_text_queries=[{"field_name": "document_title", "search_term": "psychology"}])
```

Full-text query fields: `document_author`, `document_title`, `highlight_note`, `highlight_plaintext`, `highlight_tags`.

### Browsing highlights

```
# List 20 most recent highlights
readwise_list_highlights(page_size=20)

# Highlights from a specific book
readwise_list_highlights(book_id=12345)

# Highlights from the last month
readwise_list_highlights(highlighted_at_gt="2025-02-01T00:00:00Z")

# Paginate
readwise_list_highlights(page_size=100, page=2)
```

### Creating and editing highlights

```
# Create a highlight (matched to a book by title/author, or goes into "Quotes")
readwise_create_highlights(highlights=[{"text": "The key insight here", "title": "Book Title", "author": "Author Name"}])

# Create multiple highlights at once
readwise_create_highlights(highlights=[{"text": "First quote", "title": "Book A"}, {"text": "Second quote", "title": "Book B", "note": "Great point"}])

# Update a highlight
readwise_update_highlight(highlight_id=12345, note="New note", add_tags=["concept", "review"], color="blue")

# Delete a highlight
readwise_delete_highlight(highlight_id=12345)
```

Colors: `yellow`, `blue`, `pink`, `orange`, `green`, `purple`.

### Daily review

```
# Get today's spaced repetition review
readwise_get_daily_review()
```

Returns highlights selected by the algorithm plus a URL for interactive review.

## Example Workflows

**Triage the inbox:** List recent saves, read each one, decide what's worth the user's time.
```
reader_list_documents(location="new", limit=10, response_fields=["title", "author", "summary", "word_count", "category", "saved_at"])
reader_get_document_details(document_id="<id>")
reader_move_documents(document_ids=["<id>"], location="later")     # worth reading
reader_move_documents(document_ids=["<id>"], location="archive")   # skip
```

**Search across everything:** Find all content on a topic — run both in parallel.
```
reader_search_documents(query="spaced repetition")
readwise_search_highlights(vector_search_term="spaced repetition")
```

**Quiz on a recent read:** Find a finished document, get its content and highlights.
```
reader_list_documents(location="archive", limit=10, response_fields=["title", "author", "summary", "word_count"])
reader_get_document_details(document_id="<id>")
reader_get_document_highlights(document_id="<id>")
```

**Catch up on RSS:** Browse feed items, surface the best ones, mark the rest as seen.
```
reader_list_documents(location="feed", limit=20, response_fields=["title", "author", "summary", "word_count", "site_name"])
reader_bulk_edit_document_metadata(documents=[{"document_id": "<id1>", "seen": true}, {"document_id": "<id2>", "seen": true}])
reader_move_documents(document_ids=["<id>"], location="later")
```

**Save and annotate:** Save a URL, highlight key passages, tag and organize.
```
reader_create_document(url="https://example.com/article", tags=["research"])
reader_create_highlight(document_id="<id>", html_content="<p>Key passage here</p>", note="This connects to...")
reader_add_tags_to_document(document_id="<id>", tag_names=["important"])
```

**Build a reading recap:** See what you've finished and highlighted recently.
```
reader_list_documents(location="archive", updated_after="2025-03-10T00:00:00Z", response_fields=["title", "author", "word_count", "reading_progress"])
reader_get_document_highlights(document_id="<id>")
readwise_list_highlights(highlighted_at_gt="2025-03-10T00:00:00Z", page_size=50)
```
