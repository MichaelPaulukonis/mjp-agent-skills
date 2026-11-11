# mjp-agent-skills

Skills I've written for Claude Code and other [Agent Skills](https://agentskills.io)-compatible agents.

![Students at lathes in the Manual Training Shop, Illinois State Normal University, 1917](docs/img/manual-training-shop-1917.jpg)

*Manual Training Shop, Illinois State Normal University, Normal, Ill. Stereograph, Keystone View Company, 1917. [Library of Congress](https://www.loc.gov/item/2018647661/), Stereograph Cards collection. No known restrictions on publication.*

## Two tiers

- **`skills/`** - installable. No personal paths, no dependencies on my private setup.
- **`examples/`** - skills that work for me but are wired to my environment (my repos, my voice). Read them, copy what's useful, adapt. When one gets its personal bits moved into config, it graduates to `skills/`.

## Install

    npx skills add MichaelPaulukonis/mjp-agent-skills --skill <name>

`npx skills` only discovers `skills/`; examples are meant to be read, not installed.

| Skill | What it does |
| --- | --- |
| `elevator-pitch` | Drafts or sharpens a short pitch, with a pass/fail eval loop |
| `image-frames-to-video` | Turns numbered image frames into video with ffmpeg or [image-processing-shell](https://github.com/MichaelPaulukonis/image-processing-shell)'s `tool.sh` |
| `verify-before-trusting` | Makes the agent show the mechanism behind "done", numbers, and causal claims before you have to ask |

| Example | What it shows |
| --- | --- |
| `claude-sync` | Backing up Claude Code config into version control (symlink-adopt vs copy-sync) |
| `writing-style` | A personal voice/style skill other skills defer to |

## Prompts

Not skills: one-shot instructions you point an agent at by path ("Read and execute `prompts/<name>/prompt.md`"). Each folder has a README with the why.

| Prompt | What it does |
| --- | --- |
| `multi-agent-repo-setup` | Makes Claude Code and Antigravity share one `AGENTS.md` rulebook and hand off work through [Beads](https://github.com/gastownhall/beads) |

## Related

- [claude-wikijs-skill](https://github.com/MichaelPaulukonis/claude-wikijs-skill) - Wiki.js from Claude Code via a CLI script, no MCP server.

## Credits

- The `eval.md` / `memory.md` scaffolding (`_shared/`) follows Peter Yang's pattern: [Full Tutorial: Build Self-Improving Claude Skills in 20 Min](https://creatoreconomy.so/p/full-tutorial-build-self-improving-claude-skills-in-20-min) ([transcript source repo](https://github.com/petergyang/behind-the-craft-transcripts)).
- I also use a lightly modified [named-persona-adversarial-review](https://github.com/alirezarezvani/claude-skills) - get it from upstream.

## License

MIT
