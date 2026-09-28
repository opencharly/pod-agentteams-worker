# AGENTS.md — pod-agentteams-worker

Standalone candy repo for the `agentteams-worker` candy — the Worker runtime
image the AgentTeams controller spawns per Worker custom resource. The entire
candy lives in `charly.yml` at the repo root: the `agentteams-worker:` entity
with its shared `require`s, its service, and a `plan:` that installs the shared
protocol libraries and writes the charly-owned start script. There is no source
tree — the protocol libs come from the pinned AgentTeams source and the runtime
from the shared openclaw / CLI layers.

Canonical files:

- `charly.yml` — the `agentteams-worker:` candy entity (description, `require`,
  `service`, `plan`).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-agentteams:agentteams` — the owning skill for the AgentTeams stack
  (the Manager–Worker model, the composition, both deploy substrates). Load
  before editing, building, deploying, or troubleshooting this candy.
- `/charly-agentteams:agentteams-cli` — the compiled-in `charly agentteams` REST
  CLI the Worker's `agt` client and the controller use.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, package sections, service declarations).
  Load before editing any entity field or plan step.
- `/charly-internals:git-workflow` — before any git/PR action.

The candy carries no `skill:` entity of its own; the family skill
`/charly-agentteams:agentteams` covers the surface. The gap is routed to the named skill-authoring
batch [opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291);
when that lands, add the owning skill here.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly. NB: the repo's `charly.yml`
  is still stamped `2026.249.2125` while the pinned charly requires
  `2026.261.1747`, so the check currently stops at `Run: charly migrate`; that
  is the org-wide 249→261 schema-stamp cutover, tracked as a named batch at
  [opencharly/charly#633](https://github.com/opencharly/charly/issues/633),
  not a defect of this repo. A docs-only change does not carry the migration.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.
- The live R10 witness is the stack's `check-agentteams-pod` bed
  (`charly check run check-agentteams-pod`); there is no worker-only bed.

## Modify this repo

- Edit the `agentteams-worker:` candy entity in `charly.yml`. The start script
  (config pull, file-sync loops, Matrix re-login, gateway exec) and the rootless
  directory pre-creation are authored inline in the `plan:`.
- The `/root` traversal (`chmod 755`) and the uid-1000-writable runtime dirs are
  load-bearing for the rootless posture the upstream hardcoded `HOME` requires —
  preserve them when touching the plan.
- Keep the shared `layer-agentteams-openclaw` / `layer-agentteams-cli` pins and
  the `version:` schema stamp in step.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
