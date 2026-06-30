# agent · workspace-seeds

Seed workspaces planted into a fresh agent run. Each subdir is one named template; `seeding.resolve_seed_dir`
selects one by name (config `default_template`, default `default`), so adding a flavor is a new subdir —
no code change. A subdir is a valid seed iff it carries `CLAUDE.md` (`validate_seed`).

- `default/` — the baseline seed checked out into a new agent workspace before the first turn.
- `finos/` — a FINOS-flavored knowledge-worker seed: the `default` baseline **plus** a pre-populated
  knowledge graph of the FINOS ecosystem (`kg/entities/` — companies, organizations, people, projects)
  and the `routines/finos-discovery.md` job that grows it. Captured from the `finostest@vexa.ai`
  workspace. Note the discovery routine ships `enabled: true` (reconciled onto the durable scheduler on
  seed) — flip its frontmatter to `false` if you want it dormant until the user opts in.
