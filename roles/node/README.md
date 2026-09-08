# node

Installs a single system Node.js from the NodeSource repository, plus npm-based
package managers (Yarn classic, pnpm) and global packages, all version-pinned.
Extra Node versions can be layered on top with the tj/n version manager.

This role absorbed the removed `yarn` and `nvm` roles — see
[docs/migrating-v6.md](../../docs/migrating-v6.md).

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `node_version` | — | **Required.** Major Node.js version to install (e.g. `22`). Switching majors removes the old NodeSource repo and upgrades in place |
| `node_version_manager` | `none` | `none`, `n` or `nvm` — how extra Node versions beyond the system one are provided |
| `node_npm_version` | `""` | Optional exact version to pin the bundled npm itself; empty leaves the bundled npm alone |
| `node_yarn_enabled` | `false` | Install Yarn classic globally via npm |
| `node_yarn_version` | `"1.22.22"` | Exact Yarn classic version |
| `node_pnpm_enabled` | `false` | Install pnpm globally via npm |
| `node_pnpm_version` | `"11.13.1"` | Exact pnpm version |
| `node_global_packages` | `[]` | Global npm packages to install (`name` or `name@version`) |
| `node_npmrc` | `[]` | npmrc entries; each item is `{template, dir, user}` |

Pin exact package-manager versions: `community.general.npm` reinstalls on every
run when given a range or dist-tag. Disabling a manager does not uninstall it.

## Extra Node versions with n (`node_version_manager: n`)

| Variable | Default | Description |
|----------|---------|-------------|
| `node_n_version` | `"10.2.0"` | tj/n release to install |
| `node_n_checksum` | (matching sha256) | sha256 of `bin/n` at that tag; update together with `node_n_version` |
| `node_n_versions` | `[]` | Node versions to cache; items are `{version, global_packages}` |
| `node_n_prune` | `true` | Remove cached versions not listed in `node_n_versions` |

```yaml
node_version_manager: n
node_n_versions:
  - version: 20            # major, or exact X.Y.Z
    global_packages: [gulp]
```

n is installed as a single script (`/usr/local/bin/n`), fetched from the
pinned tag and verified against `node_n_checksum` — no hosted install
scripts. The cache is **never activated**: the system Node stays the bare
`node` on PATH, and cached versions are used explicitly:

```console
$ n --offline run 20 script.js
$ n --offline exec 20 npm ci
```

`--offline` resolves the version against the cache instead of a remote index
lookup. Per-version `global_packages` are installed into that version's cache
folder (they include package managers: `global_packages: [yarn@1.22.22]`).
A major pin caches the latest release of that major once and does not chase
later patch releases; pin an exact `X.Y.Z` to control upgrades. n downloads
official nodejs.org tarballs with curl at converge time.

## Extra Node versions with nvm (`node_version_manager: nvm`)

| Variable | Default | Description |
|----------|---------|-------------|
| `node_nvm_version` | `"0.40.5"` | nvm release; installed per user as a git clone of nvm-sh/nvm at this tag |
| `node_nvm_config` | `[]` | Per-user config; see below |

```yaml
node_version_manager: nvm
node_nvm_config:
  - user: webdev              # existing account to install nvm for
    default_version: 20       # any nvm version, or 'system' for the apt Node
    versions:
      - version: 20           # or 'system' to skip installing
        global_packages: [gulp, yarn@1.22.22]
```

nvm is installed per user via a git clone at the pinned tag, without submodules
(its official manual-install method — no `curl | bash`), with the init lines managed as a
block in `~/.bashrc`. `default_version: system` falls through to the apt
Node when no version is picked. Per-version package managers are just
`global_packages` entries. `nvm install` fetches official nodejs.org
tarballs at converge time. To upgrade nvm, bump `node_nvm_version`.

## Corepack is not used

Corepack support was removed in v6. The Node.js TSC voted in March 2025 to stop
distributing corepack; Node 25+ no longer bundles it, and it downloads
package-manager binaries from the registry at run time. Package managers are
installed explicitly with pinned versions instead, and stale corepack shims
(`yarn`/`pnpm` symlinks into corepack's dist) are removed on converge.

## Yarn berry

`node_yarn_enabled` installs Yarn classic (1.x), which is also the launcher for
Yarn berry (2+) projects: berry lives per-project via `.yarnrc.yml`'s
`yarnPath` / a committed `.yarn/releases`, so there is nothing to install
globally.

## .npmrc contract

`node_npmrc` templates are playbook-relative: an item
`{template: npmrc.j2, dir: /home/deploy, user: deploy}` renders
`templates/node/npmrc.j2` from the playbook directory to
`/home/deploy/.npmrc` (mode `0600`).
