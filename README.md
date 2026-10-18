# pi-multi-repo-workspaces

Multi-repository workspace scaffolding and child-repository skill discovery for Pi.

> This repository is in product-definition stage. No installable release exists yet.

## Product direction

Build a Pi package that:

- bundles a `multi-repo-workspace` setup skill;
- exposes approved `repos/*/.pi/skills` to Pi sessions started at the workspace root;
- audits workspace topology, repository inventory, guidance, memory, and skill discovery;
- installs from GitHub through Pi's package manager.

## Planning documents

- [Product plan](docs/PLAN.md)
- [Pi package PRD](docs/prds/pi-extension-v0.1.0.md)

## Planned installation

The first tagged release is intended to install with:

```bash
cd /path/to/workspace
pi install --local --approve \
  git:github.com/cburyta/pi-multi-repo-workspaces@v0.1.0
```

The command remains provisional until release validation is complete. The PRD defines the planned trust, usage, update, uninstall, and compatibility behavior.

## Status

- Pi platform research complete.
- Pi v0.1.0 PRD drafted and independently audited.
- Initial activation, manifest, audit-schema, refresh, and package decisions recorded.
- Next step: approve the remaining product gates, then produce the technical specification and Phase 0 validation tasks.

## License

[MIT](LICENSE)
