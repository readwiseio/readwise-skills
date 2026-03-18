---
name: readwise-cli
description: How to use the Readwise CLI — access highlights, documents, and your entire reading library from the command line
---

# Readwise CLI

Use the `readwise` command to access the user's Readwise highlights and Reader documents. Readwise has two products:

- **Readwise** — highlights from books, articles, podcasts, and more. Includes daily review and spaced repetition.
- **Reader** — a read-later app for saving and reading articles, PDFs, EPUBs, RSS feeds, emails, tweets, and videos.

## Setup

If `readwise` is not installed:
```bash
npm install -g @readwise/cli
```

If not authenticated, ask the user for their Readwise access token (they can get one at https://readwise.io/access_token), then run:
```bash
readwise login-with-token <token>
```

## Discovering Commands

Every command supports `--help` for full option details:
```bash
readwise --help
readwise reader-search-documents --help
readwise readwise-list-highlights --help
```

Add `--json` to any command for machine-readable output. Use `--refresh` to force-refresh cached data.

## Reader Commands

### Searching documents

```bash
# Semantic search across all saved documents
readwise reader-search-documents --query "spaced repetition"

# Search only articles saved for later
readwise reader-search-documents --query "machine learning" --category-in article --location-in later,shortlist

# Search by author within the inbox
readwise reader-search-documents --query "AI" --author-search "Simon Willison" --location-in new

# Search documents published after a date
readwise reader-search-documents --query "transformers" --published-date-gt 2024-01-01
```

### Browsing documents

```bash
# List 10 most recent inbox items (minimal fields to save tokens)
readwise reader-list-documents --location new --limit 10 --response-fields title,author,summary,word_count,category,saved_at

# List archived articles tagged "research"
readwise reader-list-documents --location archive --tag research --category article

# List unseen documents in the inbox
readwise reader-list-documents --location new --seen false

# List RSS feed items
readwise reader-list-documents --location feed --limit 20 --response-fields title,author,summary,site_name

# Get a specific document by ID
readwise reader-list-documents --id <document_id>
```

Locations: `new` (inbox), `later`, `shortlist`, `archive`, `feed`. When the user says "inbox", use `new`.

### Reading and highlighting

```bash
# Get full document content as Markdown
readwise reader-get-document-details --document-id <id>

# Get all highlights on a document
readwise reader-get-document-highlights --document-id <id>

# Highlight a passage (html-content must match the document's HTML exactly)
# Get the HTML first via reader-list-documents with --response-fields html_content
readwise reader-create-highlight --document-id <id> --html-content "<p>The exact passage to highlight</p>"

# Highlight with a note and tags
readwise reader-create-highlight --document-id <id> --html-content "<p>Key insight</p>" --note "Connects to spaced repetition research" --tags review,concept
```

### Saving documents

```bash
# Save a URL — Reader scrapes it automatically
readwise reader-create-document --url "https://example.com/article"

# Save with metadata
readwise reader-create-document --url "https://example.com" --title "Great Article" --tags research,ai --notes "Recommended by Alice"

# Save raw Markdown content (provide a unique URL as identifier)
readwise reader-create-document --title "Meeting Notes" --markdown "# Notes from today..." --url "https://me.com#notes-march-2025"
```

### Organizing

```bash
# Move documents between locations (max 50 per call)
readwise reader-move-documents --document-ids <id1>,<id2> --location archive
readwise reader-move-documents --document-ids <id> --location later

# Bulk mark documents as seen
readwise reader-bulk-edit-document-metadata --documents '[{"document_id": "<id>", "seen": true}]'

# Bulk update metadata (title, author, tags, summary, etc.)
readwise reader-bulk-edit-document-metadata --documents '[{"document_id": "<id>", "title": "Better Title", "tags": ["ai", "research"]}]'

# Tags
readwise reader-list-tags
readwise reader-add-tags-to-document --document-id <id> --tag-names important,research
readwise reader-remove-tags-from-document --document-id <id> --tag-names old-tag

# Highlight tags and notes
readwise reader-add-tags-to-highlight --document-id <id> --highlight-document-id <hid> --tag-names concept
readwise reader-remove-tags-from-highlight --document-id <id> --highlight-document-id <hid> --tag-names old-tag
readwise reader-set-highlight-notes --document-id <id> --highlight-document-id <hid> --notes "Updated note"
```

### Exporting

```bash
# Export all documents as a ZIP of Markdown files (async)
readwise reader-export-documents
readwise reader-get-export-documents-status --export-id <id>

# Delta export — only docs updated since last export
readwise reader-export-documents --since-updated "2024-01-01T00:00:00Z"
```

## Readwise Commands

### Searching highlights

```bash
# Semantic search across all highlights
readwise readwise-search-highlights --vector-search-term "learning techniques"

# Search with full-text filter on a specific field
readwise readwise-search-highlights --vector-search-term "memory" --full-text-queries '[{"field_name": "document_title", "search_term": "psychology"}]'
```

Full-text query fields: `document_author`, `document_title`, `highlight_note`, `highlight_plaintext`, `highlight_tags`.

### Browsing highlights

```bash
# List 20 most recent highlights
readwise readwise-list-highlights --page-size 20

# Highlights from a specific book
readwise readwise-list-highlights --book-id <id>

# Highlights from the last month
readwise readwise-list-highlights --highlighted-at-gt "2025-02-01T00:00:00Z"
```

### Creating and editing highlights

```bash
# Create a highlight (matched to a book by title/author, or goes into "Quotes")
readwise readwise-create-highlights --highlights '[{"text": "The key insight here", "title": "Book Title", "author": "Author Name"}]'

# Create multiple highlights at once
readwise readwise-create-highlights --highlights '[{"text": "First quote", "title": "Book A"}, {"text": "Second quote", "title": "Book B"}]'

# Update a highlight — text, note, color, tags
readwise readwise-update-highlight --highlight-id <id> --note "New note" --add-tags concept,review --color blue

# Delete a highlight
readwise readwise-delete-highlight --highlight-id <id>
```

Colors: `yellow`, `blue`, `pink`, `orange`, `green`, `purple`.

### Daily review

```bash
# Get today's spaced repetition review
readwise readwise-get-daily-review
```

Returns highlights selected by the spaced repetition algorithm plus a URL for interactive review.

## Example Workflows

**Triage the inbox:** List recent saves, read each one, decide what's worth the user's time.
```bash
readwise reader-list-documents --location new --limit 10 --response-fields title,author,summary,word_count,category,saved_at
readwise reader-get-document-details --document-id <id>
readwise reader-move-documents --document-ids <id> --location later    # worth reading
readwise reader-move-documents --document-ids <id> --location archive  # skip
```

**Search across everything:** Find all content on a topic across both highlights and documents.
```bash
readwise reader-search-documents --query "spaced repetition"
readwise readwise-search-highlights --vector-search-term "spaced repetition"
```

**Quiz on a recent read:** Find a finished document, get its content and highlights, quiz the user.
```bash
readwise reader-list-documents --location archive --limit 10 --response-fields title,author,summary,word_count
readwise reader-get-document-details --document-id <id>
readwise reader-get-document-highlights --document-id <id>
```

**Catch up on RSS:** Browse feed items, surface the best ones, mark the rest as seen.
```bash
readwise reader-list-documents --location feed --limit 20 --response-fields title,author,summary,word_count,site_name
readwise reader-bulk-edit-document-metadata --documents '[{"document_id": "<id>", "seen": true}]'
readwise reader-move-documents --document-ids <id> --location later
```

**Save and annotate:** Save a URL, highlight key passages, tag and organize.
```bash
readwise reader-create-document --url "https://example.com/article" --tags research
readwise reader-create-highlight --document-id <id> --html-content "<p>Key passage here</p>" --note "This connects to..."
readwise reader-add-tags-to-document --document-id <id> --tag-names important
```

**Build a reading recap:** See what you've finished and highlighted recently.
```bash
readwise reader-list-documents --location archive --updated-after "2025-03-10T00:00:00Z" --response-fields title,author,word_count,reading_progress
readwise reader-get-document-highlights --document-id <id>
readwise readwise-list-highlights --highlighted-at-gt "2025-03-10T00:00:00Z" --page-size 50
```
