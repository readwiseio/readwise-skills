Agent skills for your [Readwise](https://readwise.io) and [Reader](https://readwise.io/read) data, powered by the Readwise [MCP server](https://mcp2.readwise.io)/[CLI](https://github.com/readwiseio/readwise-cli).


Triage your inbox, quiz yourself on what you've read, build a personalized now-reading page, and more.


## Installation

Prerequisite: make sure you have the Readwise [MCP server](https://mcp2.readwise.io) or [CLI](https://github.com/readwiseio/readwise-cli) installed (whichever you prefer!)

### npx skills (recommended)

Probably the easiest way to try out these skills is to install them via `npx skills`, which will allow them to work with any LLM app you might use (Claude, Codex, Opencode, Cursor, etc).

```
npx skills add readwiseio/readwise-skills
```

### Manual Installation

Copy the `skills/` directory into your agent's skills path:

- **Claude Code:** `~/.claude/skills/` or `.claude/skills/` in your project
- **Codex CLI:** `~/.codex/skills/`
- **OpenCode:** `~/.opencode/skills/`

### Claude Code Installation (with MCP Included)

Installs the MCP server and all skills in one command:

```
/plugin marketplace add readwiseio/readwise-skills
/plugin install readwise@readwise-skills
```

### Claude Cowork Installation (with MCP Included)

In the Cowork sidebar: **Customize → "+" → Add marketplace from GitHub** → enter `readwiseio/readwise-skills`. Then browse and install the Readwise plugin.


## The Skills

We've started this repo off with a few skills we've really enjoyed using, but so much more is possible!

| Skill | Description |
|-------|-------------|
| [triage](skills/triage) | AI walks you through your inbox one article at a time, telling you what's worth your time and why |
| [feed-catchup](skills/feed-catchup) | Skim your Reader feed in batches — RSS, newsletters, Twitter digests — pull out the gems, mark the rest as seen |
| [build-persona](skills/build-persona) | Build a reading profile from your highlights, tags, and history — powers personalization across all other skills |
| [quiz](skills/quiz) | Test yourself on something you just read — graded like a smart colleague who also read the piece |
| [now-reading-page](skills/now-reading-page) | Generate a "What I'm Reading" webpage from your library — host it on your personal site |
| [self-surprise](skills/self-surprise) | Dig through your reading history and tell you something surprising about yourself you didn't know |


### Personalization

Run the `build-persona` skill first to generate a `reader_persona.md` file. The other skills (triage, feed-catchup, quiz) read this file to personalize their output to your interests, goals, and reading style.

## Please share yours!

These skills are just a starting point, quickly hacked up by the Readwise team based on what's been for us to try. There is still so so much more that can be done. Please make a pull request adding your own favorite skills, or [shoot us an email](mailto:hello@readwise.io) at any time :)

