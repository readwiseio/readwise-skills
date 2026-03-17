---
name: readwise-official
description: Access your Readwise highlights and Reader documents from the command line. Search, read, organize, and manage your entire reading library.
metadata:
  openclaw:
    category: "productivity"
    requires:
      bins: ["readwise"]
    cliHelp: "readwise --help"
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

### Searching and browsing documents

```bash
# Semantic search across all saved documents
readwise reader-search-documents --query "spaced repetition"

# Search with filters — by category, location, author, date range, tags
readwise reader-search-documents --query "machine learning" --category-in article --location-in later,shortlist

# List recent inbox items with minimal fields (saves tokens)
readwise reader-list-documents --location new --limit 10 --response-fields title,author,summary,word_count,category

# List archived articles by a specific tag
readwise reader-list-documents --location archive --tag "research" --category article

# List unseen documents
readwise reader-list-documents --location new --seen false
```

Reader locations: `new` (inbox), `later`, `shortlist`, `archive`, `feed`. When the user says "inbox", use `new`.

### Reading document content

```bash
# Get full document details including Markdown content
readwise reader-get-document-details --document-id <id>

# Get all highlights on a document
readwise reader-get-document-highlights --document-id <id>
```

### Saving and creating

```bash
# Save a URL — Reader scrapes it automatically
readwise reader-create-document --url "https://example.com/article"

# Save with metadata and tags
readwise reader-create-document --url "https://example.com" --title "My Title" --tags research,important

# Save raw content (provide a unique URL as identifier)
readwise reader-create-document --title "Meeting Notes" --markdown "# Notes from today..." --url "https://me.com#notes-2024"

# Highlight a passage (html-content must match the document's HTML exactly)
readwise reader-create-highlight --document-id <id> --html-content "<p>The key passage</p>" --note "Connects to..."
```

### Organizing

```bash
# Move documents between locations (max 50 per call)
readwise reader-move-documents --document-ids <id1>,<id2> --location archive

# Bulk mark as seen
readwise reader-bulk-edit-document-metadata --documents '[{"document_id": "<id>", "seen": true}]'

# Tag management
readwise reader-list-tags
readwise reader-add-tags-to-document --document-id <id> --tag-names important,research
readwise reader-remove-tags-from-document --document-id <id> --tag-names old-tag

# Highlight tags and notes
readwise reader-add-tags-to-highlight --document-id <id> --highlight-document-id <hid> --tag-names concept
readwise reader-set-highlight-notes --document-id <id> --highlight-document-id <hid> --notes "My note"
```

### Exporting

```bash
# Export all documents as Markdown ZIP (async — poll for completion)
readwise reader-export-documents
readwise reader-get-export-documents-status --export-id <id>

# Delta export — only docs updated since last export
readwise reader-export-documents --since-updated "2024-01-01T00:00:00Z"
```

## Readwise Commands

### Searching and browsing highlights

```bash
# Semantic search across all highlights (books, articles, podcasts, etc.)
readwise readwise-search-highlights --vector-search-term "learning techniques"

# List recent highlights
readwise readwise-list-highlights --page-size 20

# Highlights from a specific book
readwise readwise-list-highlights --book-id <id>

# Highlights from the last month
readwise readwise-list-highlights --highlighted-at-gt "2024-06-01T00:00:00Z"
```

### Creating and editing highlights

```bash
# Create a highlight (goes into a book matched by title/author, or "Quotes" if no title)
readwise readwise-create-highlights --highlights '[{"text": "The key insight", "title": "Book Title", "author": "Author Name"}]'

# Update a highlight — change text, add note, manage tags, set color
readwise readwise-update-highlight --highlight-id <id> --note "Updated note" --add-tags concept,review --color blue

# Delete a highlight
readwise readwise-delete-highlight --highlight-id <id>
```

### Daily review

```bash
# Get today's spaced repetition review with highlights and interactive review URL
readwise readwise-get-daily-review
```

## Example Workflows

**Triage the inbox:** List recent saves, read each one, decide what's worth the user's time, and move to the right place.
```bash
readwise reader-list-documents --location new --limit 10 --response-fields title,author,summary,word_count,category,saved_at
readwise reader-get-document-details --document-id <id>
readwise reader-move-documents --document-ids <id> --location later
```

**Search across everything:** Search both Reader documents and Readwise highlights in parallel to find everything on a topic.
```bash
readwise reader-search-documents --query "spaced repetition"
readwise readwise-search-highlights --vector-search-term "spaced repetition"
```

**Quiz on a recent read:** Find recently finished documents, pick one, fetch its content and highlights, then quiz the user on what they read.
```bash
readwise reader-list-documents --location archive --limit 10 --response-fields title,author,summary,word_count
readwise reader-get-document-details --document-id <id>
readwise reader-get-document-highlights --document-id <id>
```

**Catch up on RSS:** Browse recent feed items, surface the best ones, mark the rest as seen.
```bash
readwise reader-list-documents --location feed --limit 20 --response-fields title,author,summary,word_count,site_name
readwise reader-bulk-edit-document-metadata --documents '[{"document_id": "<id>", "seen": true}]'
readwise reader-move-documents --document-ids <id> --location later
```

**Save and annotate:** Save a URL, highlight key passages, and organize with tags.
```bash
readwise reader-create-document --url "https://example.com/article" --tags research
readwise reader-create-highlight --document-id <id> --html-content "<p>Key passage</p>" --note "This connects to..."
readwise reader-add-tags-to-document --document-id <id> --tag-names important,research
```
