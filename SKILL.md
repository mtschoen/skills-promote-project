---
name: promote-project
description: Use when a brainstorm has fixed a project's tech stack and the user wants to start real work - "promote this to a real project", "scaffold this", "set up the bells and whistles", "make this a real repo", "wire up the linters and hooks". Also when adopting an already-created bare folder (e.g. from capture-idea) into a compliant git repo. Registers with the project-tracker MCP server when available; otherwise records the project in ~/.project-tracker/projects.json.
---

# promote-project

Turn a bare idea folder (or a brand-new project) into a real, convention-compliant git repo: history, base config, stack-aware linters + on-save hooks, an [`aislop`](https://github.com/scanaislop/aislop/) gate - the recommended lint/gate stack, `AGENTS.md`, a language manifest, CI, remote hosting, and a project-tracker registration.

**Core principle: reference the shared definition, don't reinvent it.** What a compliant project contains is defined once and consumed by both *birth* (this skill) and *upkeep* (`project-maintenance`):

- `~/.agents/project-conventions.md` - if the user maintains this personal dotfile, it's the checklist + the `.editorconfig` template. If it doesn't exist, fall back to the checklist and defaults already spelled out inline in this skill's Tier 1 / Tier 2 steps below - same procedure, just not centralized in a personal file.
- If the `project-maintenance` skill is installed, its `references/cross-project-config.md` - the **literal shapes** to paste (on-save hook JSON, CI skeleton, `aislop` config, `AGENTS.md`-pointer form, post-clean init scripts, and `.gitattributes` eol rules). If that skill isn't installed (or an older version lacks the relevant section), apply the essence directly as described inline in Tier 2 below - same procedure, just not centralized in a shared reference.
- The aislop section of the user's global `AGENTS.md` - **the pinned `aislop` version and hook-install rules**.

This skill *applies* those when present; it carries no copies that would rot, and falls back to the defaults described inline here when a referenced file (or named section) isn't available.

## When this fires

After a brainstorm has decided the stack and the user wants to start coding:

- "promote this to a real project" / "make this a real repo"
- "scaffold this" / "set up the bells and whistles"
- "wire up the linters and hooks"
- adopting an existing bare folder (e.g. one `capture-idea` created) into a full repo

**When NOT:** still mid-brainstorm with no stack decided (do Tier 1 only, defer Tier 2 - you can't choose ruff-vs-eslint yet); an already-compliant repo that just needs upkeep (that's `project-maintenance`); saving a raw idea with no commitment to build (that's `capture-idea`).

## Ordering - works *with* the brainstorming gate

`superpowers:brainstorming`'s HARD-GATE forbids implementation before a design is approved. Resolve the tension by tier:

- **Tier 1 (base) is gate-exempt and happens early.** Folder + `git init` + registration + the agent-instruction *skeleton* are not "implementation" - same precedent as `capture-idea`. On "I'm starting a project," lay the base, *then* brainstorm.
- **Tier 2 (stack-aware) runs after the brainstorm fixes the stack.** This is necessary ordering, not a delay - the stack determines the linter, manifest, hooks, and the build/test content of `AGENTS.md`.

## Procedure

### Tier 1 - deterministic base (language-agnostic)

Create from `~/.agents/project-conventions.md`'s base checklist:

1. `git init` (default branch `main`) - unless adopting a folder that's already a repo.
2. `.editorconfig` (from the conventions template), `.gitattributes` (`* text=auto`), `.gitignore` for the stack.
3. `README.md` stub (what it is, not how to build) and the roadmap file (see PLAN note below).
4. **Agent-instruction skeleton:** `AGENTS.md` (sections stubbed) + `CLAUDE.md` / `GEMINI.md` as literal **`@AGENTS.md`** import pointers (the directive, NOT a markdown link - a link does not auto-load). The *structure* is language-agnostic; the build/test/architecture *content* gets filled in Tier 2 once the stack is fixed.
5. **Register in project-tracker now** - don't assume auto-discovery (see Registration below).
6. Initial commit.

**Roadmap file:** `project-conventions.md` calls for `.plan` (terse bullets). If adopting a `capture-idea` folder it will already have a `PLAN.md` - **do not clobber it**: project-tracker's task detection reads `PLAN.md`/`TODO.md`, not `.plan`. Keep the existing `PLAN.md` for tasks; add a `.plan` only if the user wants the terse-roadmap form too. Surface the choice rather than silently renaming.

### Tier 2 - stack-aware (after the stack is fixed)

If the `project-maintenance` skill is installed and up-to-date, apply the literal shapes from its `references/cross-project-config.md`; otherwise (or if an older version lacks a referenced section) apply the essence directly: a bare `@AGENTS.md` import pointer in `CLAUDE.md`/`GEMINI.md`, a `PostToolUse` ruff/shellcheck on-save lint hook in `.claude/settings.json` (adapt to your harness's equivalent mechanism), `aislop hook install claude --project` for the aislop wiring, a minimal lint + format-check + test CI workflow, an `aislop` gate config (`.aislop/config.yml`) with `ci.failBelow: 80`, and the `init.ps1`/`init.bat`/`init.sh` post-clean scaffold. Either way, tailor to the decided stack, using the stack's standard tools (Python: ruff; JS/TS: eslint + prettier; shell: shellcheck):

1. **Fill `AGENTS.md`** as the single source of truth: build/test commands, architecture, conventions. Keep only genuinely tool-specific content (harness-specific hooks/settings paths, the `Skill` tool, subagent routing) *below* the `@AGENTS.md` line in `CLAUDE.md`.
2. **Language manifest** (e.g. `pyproject.toml`) with the test runner + a coverage gate. The coverage bar is **`fail_under = 100`** - not a softer number. (This pytest line-coverage gate is distinct from the `aislop` *score* gate in step 4.)
3. **Linter config** for the stack + a `PostToolUse` on-save lint hook in `.claude/settings.json` for the stack's own linters (copy the canonical ruff/shellcheck hook shape from `cross-project-config.md`; tailor the `case` arms to the repo's languages).
4. **`aislop` gate** (`.aislop/config.yml`) - `ci.failBelow` per `cross-project-config.md` (reference: 80). Disable `python-formatting`/`python-linting` (ruff owns those); note the `from __future__ import annotations` false positive. Pin the version.
5. **Run `aislop hook install claude --project`** to wire the aislop `PostToolUse` hook. The current contract: run the installer, do not hand-write the aislop hook entry. The generated `.claude/AISLOP.md` and `.claude/CLAUDE.md` stay gitignored (`.claude/*` + `!.claude/settings.json` in `cross-project-config.md`'s Local settings tracking convention) and are regenerated by the init script in the next step, not tracked. The tracked `.claude/settings.json` carries the managed hook entry the installer writes - a bare `aislop hook claude` call, never `npx` and never `@latest`.
6. **Post-clean scaffold**: create `init.ps1`, `init.bat`, and `init.sh` at the repository root, the `.gitattributes` `*.sh`/`*.bat`/`*.cmd` eol rules, and the AGENTS.md "Cleaning the working tree" section. If the `project-maintenance` skill is installed and its `cross-project-config.md` contains the "Safe git clean + post-clean init" section, copy the exact target shapes and required script steps from there. If that skill is not installed or its reference lacks that section (such as an older install), apply this self-contained essence - every init script must be idempotent (safe to run twice with no worse outcome the second time) and cover every step below, since a partial script leaves the checkout stuck between "cleaned" and "restored":
   - **Each of `init.ps1` / `init.sh`**, in order: (a) check prerequisites - package manager/SDK, any platform build tooling, and `aislop` - and report what's missing without installing anything; (b) restore dependencies via the stack's normal restore command; (c) run `aislop hook install claude --project`, skipped with a hint when `aislop` is not on PATH, to regenerate `.claude/AISLOP.md` and `.claude/CLAUDE.md`; (d) restore the tracked `.claude/settings.json` only when the installer left a line-ending-only diff (`git diff --ignore-cr-at-eol --quiet -- .claude/settings.json` clean but ordinary `git diff --quiet` or `git status --porcelain` reports it modified) - never when there is a real content change; (e) if a settings-provisioning tool is on PATH, call it by a feature name (never its bare pipeline command) to re-apply the project-scope settings a clean removed, skipping silently when the tool is absent; (f) an optional build behind an off-by-default flag (`-Build` for PowerShell, `--build` for bash); (g) print a summary of what ran and what prerequisites are still missing.
   - **`init.bat`** is a thin forwarder, not a reimplementation: resolve `pwsh` (fall back to Windows PowerShell), forward every argument to `init.ps1`, and propagate its exit code. This is required, not optional - `cmd.exe` cannot invoke `init.ps1` directly, so without the forwarder the documented recovery path silently does nothing under `cmd.exe`: it reports the file "not recognized" and the shell moves on, with no restore step ever running.
   - **`.gitattributes`**: `*.sh text eol=lf`, and `*.bat text eol=crlf` / `*.cmd text eol=crlf`, each with a comment naming the checkout failure the rule prevents (a CRLF `.sh` fails at the shebang on Linux; an LF-only `.bat` mis-parses on Windows).
   - **AGENTS.md "Cleaning the working tree" section**: the invariant paragraph (`git clean -ffxd` must always be safe to run, and why an exclude list would defeat the point of cleaning); a Removed/Restored-by table built from this repo's own `git clean -n -ffxd` output, not copied from another repo (build-output directories and generated-file names differ by stack); the per-platform "getting back to work" invocation (PowerShell, `cmd.exe` noting the `.\` prefix a `NoDefaultCurrentDirectoryInExePath=1` machine requires, and bash); the provisioner paragraph; and the prerequisites line.
7. **CI** (`.github/workflows/` or `.gitea/workflows/` depending on hosting) running lint + format-check + tests. Don't skip it.
8. **`LICENSE`** (MIT default).
9. Stack-specific must-haves - e.g. Python: a `requirements.txt` mirroring deps (with a one-line note that `pyproject.toml` is the install source of truth), so `aislop`'s security engine can audit them.

### Remotes + registration

- **Remote Hosting:** Set up git remotes (`origin` / `github` / `gitea` as configured for the workspace) and push `main`. Check workspace rules or user preference for default hosting locations and credentials.
- **project-tracker registration is mandatory, explicit, and early** (Tier 1 step 5):
  - Fresh empty project → project-tracker MCP `create_project`.
  - **Adopting an existing folder** (the common capture-idea case - `create_project` refuses existing folders) → project-tracker MCP **`register_existing_path`** (params: `path`, optional `name`/`description`/`status`). Fallback if that tool is absent: the CLI `project-tracker project register <path>`. Do NOT call `create_project` on an existing folder - it raises `FileExistsError`.
  - Verify it registered (`list_projects`/`get_project`) - don't assume `project-tracker scan` will find it (the CLI scan ignores `manual_include` paths).
  - **If neither the MCP server nor the CLI is available**, append `{name, path, status, description}` to `~/.project-tracker/projects.json` (create the file with `[]` if missing) and verify by reading it back. The registry is an *agent-maintained* convention for installs without project-tracker - the agent owns the file; the project-tracker tool never reads it (its own store is a SQLite database under `~/.project_tracker/`, note the underscore). Entries promote into the real tracker later with `project-tracker project register <path>`.

## Adopting an existing folder

Common case: `capture-idea` already made the folder, or the user hand-created it. Then: skip folder creation; `git init` only if not already a repo; register via `register_existing_path` (not `create_project`); keep the existing `README.md`/`PLAN.md`; layer Tier 1 + Tier 2 onto what's there without clobbering existing files.

## Verify before declaring done

The baseline failure mode is an ad-hoc scaffold with internal inconsistencies. Before reporting complete, confirm:

- [ ] **Manifest ↔ files agree** - every declared entry point / script / package actually exists (no `cli:main` without a `cli.py`).
- [ ] **No empty tracked-intent dirs** - git ignores empty dirs; commit a stub (e.g. `templates/base.html`) or omit the dir.
- [ ] **`CLAUDE.md` is a `@AGENTS.md` pointer**, not duplicated full content; `AGENTS.md` exists and is filled.
- [ ] **Stack must-haves present** (e.g. Python `requirements.txt`).
- [ ] **Registered in project-tracker** - MCP/CLI, or the JSON registry in fallback mode (verify it appears, don't assume).
- [ ] **Lint/test gate runs clean** - actually run the linter + tests once. If you genuinely can't execute locally (restricted sandbox), say so explicitly to the user and confirm CI will catch it on first push - don't silently claim it passes.
- [ ] **Any deviation from the conventions was surfaced to the user**, not made silently.

## Common mistakes

| Mistake (seen in baseline) | Fix |
|---|---|
| `CLAUDE.md` with full content, no `AGENTS.md` ("personal tool, no multi-agent need") | `AGENTS.md` is the convention regardless of audience; `CLAUDE.md` = `@AGENTS.md` pointer |
| Skipping registration, assuming `project-tracker scan` finds it | Register explicitly in Tier 1 - scan misses `manual_include` paths |
| Calling `create_project` on an existing folder | Use `register_existing_path` for adopt; `create_project` is for fresh empty folders only |
| Declaring an entry point / dep without creating the file | Verify manifest ↔ files before done |
| Silently deviating from a convention (CRLF→LF) | Make the call, but *surface it* to the user |
| Softening the coverage/quality bar to pass | Match the fleet bar (100% coverage); fix the code, not the gate |
| Skipping CI "because sandbox / small project" | Add at least the minimal lint+test workflow |
| Hand-writing the aislop `PostToolUse` hook entry | Run `aislop hook install claude --project`; it manages that entry and the generated `.claude/AISLOP.md`/`.claude/CLAUDE.md` |
| Copying the hook/CI/aislop blobs inline and editing | Reference `cross-project-config.md` (or inline essence when absent or section is missing); keep one source of truth |
| Committing `.claude/AISLOP.md` or `.claude/CLAUDE.md` | They are generated boilerplate; keep them gitignored and regenerate with `init.ps1`/`init.sh` after a clean |
| Skipping the `init.ps1`/`init.bat`/`init.sh` scaffold ("small project") | Every promoted repo needs a safe `git clean -ffxd`; scaffold the init scripts per `cross-project-config.md` (or inline essence if section is absent) |
