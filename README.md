# agentteams-worker

The AgentTeams Worker agent image, as an OpenCharly candy.

In the AgentTeams Manager–Workers model the Workers execute the tasks the
Manager delegates. This candy is the **Worker runtime image**: the controller
spawns one container from it per Worker custom resource. It composes the shared
openclaw gateway runtime and the shared `agt` / `mc` CLI, then installs the
shared AgentTeams protocol libraries from the pinned AgentTeams source.

A charly-owned start script sources those libraries, pulls the worker config
(`openclaw.json`, `SOUL.md`, `AGENTS.md`, skills) from MinIO, starts the
file-sync loops, re-logins to Matrix for a fresh E2EE device, and execs the
openclaw gateway. Workers are stateless — all state lives in MinIO. The service
runs rootless as the image user (uid 1000).

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `agentteams-worker` |
| Runtime | openclaw gateway + `agt` REST client + `mc` + shared protocol libs |
| Service | `agentteams-worker` (spawned by the controller) |
| Requires | `layer-agentteams-openclaw`, `layer-agentteams-cli` |

## How to use it

This candy is consumed **indirectly**: the controller spawns Worker containers
from the image built on top of it. Point the controller at that image with
`AGENTTEAMS_WORKER_IMAGE`. The full stack composes it for you:

```yaml
my-agentteams:
  candy:
    base: cachyos.cachyos
    candy:
      - '@github.com/opencharly/pod-agentteams-worker:<tag>'
```

Build the image with the charly CLI:

```bash
charly box build my-agentteams
```

See the owning skill for the composition, the Manager–Worker model, and both
deploy substrates.

## Layout

- `charly.yml` — the `agentteams-worker:` candy entity: the two shared requires,
  the `agentteams-worker` service, and the plan that installs the protocol libs,
  pre-creates the rootless runtime directories, and writes the start script.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-agentteams:agentteams` — the full stack composition.
- CLI: `/charly-agentteams:agentteams-cli` (`charly agentteams`).
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
