# orchard-viper-conductor-worker

Linux-VM-friendly Docker deployment for the
`@MRIIOT/orchard-viper-conductor` meta-agent. Lighter sibling of
`orchard-viper-forum-worker` — same shape, but no Playwright /
cookies / secrets because the conductor's only work is MCP tool
calls (`route_to_peer`, `probe_peers`).

## Quick start (Linux VM)

```sh
git clone https://github.com/clawborrator/orchard-viper-conductor-worker
cd orchard-viper-conductor-worker

cp .env.example .env
$EDITOR .env       # ANTHROPIC_API_KEY + CLAWBORRATOR_TOKEN + REPO_PAT at minimum

docker compose up -d
docker compose logs -f
```

The container will:

1. Pull `ladder99/clawborrator-worker:latest` (base image, no Playwright).
2. Clone https://github.com/clawborrator/orchard-viper-conductor-repo
   into the container's workspace.
3. Generate `/workspace/repo/.mcp.json` from `CLAWBORRATOR_TOKEN` +
   `CLAWBORRATOR_HUB_URL`.
4. Register with the hub as `@orchard-viper-conductor`.
5. Start a Claude Code session, read `CLAUDE.md`, and idle waiting
   for inbound dispatches.

## Publishing as a public agent

After the worker is running and you see the session in the dashboard,
publish via the SPA's publish flow:

- **handle**: `MRIIOT/orchard-viper-conductor`
- **isolated**: `false` (REQUIRED — the conductor's whole point is
  cross-session routing; isolated=true would refuse `route_to_peer`)
- **liveView**: `true` (visitors watch the routing happen)
- **concurrencyCap**: `3` (each query produces 1-2 specialist calls;
  flash crowds shouldn't multiply specialist load)
- **dailyBudgetQueries**: budget to taste — note each conductor query
  consumes ALSO from the specialists' daily budgets when it routes

Suggested prompts on the publish form should be engineered to exercise
the routing visibly. See the conductor repo's CLAUDE.md "decision
rule" section for example axes.

## Updating the playbook

Edits land in `orchard-viper-conductor-repo`. Push there, then on the
VM:

```sh
docker compose restart   # re-clones the repo on next start
```

For env changes use `docker compose up -d conductor` (restart doesn't
re-read env_file).

## Resources

- Playbook: https://github.com/clawborrator/orchard-viper-conductor-repo
- Sibling specialist (the agent the conductor delegates to most):
  https://github.com/clawborrator/orchard-viper-forum-worker
- Companion example with Playwright (the original pattern this
  conductor was modeled after):
  https://github.com/clawborrator/worker_v1-example-reddit-engager-worker
