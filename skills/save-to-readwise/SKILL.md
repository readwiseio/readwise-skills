---
name: save-to-readwise
description: Save the best insights from the current AI conversation to Readwise as tagged highlights, so they resurface in Daily Review. Use when the user types /save-to-readwise or asks to save, keep, or remember parts of the conversation in Readwise.
---

You are helping the user save key takeaways from the current conversation into Readwise as highlights. Conversations with an AI are full of useful explanations, decisions, and insights that are lost once the chat ends — this skill turns the best of them into self-contained, tagged highlights that resurface in Readwise's Daily Review.

## Readwise Access

Check if Readwise MCP tools are available (e.g. `mcp__readwise__readwise_create_highlights`). If they are, use them throughout. If not, use the equivalent `readwise` CLI commands instead (e.g. `readwise readwise-create-highlights`, `readwise readwise-update-highlight`). The instructions below reference MCP tool names — translate to CLI equivalents as needed.

## When to run

When the user types `/save-to-readwise` or asks to save parts of the conversation to Readwise.

## Process

### 1. Load the user's tags

Call `mcp__readwise__reader_list_tags` to see which tags the user already uses. Prefer reusing these over inventing new ones, so new highlights fit into the user's existing system. If the user has no tags, suggest short, lowercase, general-purpose tags based on the content (e.g. `science`, `history`, `how-to`).

### 2. Propose highlight candidates

Scan the conversation and propose **3–6** highlights. Each should be a discrete, self-contained insight — not a summary of the whole conversation. Good candidates:

- Key decisions or conclusions reached
- Surprising facts, or common misconceptions that got corrected
- Rules of thumb, frameworks, or mental models worth reusing
- Anything the user explicitly called important
- Topics the user asked to have explained more simply or in more depth, or asked follow-up questions about — these signal genuine curiosity

Write each one so that someone reading it months later, with no access to the conversation, understands it. Include the "why" when it matters. Keep each to 1–3 sentences.

For each candidate, suggest **0–2 tags**. Leave it untagged if nothing fits — don't force a tag.

Present them as a numbered list with tags inline, and propose a title:

> Here are some highlights from our conversation, to be saved under **"How Memory Works"**. Pick the ones to keep (e.g. "1, 3" or "all"), and feel free to change any tag or the title:
>
> 1. `[learning]` **Testing beats rereading** — Pulling information out of memory (quizzing yourself) strengthens it far more than rereading notes, even though rereading feels more productive.
> 2. `[learning]` `[science]` **Spacing works with forgetting** — Reviewing just as you're starting to forget, at gradually longer intervals, makes memories last longer than cramming the same amount of study.
> 3. **Explain it back** — Explaining the idea in your own words and asking for corrections showed gaps that just reading the explanation hid.

**Title:** a short descriptive name for the conversation's topic (e.g. "How Memory Works"). All highlights saved in one run share the same title so they group together as one source in Readwise.

### 3. Wait for the user to choose

Wait for the user to pick by number, "all", or "none". They may also edit text, change tags (e.g. "1 and 2, but tag 2 as science"), or rename the title. If they say "none" or "cancel", do nothing and move on.

### 4. Save the selected highlights

`readwise_create_highlights` does not accept tags, so saving is two steps. Do both before confirming.

**Step A — create.** Call `mcp__readwise__readwise_create_highlights` once with all selected highlights:

- `text`: the highlight text (the bolded label is not part of the text unless it adds meaning)
- `title`: the shared title from step 2
- `author`: the AI assistant you are (e.g. "Claude"), so the user can tell where these came from
- `category`: `"articles"`
- `note`: optional — extra context such as the question that prompted the insight

**Step B — tag.** For each created highlight that has tags, call `mcp__readwise__readwise_update_highlight` with:

- `highlight_id`: the ID returned in Step A
- `add_tags`: the tag list, e.g. `["learning", "science"]`

If Step A fails, tell the user and stop — don't retry silently in a loop. If tagging fails for some highlights, still report what was saved and list which tags could not be applied.

### 5. Confirm

Tell the user how many highlights were saved, under which title, and which tags went on which highlight:

> Saved 2 highlights under "How Memory Works":
> - `[learning]` Testing beats rereading
> - `[learning]` `[science]` Spacing works with forgetting

## Rules

- **Never save without the user explicitly choosing highlights.**
- Every highlight must stand on its own without the conversation.
- Don't save secrets, credentials, API keys, or personal data that appeared in the conversation, even if selected — point it out and ask the user to confirm a redacted version instead.
- Prefer the user's existing tags over new ones; keep tags lowercase.
- Always run Step B for any highlight that has a confirmed tag.
