Agent skills for [Readwise](https://readwise.io) and [Reader](https://readwise.io/read). Triage your inbox, catch up on feeds, quiz yourself on what you've read, and more.

These skills work with the Readwise Reader [MCP server](https://mcp2.readwise.io) or [CLI](https://github.com/readwiseio/readwise-cli). They follow the [Agent Skills specification](https://agentskills.io/specification) so they can be used by any skills-compatible agent, including Claude Code and Codex CLI.

## Prerequisites

You need one of:

- **Readwise Reader MCP server** — connected to your agent ([setup guide](https://mcp2.readwise.io))
- **Readwise CLI** — installed and authenticated ([install guide](https://github.com/readwiseio/readwise-cli))

Skills auto-detect which is available and use it.

## Installation

### Marketplace

```
/plugin marketplace add readwiseio/readwise-skills
/plugin install readwise@readwise-skills
```

### npx skills

```
npx skills add git@github.com:readwiseio/readwise-skills.git
```

### Readwise CLI

```
readwise skills install claude
readwise skills install --all
```

### Manually

#### Claude Code

Copy the `skills/` directory into `~/.claude/skills/` or into a `.claude/skills/` folder in your project. See the [official Claude Skills documentation](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).

#### Codex CLI

Copy the `skills/` directory into your Codex skills path (typically `~/.codex/skills`). See the [Agent Skills specification](https://agentskills.io/specification).

#### OpenCode

Clone the repo into the OpenCode skills directory:

```sh
git clone https://github.com/readwiseio/readwise-skills.git ~/.opencode/skills/readwise-skills
```

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
