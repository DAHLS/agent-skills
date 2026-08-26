# agent-skills

Home-made skills for [OpenCode](https://opencode.ai/).

| Skill                                               | What it does                                                                                                                                                                                                                                      |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`skills/nim-development`](skills/nim-development/) | Writing, building, and debugging **Nim** — decision guidance plus a research discipline for finding current, version-correct Nim info (thin training-data footprint). Carries a persistent trap log (`nim4friends`) with an owner-verified inbox. |
| [`skills/doc-sync`](skills/doc-sync/)               | Sync project docs to reality for a cold handoff. Inventory-first, six mandatory steps.                                                                                                                                                            |
| [`skills/nftt`](skills/nftt/)                       | Drive a consuming project through the [nFTT](https://github.com/DAHLS/nFTT) fine-tuning loop — init, config/data declaration, pipeline, train, export, eval, reading results. Topic-split reference under `references/topics/`.                   |

## Install

Clone once, symlink the skills your loader discovers:

```bash
git clone git@github.com:DAHLS/agent-skills.git ~/.agents/agent-skills
ln -s ../agent-skills/skills/nim-development ~/.agents/skills/nim-development
ln -s ../agent-skills/skills/doc-sync       ~/.agents/skills/doc-sync
ln -s ../agent-skills/skills/nftt           ~/.agents/skills/nftt
```

(Adjust paths to wherever your tool expects skills; this is the layout used
on the author's machines.) Pulling inside `~/.agents/agent-skills` updates
every installed skill at once.

## Skill metadata conventions

Every `SKILL.md` may carry a local house field `risk:`; spec-compliant
loaders ignore unknown fields. Each skill may ship an `agents/` folder with
tool-specific interface hints (`openai.yaml`).

## Validating after edits

Run the official checker from `agentskills/agentskills`:

```bash
skills-ref validate ~/.agents/agent-skills/skills/<name>
```

## License

GPL-3.0. See `LICENSE`.
