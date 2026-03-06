Agent skills for [Readwise](https://readwise.io) and [Reader](https://readwise.io/read). Triage your inbox, catch up on feeds, quiz yourself on what you've read, and more.

These skills work with the Readwise [MCP server](https://mcp2.readwise.io) or [CLI](https://github.com/readwiseio/readwise-cli). They follow the [Agent Skills specification](https://agentskills.io/specification) so they can be used by any skills-compatible agent, including Claude Code, Codex CLI, and OpenCode.

## Installation

### Claude Code

Installs the MCP server and all skills in one command:

```
/plugin install readwise-skills
```

You'll need to set your Readwise access token — get it at [readwise.io/access_token](https://readwise.io/access_token):

```
export READWISE_ACCESS_TOKEN=your_token_here
```

### npx skills

```
npx skills add readwiseio/readwise-skills
```

### Manually

Copy the `skills/` directory into your agent's skills path:

- **Claude Code:** `~/.claude/skills/` or `.claude/skills/` in your project
- **Codex CLI:** `~/.codex/skills/`
- **OpenCode:** `~/.opencode/skills/`

## Skills

| Skill | Description |
|-------|-------------|
| [triage](skills/triage) | AI walks you through your inbox one article at a time, telling you what's worth your time and why |
| [feed-catchup](skills/feed-catchup) | Skim your Reader feed in batches — RSS, newsletters, Twitter digests — pull out the gems, mark the rest as seen |
| [build-persona](skills/build-persona) | Build a reading profile from your highlights, tags, and history — powers personalization across all other skills |
| [quiz](skills/quiz) | Test yourself on something you just read — graded like a smart colleague who also read the piece |
| [now-reading-page](skills/now-reading-page) | Generate a "What I'm Reading" webpage from your library — host it on your personal site |

## Personalization

Run the `build-persona` skill first to generate a `reader_persona.md` file. The other skills (triage, feed-catchup, quiz) read this file to personalize their output to your interests, goals, and reading style.
