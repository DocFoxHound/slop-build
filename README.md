# Slop Build

The shared build recipe makers use to ship on Slop. A maker adds
one small workflow file to their repository (Slop's **Add Slop Build** button fills it in; see
`examples/slop.yml`), and each push to their main branch or `v*` tag:

1. builds the app for Windows and Linux on GitHub-hosted runners,
2. packs each build (Windows: a `.zip`; Linux: a `.tar.gz` plus `.deb` and `.rpm` packages that
   install to `/opt/<app>` with `/usr/bin/<app>`) and signs every file with a GitHub artifact
   attestation (Sigstore), which names the repository, the commit and this recipe ("Built by Slop
   Build from source"),
3. attaches the files to a release on the maker's repository (Slop stores no builds),
4. sends the release to Slop with the run's GitHub OIDC token. Slop checks the token was issued
   to this recipe, in the app's repository, at this commit; no secret is shared.

Slop then fetches and hashes the files, verifies the attestations, and the release goes to review.

## Project types

| `project-type` | What it does |
|---|---|
| `rust` | `cargo build --release --locked`; ships the first binary target |
| `go` | `go build -trimpath`; ships a binary named after the repository |
| `custom` | Runs `build-command`, ships `output-dir`, starts `exec` (`{ext}` becomes `.exe` on Windows) |

Godot, web/HTML5, Electron/Tauri and Python projects use `custom` for now.

## Inputs

| Input | Meaning |
|---|---|
| `app` | The app's store address on Slop (required) |
| `slop-url` | The Slop site (required) |
| `project-type` | `rust`, `go` or `custom` (required) |
| `platforms` | `windows,linux` by default |
| `linux-packages` | `deb,rpm` by default; Slop keeps the ones the store page lists |
| `build-command`, `output-dir`, `exec` | `custom` only |
| `sign` | `true` by default; `false` for private repositories without GitHub Enterprise |

## Where this lives

This folder is the recipe's source, published as its own public repository
(`DocFoxHound/slop-build`, at `.github/workflows/build.yml`) with:

- releases tagged and makers pinning a tag (`@v1`);
- two-person review for every change (it's trusted by every app built with it);
- the API configured to match: `SLOP_BUILD_WORKFLOW=DocFoxHound/slop-build/.github/workflows/build.yml`
  and `SLOP_BUILD_VERSION=v1`.

## Checking it

GitHub Actions can't run locally. Lint the recipe with
[actionlint](https://github.com/rhysd/actionlint):

```sh
actionlint .github/workflows/build.yml
```

The source of truth is the `slop-build/` folder in the Slop repository, where the workflow Slop
generates for makers is tested against these inputs. Change it there, then copy it here and move
the `v1` tag (or tag `v2` for a breaking change).
