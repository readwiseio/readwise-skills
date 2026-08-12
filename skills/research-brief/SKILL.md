---
name: research-brief
description: Build a research brief for something you're about to write — quotable passages from your own library, the sources that contradict you, and the gaps you still need to read
---

You are building a research brief for something the user is about to write: an essay, a talk, a
newsletter, a memo, a thread. The raw material is their own library. Every claim in the brief has
to trace back to a highlight or a document they actually saved.

This is the step before writing. It answers three questions:

1. What have I already read that supports this?
2. What have I already read that argues against it?
3. What am I missing?

Question 2 is the one that makes the piece good, and question 3 is the one that sends the user back
into Reader. Do not skip either to get to a tidy answer faster.

## Readwise Access

Check if Readwise MCP tools are available (e.g. `mcp__readwise__reader_list_documents`). If they
are, use them throughout. If not, use the equivalent `readwise` CLI commands instead (e.g.
`readwise reader-search-documents`, `readwise readwise-search-highlights`,
`readwise reader-get-document-highlights`). The instructions below reference MCP tool names,
translate to CLI equivalents as needed.

## Setup

1. **Check for persona file.** Read `reader_persona.md` in the current working directory if it
   exists. Use it to judge which sources the user already trusts, which arguments they've already
   made elsewhere, and what counts as new to them. Without it the brief still works, it just can't
   tell "the user has been chewing on this for two years" from "the user read this once."

2. **Parse the argument** as the thing being written. A thesis is better than a topic, because a
   thesis has an opposite and a topic doesn't.

```
/research-brief spaced repetition doesn't work for skills, only for facts
/research-brief why remote teams over-index on written culture
/research-brief a talk on why small teams ship faster
```

If the user gives a bare topic, ask for the claim they expect to make. If they don't have one yet,
say so and run anyway, but tell them the counter-evidence phase will be weak, since there's nothing
to be counter to.

---

## Phase 1: Expand the Angles

Do not search once with the user's phrasing. Semantic search returns one slice of the library per
phrasing, and the user's phrasing is the slice they already have in their head. The whole value of
this phase is retrieving material they forgot they had.

Write 5 to 8 queries before running any of them, covering:

- **The claim itself**, in the user's words
- **The claim inverted.** If the thesis is "spaced repetition doesn't work for skills," search
  "deliberate practice builds skill" and "spaced repetition works"
- **The mechanism**, one level down. Not "remote work" but "asynchronous decision making,"
  "written status updates," "meeting cost"
- **The adjacent field.** The user read about this in biology, sports, aviation safety, or military
  logistics without ever filing it under the topic name
- **The jargon term and the plain term.** People highlight both "interleaved practice" and "mixing
  up what you study"
- **Named authors and works** the user is likely to have saved on this topic
- **Every language the user reads in.** If their library is bilingual, an English query will not
  reliably reach the Spanish book and the reverse is just as true. One query per language, at
  minimum for the claim itself

Show the query list to the user before running it, in one compact block. They will spot the missing
angle instantly, and it costs one round trip.

---

## Phase 2: Pull the Evidence

Run every query against both products. Highlights and documents are different signals and you need
both.

```
mcp__readwise__readwise_search_highlights(vector_search_term="[query]")
mcp__readwise__reader_search_documents(query="[query]", limit=15)
```

For documents that look relevant and aren't already surfacing as highlights, pull what the user
marked up:

```
mcp__readwise__reader_get_document_highlights(document_id="[id]")
```

For anything you're going to quote, get the source metadata, because the citation needs the original
URL and not the Reader URL:

```
mcp__readwise__reader_get_document_details(document_id="[id]")
```

Deduplicate as you go. The same passage will come back from several angles. That's a ranking signal,
not noise: note how many distinct queries surfaced it, and keep one copy.

Two things about the results:

- `readwise_search_highlights` returns a flat list, and the text lives under `attributes`
  (`highlight_plaintext`, `highlight_note`, `document_title`, `document_author`) alongside a
  `score`. Scores are small in absolute terms and only meaningful relative to each other in the
  same result set, so rank by how many angles converged on a passage first and by score second
- Drop the product's own onboarding documents, such as "How to Use Readwise" and
  "Reader: Frequently Asked Questions". Every account has them, they can be a large share of the
  highlight count, and they will surface on any thesis about reading, notes, or knowledge work

---

## Phase 3: Sort by What It Actually Is

Every piece of evidence gets two labels. Do this before writing anything.

**Label one, where it came from:**

| Tier | What it is | Why it ranks here |
|------|-----------|-------------------|
| A | A note the user wrote on a highlight | Their own thinking, in their own words. This is the only material in the library that nobody else has |
| B | A highlighted passage, no note | They already judged this worth marking. Quotable as is |
| C | A document in the library with no highlights on it | Context and citation, but the user may not have read it closely. Never quote it as if they had |

Lead with tier A. A brief built only from tier B is a reading list with quotes; a brief that opens
with the user's own margin note from two years ago is the thing they can't get anywhere else.

**Label two, what it does to the claim:** supports, complicates, or contradicts.

Sort the contradictions to the top of their own section and do not soften them. If the user's
library disagrees with the user, that is the single most useful output of this skill. A piece that
survives its best counter-argument is better than a piece that never met one, and the user is going
to meet it in the replies either way.

---

## Phase 4: Find the Gaps

Now look at the shape of what came back and name what isn't there.

- Which obvious work on this topic is missing from the library entirely?
- Is every source from the same three years, or the same five authors?
- Is the supporting evidence tier A and B while the counter-evidence is all tier C? That means the
  user has been reading their own side closely and the other side loosely
- Is the whole brief one field? Most claims about people have a literature in psychology, one in
  economics, and one in whatever domain the user works in

Name specific works and authors, not categories. "You have nothing from the deliberate practice
literature" is weaker than "you have no Ericsson, and he's the person the counter-argument comes
from."

Then offer to file them. This is the one skill in this repo that puts things back into Reader:

```
mcp__readwise__reader_create_document(
  url="[url of the missing source]",
  tags=["research", "[topic-slug]"],
  notes="Gap found while building the brief on: [thesis]"
)
```

Ask before saving, save in one batch, and only save sources where you have a real URL. Never invent
one to fill the list out.

---

## Phase 5: Write the Brief

Markdown, paste ready. The user is going to move these blocks straight into a draft.

```
# Brief: [the thesis, as stated]

**Library coverage:** [N] highlights across [M] sources, [K] of them with your own notes.
[One sentence on whether that's thin or deep, honestly.]

## Where you already stand

[Tier A material. The notes the user wrote themselves, quoted and attributed to their past self
with the date. This section is what makes the piece theirs. If there is no tier A material, say
"no notes of your own on this yet" and move on. Do not pad it.]

## Supporting evidence

> "Exact highlight text, verbatim."
> [Author], *[Title]* ([link to the original source URL])

[One line on what this does for the argument, and where it belongs in the piece.]

[Repeat. Group by the sub-claim each passage supports, not by source, so the user can see which
parts of the argument are well armed and which are resting on a single quote.]

## What contradicts you

> "Exact highlight text, verbatim."
> [Author], *[Title]* ([link])

[What it costs the argument, stated plainly. Then either the strongest available response, or an
admission that the library doesn't contain one. "You have no answer to this in anything you've
read" is a legitimate and valuable finding.]

## Gaps

- **[Specific work or author]** — [what it would settle, and which section is weak without it]

## Thin ice

[Every place the brief is leaning on tier C, on a single source, or on a highlight whose
surrounding context you never read. Name them. The user needs to know which sentences to check
before publishing.]
```

### Citation rules

These are not stylistic. Getting one wrong turns a research tool into a liability.

- **Quotes are verbatim.** The text between quote marks is exactly the highlight text. If you
  shorten, use `[...]`. If you add a word for grammar, bracket it. Never smooth a quote out
- **Every quote traces to a highlight.** If you can't point at the highlight it came from, it
  doesn't go in the brief
- **Cite the original source URL**, from document details, not the Reader URL
- **Never fill a thin section from your own knowledge.** If the library has three highlights on
  this, the brief has three highlights on this and says so. General knowledge dressed up as the
  user's reading is the one failure mode that makes this skill worse than useless
- **Distinguish what they read from what they saved.** Tier C is cited as "in your library,"
  never as "you highlighted"
- **Flag translated editions.** A highlight from a Spanish edition of an English book is not what
  the author wrote. Quote it as it stands, mark it as a translation, and tell the user to pull the
  original wording before publishing the quote

---

## Phase 6: Save It Back

Offer to file the brief in Reader so it's searchable next time and shows up on mobile:

```
mcp__readwise__reader_create_document(
  url="https://research-brief.internal/[slug]-[ISO-timestamp]",
  title="Brief: [thesis]",
  author="Research Brief",
  category="article",
  summary="[N] sources, [K] contradictions, [M] gaps",
  markdown="[the full brief]",
  tags=["research", "[topic-slug]"]
)
```

Also offer to tag the sources the brief actually used, which turns the next brief on this topic
into a one command job:

```
mcp__readwise__reader_add_tags_to_document(document_id="[id]", tag_names=["[topic-slug]"])
```

---

## Error Handling

- **Nothing in the library on this topic**: Say so in one line, show the queries you tried, and go
  straight to Phase 4. A brief that is only a gap list is a real answer, and it's the honest one
- **One product empty**: Plenty of people have years of Kindle highlights in Readwise and an empty
  Reader, or the reverse. Note it once, keep searching the side that has data, and don't repeat the
  warning for every query
- **No notes anywhere in the library**: Tier A will be empty for most users, since highlighting is
  common and annotating is not. Say it once, in one line, and lean on tier B. Do not treat it as an
  error and do not nag
- **Only supporting evidence, no contradictions**: Do not manufacture a counter-argument from
  general knowledge. Report it as a finding: the library is one sided on this, here is who to read
- **Search returns hundreds of hits**: The thesis is too broad. Show the top themes and ask the user
  to narrow before pulling document details for all of them
- **Document details fail for a source you want to quote**: Quote it with the metadata you have and
  mark the citation `[link unavailable]` rather than dropping the passage or inventing a URL
- **Any unexpected failure**: Say what failed, what's already collected, and output the partial
  brief. Half a brief is still useful
