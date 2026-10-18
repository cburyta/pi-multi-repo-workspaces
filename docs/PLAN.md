# Pi Multi-Repo Workspaces — Product Plan

**Date:** 2026-10-18

**Status:** Draft for review

**Decision:** Build a focused Pi package for multi-repository workspace setup, audit, and child-skill discovery.

## Problem

A multi-repository project needs a workspace-level operating model: principal checkouts, task worktrees, shared notes, secrets, scripts, and concise instructions that explain how the repositories relate.

Pi should start at the workspace root so it can reason across repositories. Today that creates a discovery gap: Pi sees workspace-level resources, but does not automatically discover skills under each `repos/<repo>/.pi/skills` directory.

## Product thesis

Ship one Pi package containing:

```text
pi-multi-repo-workspaces
├── extension
│   ├── child-skill discovery
│   ├── workspace status
│   ├── read-only audit
│   └── resource refresh
├── bundled setup skill
└── internal workspace model
    ├── manifest
    ├── validation
    └── guidance and resource inventory
```

The extension owns deterministic discovery, status, and audit behavior. The bundled skill lets a user request workspace setup or migration in natural language while using Pi's ordinary tools and approval model.

## Why start with discovery and audit

1. Pi's `resources_discover` event directly solves the child-repository skill gap.
2. A Pi package can distribute executable extension behavior and the setup skill together.
3. Project-local installation gives the workspace an explicit activation and trust boundary.
4. Discovery, status, and audit can ship before transactional migration automation.
5. The package remains useful for both existing and newly created workspaces.

## Delivery sequence

### Phase 0 — Validate risky assumptions

- Prove Git install, update, removal, local trust, startup discovery, and `/reload` behavior against a tagged repository.
- Prove collision order across bundled, workspace, child-repository, and `.agents/skills` resources.
- Validate the v0.1 activation decision: child discovery requires a project-local package declaration in the workspace root's `.pi/settings.json`; a personal/global install alone never scans child repositories.
- Validate `workspace.toml` schema v1, the root-discovery algorithm, and audit schema `multi-repo-workspace.audit.v1`.
- Confirm that the implementation can remain an internal `src/core` module through v0.1.
- Test symlinks, path escapes, malformed skills, denied trust, child-directory startup, and non-interactive Pi modes.

### Phase 1 — Pi MVP

- Publish a valid Pi package.
- Bundle the `multi-repo-workspace` setup skill.
- Expose opted-in child-repository Pi skills.
- Add `/workspace status`, `/workspace audit`, and `/workspace refresh` commands.
- Let the bundled skill guide request-driven setup with ordinary Pi tools.
- Document local and global installation, pinned-update behavior, trust, limitations, and uninstall.

### Phase 2 — Deterministic scaffold and migration

- Add transactional `create`, `migrate`, and reusable validation behavior.
- Generate concise `AGENTS.md` and optional Pi workspace configuration.
- Initialize the local-only `memory/` repository.
- Require previews and explicit approval for ambiguous or destructive changes.
- Produce recovery reports for partial failure and interruption.

## Workspace model

```text
<workspace>/
├── AGENTS.md
├── .pi/
│   └── settings.json
├── workspace.toml
├── repos/
├── worktrees/<repo>/<branch>/
├── memory/
│   ├── README.md
│   ├── tickets/
│   ├── lessons/
│   ├── fleeting/
│   └── archive/YYYY/MM/<slug>/INDEX.md
├── scripts/
└── secrets/
```

Core invariants:

- The workspace root is not a Git repository.
- Principal checkouts live under `repos/`; development occurs in worktrees.
- Network refresh is an explicit user action before investigation. Audits report local upstream/fetch age without fetching; refresh failures are surfaced.
- `memory/` is a separate local Git repository with no remote.
- Secrets never enter workspace memory or generated context.
- Existing content is preserved unless the user explicitly approves a change.
- Generated guidance and repository inventories reflect the filesystem.
- Auditing is read-only and idempotent.

## Mapping from the source template

- `AGENTS.md` is the default workspace guidance file and contains the Apps inventory, folder ownership, refresh policy, ticket/branch policy, and worktree workflow.
- Ticket identification is configurable and optional by default; when enabled, branch naming includes the ticket.
- Native Git is the baseline worktree manager. Worktrunk is an optional integration, not an unconditional dependency.
- The memory commit policy, fleeting-note lifecycle, dated archives, and required archive `INDEX.md` are scaffold and audit requirements.
- Pi configuration is merged rather than replacing unknown settings.
- The setup-prompt comment from the source template is research input, not generated final guidance.

## Product decisions

Decided for the first technical specification:

1. **Manifest:** `workspace.toml`, schema version 1, is the workspace contract.
2. **Activation:** v0.1 child discovery requires the project-local package declaration in `<workspace>/.pi/settings.json`; a personal install alone does not activate scanning.
3. **Audit contract:** `multi-repo-workspace.audit.v1` uses stable path ordering and `pass`, `warn`, `fail`, and `error` findings. Headless exit codes are `0` (no failures), `1` (validation failure), and `2` (operation error). Interactive commands render the same result without promising a process exit code.
4. **Implementation boundary:** use an internal `src/core` module through v0.1.
5. **Refresh:** network operations are explicit; audit never fetches.
6. **Skill scope:** v0.1 dynamically adds only `repos/*/.pi/skills`; Pi's native `.agents/skills` behavior remains untouched.

Remaining specification gates:

1. Should repository descriptions be user-authored only, generated as drafts, or inferred from repository metadata?
2. How should a Pi session started below the workspace root fail closed or select the root explicitly?
3. Which Pi and operating-system versions are supported by the first release?
4. Does deterministic scaffold application belong in v0.1 or the next minor release after discovery and audit?

## Sources

- Internal research inputs: `_workspaces/WORKSPACES_TEMPLATE-20260825.md`, `WORKSPACES_TEMPLATE-20260922.md`, and `WORKSPACES_TEMPLATE-20261007.md` from the author's local template collection
- Pi packages: <https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/packages.md>
- Pi extensions: <https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/extensions.md>
- Pi skills: <https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/skills.md>
- Agent Skills specification: <https://agentskills.io/specification>
