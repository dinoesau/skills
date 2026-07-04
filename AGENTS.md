# Agent Notes

This repo is a [skills.sh](https://skills.sh) skill pack. It contains no build system, tests, or dependencies — only Markdown skill definitions.

## Repo structure

- `skills.sh.json` — registry manifest. Lists skill groupings and references each skill by folder name. Schema: `https://skills.sh/schemas/skills.sh.schema.json`.
- `skills/<skill-name>/SKILL.md` — main skill definition. Must have YAML frontmatter with at least `name` and `description`.
- `skills/<skill-name>/REFERENCE.md`, `EXAMPLES.md` — optional supporting docs referenced by `SKILL.md`.

## Adding or updating a skill

1. Create/edit `skills/<skill-name>/SKILL.md`.
2. Include YAML frontmatter:
   ```yaml
   ---
   name: <skill-name>
   description: <one-line description>
   license: MIT
   allowed-tools: Bash
   ---
   ```
3. If the skill is new, add it to `skills.sh.json` under the correct `groupings` entry or `notGrouped`.
4. No tests or linting exist — verify by reading the skill file and the registry.

## Conventions

- Skill content is in Spanish; file names and frontmatter keys are in English.
- Keep `SKILL.md` focused. Move long catalogs or examples to `REFERENCE.md` / `EXAMPLES.md`.
- No `package.json`, CI, or generated artifacts to manage.

## Install / publish

Users install the pack with:

```bash
npx skills@latest add dinoesau/skills
```