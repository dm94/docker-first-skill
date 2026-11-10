# docker-first

[![skills.sh](https://skills.sh/b/dm94/docker-first-skill)](https://skills.sh/dm94/docker-first-skill)

An agent skill that makes AI coding agents **run code and tools inside
ephemeral Docker containers instead of installing things on your machine**.

## Why

Agents routinely run `pip install`, `npm install -g`, `apt install` or
`curl | sh` on the host. That leads to:

- Polluted global environments and version conflicts
- Unaudited install scripts (supply-chain risk)
- Non-reproducible results
- Leftover tools nobody remembers installing

`docker-first` tells the agent to use throwaway, version-pinned containers
(official task-specific images when available), mounting only the project
directory, running as your user, and with the network off by default.

## Install

### Skills CLI (recommended)

One command installs the skill into every detected agent — Claude Code, Codex,
Cursor, OpenCode, GitHub Copilot, Windsurf, Gemini and
[75+ more](https://github.com/vercel-labs/skills/blob/main/docs/agents.md).

```bash
# Install into the current project
npx skills add dm94/docker-first-skill

# Install globally, available in every project
npx skills add dm94/docker-first-skill -g

# Non-interactive: only this skill, only Claude Code
npx skills add dm94/docker-first-skill --skill docker-first -g -a claude-code -y
```

The published page is <https://skills.sh/dm94/docker-first-skill>.

### Manual install

The skill is the `skills/docker-first/` folder (a `SKILL.md` plus a reference
file). Copy it to where your agent looks for skills:

| Agent | Location |
|---|---|
| Claude Code (personal) | `~/.claude/skills/docker-first/` |
| Claude Code (project) | `.claude/skills/docker-first/` |
| OpenCode (project) | `.opencode/skills/docker-first/` |
| Codex / Cursor / other agents | `.agents/skills/docker-first/` |
| Claude.ai / Claude apps | Zip the `skills/docker-first/` folder and upload it in Settings → Capabilities → Skills |
| Other agents | Copy the contents of `SKILL.md` into the agent's rules/instructions file (`.cursor/rules`, `AGENTS.md`, ...) |

```bash
git clone https://github.com/dm94/docker-first-skill
cp -r docker-first-skill/skills/docker-first ~/.claude/skills/
```

## Important limitations

- **A skill is guidance, not enforcement.** An agent can ignore it. For real
  guarantees, also use the agent's permission system (deny `pip install`,
  `npm install -g`, `apt`, `brew`, `curl | sh`) or a devcontainer/VM sandbox.
- **Docker is not a perfect sandbox.** Mounting the Docker socket, your home
  directory, or using `--privileged` defeats the isolation. The skill forbids
  these, but review what the agent runs.
- **Images are supply chain too.** Prefer official images, pin tags (or
  digests), and review unfamiliar ones.
- Requires Docker (or rootless Podman) on the host.

## Contents

```
docker-first-skill/
├── skills/
│   └── docker-first/
│       ├── SKILL.md              # Rules and workflow
│       └── references/
│           └── images.md         # Task → recommended image table and tips
├── skills.sh.json                # skills.sh repository page layout
├── README.md
└── LICENSE
```

## Development

The skill follows the [Agent Skills specification](https://agentskills.io/specification)
and the repository uses the standard `skills/<name>/` layout discovered by the
Skills CLI. To verify that the skill is discoverable before publishing:

```bash
npx skills add ./ --list
```

## Contributing

PRs welcome, especially for the image table and agent-specific install notes.

## License

MIT
