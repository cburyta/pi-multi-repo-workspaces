# Pi Multi-Repo Workspaces — Product Requirements Document

**Date:** 2026-10-18

**Status:** Draft

**Version:** v0.1.0

**Route:** Pi package containing an extension and bundled skill

## 1. Summary

`pi-multi-repo-workspaces` is an installable Pi package for multi-repository workspaces. Its first release closes Pi's workspace-root skill-discovery gap by exposing skills under opted-in child repositories while preserving Pi's normal skill validation and lazy-loading behavior.

The package also bundles a `multi-repo-workspace` skill. A user can request a new workspace or an audit in natural language, and the skill guides setup with ordinary Pi tools using the conventions derived from the `_workspaces` templates. Deterministic extension commands that apply scaffold or migration plans are a later phase, not part of v0.1.

## 2. Problem statement

Multi-repository projects need the agent's working directory to be the workspace root so the agent can see shared context and reason across applications. Pi does not automatically discover `.pi/skills` inside arbitrary child repositories. Starting Pi in the root therefore hides useful repository-specific skills under `repos/*/.pi/skills`.

Manual copying or symlinking causes drift, collisions, unclear ownership, and unsafe implicit trust. Users also lack a repeatable way to create or audit the workspace topology.

## 3. Goals

1. Make opted-in `repos/*/.pi/skills` available to Pi sessions started at the workspace root.
2. Ship as a public Git-installable Pi package under `cburyta`.
3. Bundle a skill that can guide workspace creation, migration planning, and audit using Pi's ordinary tools.
4. Provide deterministic status, audit, and refresh commands.
5. Preserve child repositories as the source of truth; do not copy or rewrite their skills.
6. Apply the workspace invariants documented in [`../PLAN.md`](../PLAN.md).
7. Make trust, collisions, diagnostics, and limitations visible.
8. Leave a clean seam for later deterministic scaffold and migration commands.

## 4. Non-goals for v0.1.0

- Universal translation of Claude, Codex, or other vendor-specific skills.
- Automatic loading from every directory named `repos` after a global install.
- Recursive discovery of arbitrary nested repositories.
- Deterministic extension commands that apply repository cloning, migration, or worktree creation; those are Phase 2. The bundled skill may guide the user and use ordinary Pi tools with their normal approvals.
- A replacement for Git, Worktrunk, or Pi's project-trust system.
- Live file watching; startup and `/reload` are sufficient.
- Merging duplicate skills or resolving their semantic conflicts.

## 5. Target users and use cases

### Primary user

A developer with several related repositories who launches Pi from one workspace root.

### Core use cases

- Install the package locally in an existing workspace.
- Ask Pi to set up a new multi-repo workspace.
- Expose a child repository's Pi skills without moving them.
- Add a repository, run `/reload`, and use its skills.
- Audit topology, guidance, memory, repository inventory, and skill-discovery health.
- Preview a migration of an existing non-empty directory.

## 6. Product principles

- **Explicit opt-in:** discovery must not silently expand the trust boundary.
- **Read-only discovery:** resource discovery never writes to repositories.
- **Source ownership:** skills remain in their child repositories.
- **Determinism:** sorted canonical paths make behavior and collisions reproducible.
- **Narrow traversal:** direct children only in the MVP.
- **Agent Skills portability:** compatibility claims are based on the Agent Skills specification, not vendor branding.
- **Auditability:** setup and migration produce structured and human-readable results.

## 7. User experience

### 7.1 Installation

Recommended workspace-local installation:

```bash
cd /path/to/workspace
pi install --local --approve \
  git:github.com/cburyta/pi-multi-repo-workspaces@v0.1.0
pi
```

One-session trial:

```bash
cd /path/to/workspace
pi -e git:github.com/cburyta/pi-multi-repo-workspaces@v0.1.0
```

Global installation is supported but does not enable scanning by itself:

```bash
pi install git:github.com/cburyta/pi-multi-repo-workspaces@v0.1.0
```

Maintenance:

```bash
pi list
pi update --extensions
pi config --local
pi remove --local git:github.com/cburyta/pi-multi-repo-workspaces@v0.1.0
```

`pi update --extensions` reconciles the configured ref but does not advance a pinned tag or commit. Moving to a new release requires changing the package declaration to the new ref. The tagged commands are provisional until Phase 0 verifies them against a published release.

### 7.2 Bundled skill

The package provides:

```text
/skill:multi-repo-workspace
```

Example requests:

```text
/skill:multi-repo-workspace create a workspace for these three Git repositories
/skill:multi-repo-workspace audit this workspace
/skill:multi-repo-workspace plan a safe migration of the existing folder
```

The setup skill must defer ambiguous moves, merges, archives, and deletions to the user. Executable correctness must live in extension commands or the internal workspace module rather than depending only on model interpretation.

### 7.3 Commands

The v0.1 extension exposes:

- `/workspace status` — show activation, root, discovered repositories, loaded skill roots, and warnings.
- `/workspace audit` — perform a read-only topology and resource audit.
- `/workspace refresh` — rerun resource discovery through Pi reload behavior.

Phase 2 adds deterministic `/workspace init` and `/workspace migrate` preview/apply commands. The v0.1 bundled skill can still guide request-driven setup through Pi's ordinary tools, but it must not claim the later transactional guarantees.

## 8. Functional requirements

### 8.1 Packaging

- The public repository must contain a valid `package.json` with the `pi-package` keyword.
- `package.json#pi` must explicitly declare the extension and bundled skill directory.
- Pi host libraries must appear in `peerDependencies` with `"*"`, not runtime `dependencies`.
- Releases must be tagged; documentation must recommend pinned tags or commits.
- README must cover install, trust, setup, audit, update, uninstall, and compatibility.

### 8.2 Workspace activation

- Workspace-local installation is the required v0.1 activation mechanism for child discovery.
- Activation means the package is declared in `<workspace>/.pi/settings.json` and project trust is granted.
- A personal/global installation may expose the bundled package resources, but by itself must never scan child repositories.
- Phase 0 must verify this activation and trust matrix, including `--approve`, `--no-approve`, saved trust, and denial.
- The extension must document that Pi is expected to start at the workspace root.
- Starting below the root must fail closed or provide an explicit root-selection flow; it must not guess across unrelated directories.

### 8.3 Skill discovery

On Pi's `resources_discover` event, the extension must:

1. Use the event's `cwd`, not process-global cwd.
2. Confirm that the workspace is opted in.
3. Enumerate only direct directories under `<cwd>/repos`.
4. Skip symlinked repository roots by default.
5. Look only for `repos/<repo>/.pi/skills` in v0.1.
6. Canonicalize paths and reject escapes outside the workspace.
7. Enumerate individual `SKILL.md` files, then canonicalize, deduplicate, and sort them by repository-relative path.
8. Return absolute skill-file paths.
9. Make no filesystem writes or network requests.
10. Rediscover on startup and `/reload`.

Existing Pi package and project resources precede dynamically returned child paths. Among child paths, sorted repository-relative order defines precedence. Pi retains the first-loaded duplicate name and emits collision diagnostics. Phase 0 must verify the complete precedence against the supported Pi version.

Pi remains responsible for parsing `SKILL.md`, warning about malformed skills, lazy loading, skill commands, and collision diagnostics.

### 8.4 Skill compatibility

- A skill is portable when its `SKILL.md` conforms to the Agent Skills specification and does not require unsupported vendor semantics.
- `.claude/skills` and `.codex/skills` are not automatically supported in v0.1.
- `.agents/skills` remains under Pi's native discovery behavior and must not be duplicated by this extension.
- Vendor rules or instruction files must not be represented as skills merely because they are Markdown.
- Future adapters may add explicitly tested roots and compatibility reports.

### 8.5 Audit

Audit must be read-only and report:

- Whether the cwd is a valid workspace root.
- Required and unexpected top-level paths.
- Repository directories and guidance-inventory drift.
- Nested repositories, missing remotes, and zero-commit repositories.
- Existing Git worktrees and metadata/path mismatches.
- Memory repository initialization, remote presence, nested repositories, and uncommitted files.
- Skill roots, valid skills, malformed skills, name collisions, symlinks, and rejected paths.
- Opt-in and project-trust diagnostics.
- Agent-specific configuration detected but not managed.

Audit must emit both human-readable PASS/FAIL output and a stable machine-readable result.

### 8.6 Guided setup in v0.1

The bundled skill must explain and check the workspace template mapping in [`../PLAN.md`](../PLAN.md). It may guide Pi through ordinary file and Git tools, subject to normal approvals, but v0.1 does not promise a transactional scaffold engine.

The Phase 2 deterministic scaffold/migration specification must require dry-run for non-empty roots, item-level approval for ambiguity, local-only `memory/`, generated repository inventory, preservation of unknown configuration, idempotency, and partial-failure recovery.

## 9. Security and trust requirements

- Documentation must state that Pi extensions run with the user's OS permissions and skills can instruct the model to run commands.
- Globally installed extension code must not treat `ctx.isProjectTrusted()` alone as proof that child resources were explicitly approved.
- Unknown symlinks and path escapes must fail closed.
- Discovery must never execute skill scripts.
- Setup must never copy credentials, authentication configuration, or `secrets/` contents into generated context.
- A denied trust decision must result in no child-repository skills being exposed.
- Diagnostics should name accepted and rejected resource paths without revealing secret content.

## 10. Suggested package layout

```text
pi-multi-repo-workspaces/
├── package.json
├── README.md
├── LICENSE
├── extensions/
│   └── index.ts
├── skills/
│   └── multi-repo-workspace/
│       ├── SKILL.md
│       └── references/
├── src/
│   ├── workspace/
│   ├── discovery/
│   └── validation/
├── test/
└── docs/
```

This is a target for the technical specification, not a frozen implementation decision.

## 11. Quality and compatibility requirements

Required test coverage:

- Missing, empty, and populated `repos/` directories.
- Malformed skills and missing descriptions.
- Duplicate names and deterministic winners.
- Symlinked repositories, symlinked skill roots, and path escapes.
- Reload after adding or removing a repository.
- Local trust accepted and denied.
- Global install without opt-in.
- Execution from the workspace root and from a child path.
- Interactive, print, JSON, and RPC modes without assuming terminal UI.
- Audit fixtures covering empty, valid, drifted, and operational-error workspaces.

Phase 2 adds empty create, migration preview, idempotency, and interrupted-apply coverage.

Supported platform and Pi versions must be chosen in the technical specification and enforced in CI.

## 12. Success metrics

The MVP succeeds when:

- A user installs from `cburyta` using one documented Git command.
- Pi started at an opted-in workspace root advertises valid child skills without copying them.
- Adding a repository followed by `/reload` makes its valid skills available.
- No child skills load for an untrusted or non-opted-in workspace.
- `/workspace audit` identifies known drift in a representative workspace.
- The bundled setup skill documents and validates the expected topology without claiming Phase 2 transactional behavior.

## 13. Risks and mitigations

- **Implicit trust expansion:** require explicit project opt-in and path allowlisting.
- **Instruction injection from child skills:** preserve Pi warnings, show provenance, and load only approved roots.
- **Skill collisions:** existing Pi resources load before dynamic child skills; sorted individual child paths provide stable child ordering, and diagnostics expose first-wins collisions.
- **Cwd ambiguity:** require root cwd for v0.1.
- **Vendor incompatibility:** claim only tested Agent Skills compatibility.
- **Migration data loss:** dry-run, explicit approval, atomic writes where practical, and recovery reports.
- **Scope growth:** ship discovery/status/audit before full migration automation.

## 14. Open decisions and validation spikes

1. Validate project-local activation and the full trust matrix; global-only installation must not scan.
2. Validate exact Git install/update/remove commands against tag `v0.1.0`.
3. Measure and document resource collision precedence, including filesystem-order independence from returning individual sorted files.
4. Test representative Pi, Agent Skills, Claude-specific, and Codex-specific skill fixtures.
5. Decide how a child-started Pi session fails closed or selects a root explicitly.
6. Validate audit schema `multi-repo-workspace.audit.v1`: stable ordering; `pass`, `warn`, `fail`, and `error`; and headless exit codes 0, 1, and 2.
7. Validate `workspace.toml` schema v1 and root discovery.
8. Define fixed empty, valid, drifted, collision, symlink, trust-denied, and operation-error fixtures with expected audit results.

## 15. Acceptance criteria

- [ ] Public Git repository and tagged release install successfully with Pi.
- [ ] Package declares exactly the intended extension and bundled skills.
- [ ] Local installation and project trust are documented and tested.
- [ ] Discovery is narrow, canonical, deterministic, deduplicated, and read-only.
- [ ] Startup and `/reload` both reflect repository skill changes.
- [ ] Symlinks and escapes fail closed by default.
- [ ] A personal/global-only install does not scan; child discovery activates only from a trusted project-local package declaration.
- [ ] Skill provenance and collisions are visible.
- [ ] Bundled setup skill is invocable directly.
- [ ] Audit emits human and machine-readable results.
- [ ] No vendor-specific compatibility is claimed without tests.
- [ ] README includes install, usage, trust, update, and uninstall instructions.

## 16. Audit log

- **2026-10-18 — technical/platform review:** clarified v0.1 scope, project-local activation, pinned updates, resource precedence, individual-file ordering, and Phase 0 trust/collision fixtures.
- **2026-10-18 — product/spec review:** aligned the release boundary with the roadmap, mapped source-template requirements to Pi package behavior, and defined manifest/audit contracts.
- Reviews were context-isolated within the same provider family; they were independent reviews, not cross-family verification.

## 17. Primary references

- Pi package documentation: <https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/packages.md>
- Pi extension documentation: <https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/extensions.md>
- Pi skills documentation: <https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/skills.md>
- Pi dynamic-resources example: <https://github.com/earendil-works/pi/tree/main/packages/coding-agent/examples/extensions/dynamic-resources>
- Agent Skills specification: <https://agentskills.io/specification>
- Internal research input (not a portable repository dependency): `_workspaces/WORKSPACES_TEMPLATE-20261007.md` from the author's local template collection
