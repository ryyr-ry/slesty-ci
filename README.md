# slesty-ci

Public CI harness for [Slesty](https://github.com/ryyr-ry/Slesty) (private).

**This repository contains NO source code.** The Slesty codebase is private.
This repository exists to run the build and test pipeline on **public GitHub
Actions runners**, on the owner's instruction, and to publish the resulting
binaries and test logs for download.

## How it works

1. A `workflow_dispatch` (manual) or scheduled run starts on a public runner.
2. The runner clones the **private** `ryyr-ry/Slesty` repository at runtime,
   authenticating with a **read-only SSH deploy key** stored in this
   repository's **encrypted GitHub Secrets** (`SLESTY_DEPLOY_KEY`). The key
   is scoped to that single repository, is read-only (cannot push), and is
   never printed or committed.
3. It runs the full pipeline: `cargo fmt --check`, `cargo clippy -D warnings`,
   `cargo test --workspace`, `cargo build --release`.
4. The runner uploads the release binaries and the complete test logs as
   workflow **artifacts**, and (on success) publishes a **public release**
   containing the binaries.
5. The cloned source is discarded with the ephemeral runner. At no point
   does any source file, build script, or manifest from the private
   repository get committed to, or displayed in, this repository.

## Artifacts

- `slesty-linux-<sha>` / `slesty-macos-<sha>` / `slesty-windows-<sha>`:
  release binaries per OS.
- `test-logs-<os>-<sha>`: full `cargo test` output.
- Releases: `vX.Y.Z` tags built from the tested commit, with binaries for
  all three platforms attached.

## Why a separate public CI repository?

The owner wants CI to run on GitHub's public runner pool while keeping the
source private. A public repository with a runtime read-only deploy key
achieves both: the Actions infrastructure is public (visible run history,
public artifacts), the code is not. The deploy key grants `contents:read`
on the private repository only — it cannot write, cannot touch any other
repository, and is revocable from Slesty's Settings → Deploy keys at any
moment.

## Triggering

```sh
gh workflow run ci.yml -r ryyr-ry/slesty-ci
```

or from the Actions tab. Runs also trigger on a `cron` schedule (weekly) to
keep binaries fresh against toolchain updates.
