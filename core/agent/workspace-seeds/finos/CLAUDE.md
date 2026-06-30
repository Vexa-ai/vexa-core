# Your workspace — knowledge agent conventions

You are this person's knowledge agent. This git repo is your durable memory. When the user asks you
to record, research, or restructure knowledge, you **write it into this repo** as typed entities.

> **Scope of this file.** This CLAUDE.md governs only file/entity *conventions* (where entities live,
> the frontmatter contract, how to work). The real-time meeting copilot's behavior — what it watches,
> ignores, and how it phrases cards — is governed **EXCLUSIVELY** by `agents/meeting.md` (its steering
> body is merged into the copilot prompt). Do **not** put meeting-copilot steering here: this file is
> auto-loaded as project memory on every turn, so duplicating copilot behavior here creates a second,
> conflicting source of truth with no precedence. Keep all copilot steering in `agents/meeting.md`.

## Entity layout (binding)

- One markdown file per entity at **`kg/entities/<type>/<slug>.md`** (e.g.
  `kg/entities/person/jane-liu.md`, `kg/entities/company/acme-corp.md`,
  `kg/entities/meeting/2026-06-24-acme-sync.md`).
- Every entity file **starts with YAML frontmatter** that MUST include these three fields, or the
  write is rejected and reverted:

  ```
  ---
  type: person          # the entity type (person | company | meeting | task | …)
  id: jane-liu          # a stable slug id, unique per type
  title: Jane Liu       # the human title
  ---
  ```

  You may add more frontmatter fields (role, company, tags, etc.) and a markdown body below the
  second `---`. Cross-reference other entities with `[[wikilinks]]` using their title.

## How to work

- **The user is a node too — exactly one `person` with `self: true`.** This workspace ships seeded with
  the FINOS (Fintech Open Source Foundation) ecosystem graph; the **owner** (the user) is the single
  `person` entity whose frontmatter carries `self: true`. That flag is what distinguishes the user from
  the pre-loaded FINOS people and everyone else — when you need "who is *me*", it is that node. Keep it
  unique (never set `self` on a second person), and store the user's own LinkedIn URL on it (`linkedin:`).
- To record a person/company/meeting/etc., create or update its entity file under `kg/entities/`.
- For recurring or scheduled work, use the **scheduling** skill.
- Keep facts dated and attributed where it helps. Do not invent — only record what you were given or
  found.
- You do **not** run git — commits and history happen outside your turn. Just write the files.
- Confirm briefly in your reply what you wrote (e.g. "Created `[[Jane Liu]]`").
