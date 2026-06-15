# Git Source Code — Project Structure Overview

This is the **official Git source repository** (git/git). Git is a distributed version control system written primarily in C.

---

## Top-Level Layout

```
gitcon/
├── git.c                  # Entry point — dispatches all git commands
├── common-main.c          # Shared main() bootstrap
├── Makefile               # Primary build system
├── meson.build            # Alternative Meson build system
├── Cargo.toml / build.rs  # Rust bindings (experimental)
├── builtin/               # One .c file per git subcommand
├── refs/                  # Reference storage backends
├── reftable/              # Reftable ref storage format
├── trace2/                # Structured tracing/telemetry system
├── compat/                # Platform compatibility shims
├── xdiff/                 # Built-in diff engine (xdiff)
├── block-sha1/            # SHA-1 implementations
├── sha1dc/                # SHA-1 collision detection
├── sha256/                # SHA-256 implementation
├── ewah/                  # Compressed bitmaps (EWAH)
├── Documentation/         # All man pages and guides (.adoc)
├── t/                     # Test suite (shell scripts)
├── contrib/               # Third-party tools & integrations
├── git-gui/               # Tcl/Tk GUI client
├── gitk-git/              # Tcl/Tk commit graph viewer
├── gitweb/                # CGI web interface (Perl)
├── po/                    # Translations (gettext .po files)
├── perl/                  # Perl modules used by git scripts
├── templates/             # Default .git/hooks and info templates
├── ci/                    # CI scripts (GitHub Actions, etc.)
├── src/                   # Rust library source (libgit-rs)
├── me/                    # Personal notes (this folder)
└── *.c / *.h              # Core C library files (flat in root)
```

---

## Core Layers (C files in root)

The root is flat by design — every `.c/.h` pair is a module in the shared C library that builtins and plumbing both link against.

| Area | Key Files | What it does |
|------|-----------|--------------|
| Object model | `object.c`, `blob.c`, `commit.c`, `tag.c`, `tree.c` | The 4 Git object types |
| Object storage | `object-file.c`, `odb.c`, `packfile.c`, `loose.c` | Read/write objects from disk |
| Index / staging | `read-cache.c`, `sparse-index.c`, `split-index.c` | The `.git/index` file |
| Refs | `refs.c`, `refs/` subdirectory | Branch, tag, HEAD management |
| Diff engine | `diff.c`, `diffcore-*.c`, `xdiff/` | Computing and displaying diffs |
| Merge | `merge-ort.c`, `merge-ll.c`, `merge-blobs.c` | Merge algorithms (ORT is default) |
| Revision walk | `revision.c`, `commit-reach.c`, `list-objects.c` | Walking commit history |
| Config | `config.c` | `.git/config` and global config parsing |
| Remote / transport | `remote.c`, `transport.c`, `connect.c`, `http.c` | Talking to remotes (SSH, HTTP, git://) |
| Fetch/push | `fetch-pack.c`, `send-pack.c`, `upload-pack.c` | Pack protocol |
| Sequencer | `sequencer.c` | Rebase, cherry-pick, revert logic |
| Ref-log | `reflog.c`, `reflog-walk.c` | Reflog reading and writing |
| Submodules | `submodule.c`, `submodule-config.c` | Submodule handling |
| Pathspecs | `pathspec.c`, `dir.c`, `attr.c` | File matching and .gitattributes |
| Tracing | `trace.c`, `trace2.c`, `trace2/` | Debug and telemetry output |
| Packing | `pack-objects.c`, `pack-bitmap.c`, `midx.c` | Creating and reading pack files |
| Fsmonitor | `fsmonitor.c`, `fsmonitor-settings.c` | Filesystem change monitoring |
| Crypto | `gpg-interface.c`, `credential.c` | GPG signing, credential helpers |

---

## `builtin/` — Git Subcommands

Each file here implements one user-facing command. Examples:

| File | Command |
|------|---------|
| `builtin/commit.c` | `git commit` |
| `builtin/clone.c` | `git clone` |
| `builtin/rebase.c` | `git rebase` |
| `builtin/merge.c` | `git merge` |
| `builtin/log.c` | `git log` |
| `builtin/diff.c` | `git diff` |
| `builtin/fetch.c` | `git fetch` |
| `builtin/push.c` | `git push` |
| `builtin/checkout.c` | `git checkout` / `git switch` |
| `builtin/add.c` | `git add` |

130 commands total. Each registers via `cmd_<name>()` and is dispatched from `git.c`.

---

## `refs/` — Reference Backends

Git supports two storage formats for refs (branches, tags):

- `files-backend.c` — classic loose files + packed-refs (default for existing repos)
- `packed-backend.c` — packed-refs only
- `reftable-backend.c` — new binary reftable format (faster, atomic)
- `reftable/` — the reftable library implementation

---

## `t/` — Test Suite

Shell-based tests using a custom TAP harness:

```
t/
├── t0001-*.sh            # Numbered test scripts (t0000–t9999)
├── helper/               # C test helper binaries
├── lib-*.sh              # Shared test library functions
├── interop/              # Cross-version interop tests
└── chainlint/            # Shell linting for test scripts
```

Run tests with `make test` or individual scripts with `sh t/t1234-foo.sh`.

---

## `Documentation/`

AsciiDoc source for all man pages:

- `git-<command>.adoc` → `man git-<command>`
- `RelNotes/` — release notes for every version
- `CodingGuidelines` — how to write C code for this project
- `MyFirstContribution.adoc` — contributor onboarding guide

---

## `contrib/` — Extra Tools

Not part of the core install, but maintained here:

| Directory | Purpose |
|-----------|---------|
| `contrib/completion/` | bash/zsh/fish tab completion |
| `contrib/diff-highlight/` | Word-level diff highlighting |
| `contrib/subtree/` | `git subtree` command |
| `contrib/credential/` | OS credential helpers (wincred, osxkeychain) |
| `contrib/vscode/` | VS Code extension support |
| `contrib/contacts/` | `git contacts` — find relevant reviewers |

---

## GUI Components

| Directory | Language | Purpose |
|-----------|----------|---------|
| `git-gui/` | Tcl/Tk | GUI for staging, committing, and pushing |
| `gitk-git/` | Tcl/Tk | Visual commit graph browser (`gitk`) |
| `gitweb/` | Perl (CGI) | Web interface for browsing repos |

---

## `src/` — Rust Library (Experimental)

```
src/
├── lib.rs       # Rust crate root
├── hash.rs      # Hash object bindings
├── loose.rs     # Loose object access
├── csum_file.rs # Checksum file handling
└── varint.rs    # Variable-length integers
```

Early work on `libgit-rs`, a Rust interface to Git's C internals via `libgit-sys` (in `contrib/libgit-rs/`).

---

## Build Systems

| File | Purpose |
|------|---------|
| `Makefile` | Primary build — `make all`, `make install`, `make test` |
| `meson.build` | Meson build — `meson setup build && ninja -C build` |
| `configure.ac` | Autoconf script — generates `config.mak` |
| `config.mak.uname` | Per-OS build settings |
| `config.mak.dev` | Developer warnings and extra checks |

---

## CI / Workflows

```
.github/workflows/
├── main.yml          # Main CI — builds on Linux, macOS, Windows
├── check-style.yml   # clang-format style check
├── check-whitespace.yml
├── coverity.yml      # Static analysis
└── l10n.yml          # Translation checks
```

`.cirrus.yml` and `.gitlab-ci.yml` cover additional CI platforms.

---

## Key Concepts for Navigation

- **Plumbing vs Porcelain**: Low-level plumbing (`hash-object`, `update-index`) lives alongside high-level porcelain (`commit`, `merge`) all in `builtin/`.
- **The object database**: Everything is content-addressed. `object-file.c` + `odb.c` are the gateway.
- **The index**: `read-cache.c` owns the staging area. Most commands touch it.
- **Pack files**: Large repos use pack files (`packfile.c`, `pack-objects.c`, `pack-bitmap.c`) for efficiency.
- **Ref storage**: Being migrated from loose files to reftable. Both backends coexist via `refs.c` abstraction.
