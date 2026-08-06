# Changelog

All notable changes to the `tborealis.roles` collection are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this collection adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Each entry is prefixed with the affected role, e.g. `**nginx:**`. Changes that affect
the whole repository (CI, docs, tooling) use the `**meta:**` scope. Breaking changes are
flagged with **BREAKING** and require a MAJOR version bump.

## [Unreleased]

## [6.2.0] - 2026-08-06

### Added

- **certbot:** per-plugin DNS propagation wait, passed as
  `--dns-<plugin>-propagation-seconds` on certificate creation:
  `certbot_digitalocean_dns_propagation_seconds` (default `10`, matching the
  upstream plugin default) and `certbot_cloudflare_dns_propagation_seconds`
  (default `30`, raised from the upstream 10s which is too short for
  Cloudflare). Existing certificates are updated too, so the value also
  applies to renewals of already-issued certificates — via `certbot
  reconfigure` on trixie, or by editing the renewal config directly on
  bookworm (whose certbot predates `reconfigure`).

## [6.1.0] - 2026-08-05

### Added

- **certbot:** new `dns-cloudflare` mode — DNS-01 validation via a Cloudflare
  API token (`certbot_cloudflare_dns_token`, scoped to Zone → DNS → Edit on
  the certificates' zones). Standalone like `dns-digitalocean`: it only issues
  certificates — order the role before the webserver role and use `deploy_hook`
  to reload the webserver on renewal.

## [6.0.0] - 2026-07-20

### Added

- **meta:** custom ansible-lint rule `loop-var-in-name`
  (`.config/ansible-lint-rules/`) failing the lint gate when a task's name
  references its own loop variable — names are templated before the loop, so
  such names always emit an "'item' is undefined" template warning at run
  time, which molecule never fails on.
- **node:** optional tj/n version manager (`node_version_manager: n`): extra
  Node versions cached machine-wide from official nodejs.org builds
  (`node_n_versions`, with per-version global packages and pruning of
  unlisted versions), used via `n --offline run`/`n --offline exec` while
  the system Node stays the default. n itself is a single script pinned by
  tag and sha256 — no hosted install scripts.
- **php:** new consolidated role replacing `php_repo_sury`, `php_cli`,
  `php_fpm`, `composer` and `new_relic`: one required `php_version` (8.1–8.5,
  validated), feature switches (`php_fpm`/`php_composer` default on,
  `php_new_relic` default off; CLI always installed) and extensions by name
  (`php_extensions: [curl]`) that follow version bumps automatically. Config
  is applied as `conf.d/99-ansible.ini` drop-ins over the untouched stock
  `php.ini`, so upstream config changes flow through package upgrades.
  Changing `php_version` stops, purges and de-configures superseded versions
  (`php_remove_other_versions`), and hosts converged by the old roles get the
  stock `php.ini` restored once; both paths are exercised by a dedicated
  molecule `upgrade` scenario. Also new: sessions moved off the `/tmp`
  fallback to `/var/lib/php/sessions` with strict-mode/httponly hardening and
  optional per-pool session directories, and opcache tuning that was
  previously unavailable (preloading, tracing JIT, interned strings buffer,
  secured file cache). Migration guide: `docs/migrating-v6.md`.
- **phpbu:** an uninstall entrypoint (`import_role: … tasks_from: remove`)
  removing the cron entry, phar, config, data/log directories and the `phpbu`
  user, with its own molecule scenario proving removal converges and is
  idempotent. (#127)
- **env_file:** new role replacing `dotenv` and `environment`: one
  `env_file_list` whose items render `NAME=value` files (mapping `vars`,
  names verbatim, values shell-quoted, `ansible_managed` header) with
  `group` defaulting to the owner and `mode` to `0644`; the file-level
  `export: true` flag renders `export NAME=value` profile scripts instead.
  See `docs/migrating-v6.md`. (#136)
- **aws:** new consolidated role replacing `aws_cli` and `aws_config`:
  `aws_cli_install` (default true) gates the CLI, Session Manager plugin and
  bash completion install, so config-only (instance-profile/SSO) hosts work
  with `aws_cli_install: false`. The Session Manager plugin now defaults to
  off (`aws_cli_ssm_install: false`). Per-user config uses `aws_config`'s
  multi-profile shape, generalised: every profile key besides the credential
  pair renders verbatim into `~/.aws/config` (`region`, `output`, `role_arn`,
  `source_profile`, `sso_*`, …) and credentials entries are emitted only for
  profiles carrying `access_key_id`, with the paired key asserted.
  Prerequisites are scoped to the paths that need them (unzip/gnupg to the
  CLI install, acl to the per-user config). Migration guide:
  `docs/migrating-v6.md`.

### Changed

- **certbot:** **BREAKING** — `apache2_ssl_vhosts` is renamed to
  `certbot_apache2_ssl_vhosts`, following the role-prefix naming convention,
  and now defaults to `apache2_vhosts` (falling back to `[]`), so apache mode
  reuses the apache2 role's vhost list unless overridden.
  See `docs/migrating-v6.md`.
- **chrome:** the internal chromedriver registers/facts gain the `chrome_`
  prefix (no user-facing variables changed).
- **stripe_cli:** the CLI is installed from Stripe's GPG-signed apt repository
  (`packages.stripe.dev`, key managed by `apt_keys`) instead of an unverified
  `.deb` downloaded from GitHub releases. Version pinning via
  `stripe_cli_version` is unchanged (the repository retains every release).
- **redis:** the `Restart Redis` handler is renamed `Restart redis`, matching
  the lowercase handler naming used by every other role.
- **meta:** ansible-lint moves from the `shared` profile to the strictest
  `production` profile, and `var-naming[no-role-prefix]` is no longer skipped,
  so role variables must carry the role-name prefix.
- **phpbu:** molecule prepare uses the new `php` role instead of
  `php_repo_sury` plus manual package installs.
- **phpbu:** fail fast with a clear error if PHP is not installed, and
  install the cron package for the crontab entry.
- **phpbu:** the README documents the role variables and the playbook-relative
  `templates/phpbu/phpbu.xml` contract, and points at GitHub for releases
  (phar.phpbu.de is dead). (#127, #137)
- **phpbu:** **BREAKING** — the phar is upgraded to 6.0.33 and its GPG
  signature is now verified against the signing key shipped with the role
  (`GOODSIG` required, so an expired or wrong key fails loudly — same
  mechanism as aws_cli); upstream recommends signature-verified installs and
  has disabled `--self-update`. The install is pinned by a single
  `phpbu_version` variable, replacing `phpbu_phar_url`,
  `phpbu_phar_checksum` and `phpbu_schema_url`. See `docs/migrating-v6.md`.
  (#127)
- **chrome:** fail fast with a clear error if Node.js (npx) is not installed.
- **symfony_params:** the README documents the role variables and the
  playbook-relative `templates/symfony-params/` contract. (#137)
- **meta:** `make test` and CI run every molecule scenario a role ships
  (`molecule test --all`); `make test` accepts `SCENARIO=<name>`.
- **meta:** all v6.0.0 breaking changes are collected in one migration guide,
  `docs/migrating-v6.md`, which absorbs `docs/migrating-php-roles.md`.
- **pgsql:** **BREAKING** — consolidated with `pgsql_client` behind a
  `pgsql_mode` switch (`server`, the default, or `client`), and all
  `postgres_*` variables renamed to `pgsql_*`. The colliding
  `postgres_default_packages` (server+client packages in `pgsql`, client
  packages in `pgsql_client`) is split into `pgsql_server_packages` and
  `pgsql_client_packages`; `pgsql_locale` is decoupled from base's
  `system_default_locale` (same `en_GB.UTF-8` effective default); the dead
  `Restart postgres` handler and the unreachable
  `remove-postgres.yml`/`remove-postgres-client.yml` task files are removed.
  See `docs/migrating-v6.md`. (#128)
- **pgsql:** **BREAKING** — server mode drops PostgreSQL 12/13 and supports
  15–18 (16 newly accepted); the per-version config templates — identical
  for 15/17/18 — are unified into one `postgresql.conf.j2`/`pg_hba.conf.j2`.
  New tunables: `pgsql_work_mem` (4MB), `pgsql_maintenance_work_mem` (256MB,
  sized for the new baseline), `pgsql_random_page_cost`,
  `pgsql_wal_compression`, `pgsql_timezone` (replaces the hardcoded
  Europe/London `timezone`/`log_timezone`, same default) and
  `pgsql_io_method` (rendered on 18+ only, stock `worker`). Changed
  defaults, deliberately fleet-tuned for SSD/NVMe hosts: the memory baseline
  moves from 2GB to 4GB RAM (`pgsql_shared_buffers` 512MB → 1GB,
  `pgsql_effective_cache_size` 1GB → 2GB), `pgsql_random_page_cost` is 1.1
  (stock 4.0 models spinning disks — query plans can shift, generally for
  the better), and `pgsql_wal_compression` is `lz4` (stock off; compresses
  WAL full-page images, safe on existing clusters as it only affects newly
  written WAL). The README gains a per-variable tuning guide. (#128)
- **base:** **BREAKING** — the remaining unprefixed variables are renamed
  with a `base_` prefix (`system_locales`, `system_default_locale`,
  `system_timezone`, `system_packages`, `dns_primary_nameserver`,
  `dns_secondary_nameserver`, `default_users`, `users`, `local_hostnames`,
  `known_hosts`, `ssh_extra_user_groups`, `ssh_allow_tcp_forwarding` →
  `base_<same>`). Rename table in `docs/migrating-v6.md`. (#139)
- **exim4:** **BREAKING** — `mailname` is renamed to `exim4_mailname`. See
  `docs/migrating-v6.md`. (#139)

### Removed

- **mysql:** **BREAKING** — the deprecated `mysql_innodb_log_file_size` and
  `mysql_innodb_log_files_in_group` variables are removed and now silently
  ignored; set `mysql_innodb_redo_log_capacity` (size × group) instead. See
  `docs/migrating-v6.md`.
- **rabbitmq:** the unused `Restart rabbitmq` handler — nothing in the role
  notifies it and no task writes config that would need a restart.
- **tasks:** **BREAKING** — the shared task-file pseudo-role (`roles/tasks/`)
  is removed. `add-key.yml` was superseded by the `apt_keys` role, and
  `secure-vars.yml` had been non-functional since the FQCN conversion (#87)
  dropped the `path` argument from its `stat` task. See `docs/migrating-v6.md`.
- **apt_keys:** **BREAKING** — the yarn keyring is removed from the manifest
  (the yarn apt repo is no longer used); the `node` role deletes
  `/etc/apt/keyrings/yarn-repo.gpg` from hosts on converge.
- **yarn:** **BREAKING** — the role is removed; the `node` role installs the
  pinned Yarn classic via npm (`node_yarn_enabled`) and removes the frozen
  dl.yarnpkg.com apt repo, its keyring and the apt package on converge. See
  `docs/migrating-v6.md`.
- **nvm:** **BREAKING** — the role is removed, absorbed by `node`
  (`node_version_manager: nvm`, `nvm_config` → `node_nvm_config`,
  `nvm_version` → `node_nvm_version`). nvm is no longer installed by piping
  the hosted install script to bash: each user's `~/.nvm` is a git clone at
  the pinned tag and the `.bashrc` init lines are a role-managed block. See
  `docs/migrating-v6.md`.
- **node:** **BREAKING** — corepack support (`node_corepack_enable`) is
  removed: the Node.js TSC voted to stop distributing corepack (gone from
  Node 25+) and it downloads package-manager binaries at run time. Package
  managers are now explicit pinned npm installs (`node_yarn_*`,
  `node_pnpm_*`, `node_npm_version`) and stale corepack shims are cleaned up
  on converge. Switching `node_version` now removes NodeSource repos for
  other majors. See `docs/migrating-v6.md`.
- **php_repo_sury, php_cli, php_fpm, composer, new_relic:** **BREAKING**
  removed, replaced by the consolidated `php` role. Variables are renamed or
  collapsed (`php_` prefix throughout, e.g. `newrelic_key` →
  `php_newrelic_key`, `composer_global_packages` →
  `php_composer_global_packages`, per-SAPI duplicates merged, `php_packages`
  → `php_extensions` without the `phpX.Y-` prefix), PHP 7.4/8.0 support is
  dropped, `php.ini`/`php-fpm.conf` are no longer templated, the New Relic
  agent is no longer installed unconditionally, and `php_repo_sury`'s
  `remove-php.yml` entrypoint is gone. See `docs/migrating-v6.md`.
- **mailhog:** **BREAKING** — the role is removed; use `mailpit` instead. Converging
  `mailpit` removes a legacy MailHog install (stock paths) before starting Mailpit,
  since both bind SMTP 1025 and HTTP 8025. See `docs/migrating-v6.md`. (#130)
- **apache2:** **BREAKING** — dead `apache2_load_modules` and `apache2_conf`
  variables removed; they were used by no task, so nothing configured through
  them ever took effect. See `docs/migrating-v6.md`. (#138)
- **base:** **BREAKING** — dead `ssh_extra_conf_files` variable removed (used
  by no task) along with the unreachable `fix-prompt.yml` task file. See
  `docs/migrating-v6.md`. (#138)
- **dotenv, environment:** **BREAKING** — both roles are removed, replaced by
  the new `env_file` role. See `docs/migrating-v6.md`. (#136)
- **pgsql_client:** **BREAKING** — the role is removed, merged into `pgsql`
  as `pgsql_mode: client`. See `docs/migrating-v6.md`. (#128)
- **aws_cli, aws_config:** **BREAKING** — both roles are removed, replaced by
  the consolidated `aws` role. The flat `aws_cli_config` shape is gone — use
  `aws_config`'s multi-profile shape (`default_region` → per-profile
  `region`); the `aws_cli_*` install variables keep their names. See
  `docs/migrating-v6.md`.
- **rrsync:** **BREAKING** — the role is removed. rsync ships
  `/usr/bin/rrsync` since bookworm, so the role was reduced to installing
  rsync plus a redundant `/usr/local/bin/rrsync` symlink; `dbcd` installs
  rsync itself and resolves `rrsync` via PATH. See `docs/migrating-v6.md`.
  (#135)

### Fixed

- **base:** the "Restart cron" handler (notified on timezone changes) now
  restarts cron only when it is installed, instead of failing on images that
  ship without cron (e.g. Debian 13 DigitalOcean droplets); base does not
  install cron itself.
- **node, php:** looped task names no longer reference the loop variable
  (node's n-version assert; php's remove-other-versions tasks), which emitted
  an "'item' is undefined" template-error warning on every run.
- **certbot, dbcd, gpg:** READMEs rewritten to the standard variables-table
  format. dbcd's fixes the wrong variable name it documented (`dbcd_mode` →
  `dbcd_modes`), documents `dbcd_server_user`/`dbcd_data_dir`, and corrects the
  consumer-mode requirements (`dbcd_consumer_user` + `dbcd_known_hosts`, not
  `dbcd_client_user`).
- **base, git, supervisor:** READMEs document the previously missing
  `base_use_backports`, `base_config_dhclient` and `git_use_backports`
  variables, and the `supervisor_crashmail_*` settings required when
  `supervisor_crashmail_enable` is on.
- **meta:** the README role table no longer lists the long-removed `ant` role,
  and AGENTS.md matches reality (READMEs use a `Variable` column, `meta/main.yml`
  added to the role-structure diagram).
- **phpbu:** `phpbu_cron_time` now defaults to daily at 02:30, so the role
  converges on stock defaults instead of erroring; the phar and XSD are no
  longer re-downloaded on every run (the install is skipped entirely while
  the pinned version is already in place). (#127)
- **cron:** install the cron package instead of assuming the host provides it.
- **sudo:** install the sudo package (which provides the visudo validator)
  instead of assuming the host provides it.
- **ssl:** install openssl and ssl-cert (provides the ssl-cert group) instead
  of assuming the host provides them.
- **pgsql:** install locales (for locale generation) and acl (for
  become_user postgres) instead of relying on the base role.
- **nvm:** install curl (fetches the installer) and acl (for unprivileged
  become) instead of relying on the base role.
- **aws_cli, aws_config:** install acl for the unprivileged per-user config
  tasks instead of relying on the base role (aws_config intentionally still
  does not install the AWS CLI).
- **php:** the composer tasks install curl instead of assuming the host
  provides it.
- **mailpit:** install tar and gzip for the release unarchive instead of
  assuming the image provides them.
- **dbcd:** server mode installs rsync (which ships rrsync since bookworm)
  and creates the ssh-users group instead of relying on the rrsync and base
  roles.
- **aws_cli:** **BREAKING** — replace the wrapped `ecgalaxy.aws_cli` role with a
  first-party install. The external role verified downloads against a GPG key that
  expired in 2023 (gpg still exits 0, so the check was theatre); the role now ships
  AWS's current signing key, requires `GOODSIG` (an expired key fails loudly), pins
  the CLI and Session Manager plugin versions with checksums, supports arm64, and
  installs its own `unzip`/`gnupg` prerequisites. The `ecgalaxy.aws_cli` entry is
  gone from `requirements.yml`, and the external `awscli_*` variables are replaced
  by `aws_cli_*` equivalents (see the role README and `docs/migrating-v6.md`). (#129)

## [5.2.0] - 2026-07-14

### Added

- **pgsql:** support PostgreSQL 17 and 18: config templates for both majors,
  accepted by the `postgres_version` validation and exercised by the molecule
  scenario (bookworm converges 15, trixie 18). The 15 template's hardcoded
  `cluster_name` and `lc_*` values are re-templated on `postgres_version` /
  `postgres_locale`, matching the 13 template (identical rendering on default
  locale). (#141)
- **mailpit:** new role installing [Mailpit](https://github.com/axllent/mailpit), the
  maintained MailHog successor (same default ports: SMTP 1025, web UI/API 8025), with
  a pinned version, per-architecture sha256 checksums (amd64/arm64), a correctly-moded
  systemd unit, and a daemon-reloading restart handler. Replaces the deprecated
  `mailhog` role. (#130)

### Fixed

- **nginx, apache2, supervisor, rabbitmq, exim4:** the roles now explicitly enable
  and start their services instead of relying on Debian's start-on-install policy
  and config-change handlers, so a re-run recovers a stopped or disabled unit. (#132)
- **chrome:** system dependencies now use the t64 package names on trixie instead of
  relying on virtual-package resolution, and the target version is resolved from the
  Chrome for Testing channel data before installing, so the browser is only downloaded
  when the installed version differs (previously `npx @puppeteer/browsers` ran on
  every play, and a stable promotion between runs broke idempotence).
  Same for chromedriver. (#133)
- **mysql:** removed the undefined `{{ item }}` reference from the looped
  root-user-removal task name, fixing the "'item' is undefined" template warning
  emitted on every run.
- **apt_keys:** same fix for two looped task names in the Molecule verify playbook.
- **certbot:** apache mode now works standalone: the role ships the `Restart apache2`
  handler it notifies, defaults `apache2_ssl_vhosts` to `[]`, and documents the
  variable and its playbook-relative `templates/apache2/vhosts/` contract. (#124)
- **exim4:** include the mailname in `dc_other_hostnames` so aliased local mail
  (e.g. postmaster→root, qualified with `/etc/mailname`) stays routeable when the
  mailname differs from the sender hostname. (#126)
- **pgsql:** `postgres_version` is now documented as required and validated
  against the majors the role ships config templates for (12, 13, 15), failing
  early with a clear message instead of a template-not-found error. The
  `postgres_version` requirement is also documented for `pgsql_client`. (#128)
- **nvm, node:** the corepack-enable and nvm global-package tasks reported changed on
  every run; they now probe the existing shims/packages first, so converges with those
  options enabled are idempotent. The nvm installer is bumped to v0.40.5 and the
  README no longer references a nonexistent archive checksum. (#131)

### Changed

- **do_agent:** the tasks file is renamed from `main.yaml` to `main.yml`, matching
  every other role. Internal only — Ansible loads either extension. (#139)
- **nginx, apache2, sudo:** document the playbook-relative template contract —
  the `src` entries of `nginx_conf`/`nginx_snippets`/`nginx_vhosts`,
  `apache2_vhosts`, and `sudo_additional_config` resolve to `templates/<role>/...`
  directories next to the consuming playbook. (#137)
- **yarn:** document that the `node` role must be ordered before `yarn`, because
  the Yarn deb's `nodejs` dependency is otherwise satisfied by Debian's distro
  Node.js rather than the NodeSource build. (#134)

### Deprecated

- **mailhog:** upstream is archived/unmaintained, the binary download is unverified
  and amd64-only. Use the new `mailpit` role instead; `mailhog` will be removed in the
  next major release. (#130)

## [5.1.0] - 2026-07-14

### Added

- **meta:** Molecule test infrastructure (systemd-enabled bookworm/trixie test images
  on GHCR, shared base config, changed-role CI detection with a weekly full matrix,
  `make test ROLE=<name>`) and pilot scenarios for the `mysql` and `git` roles. See
  `docs/testing.md`.
- **mysql:** optional `mysql_root_password_salt` and per-user `salt` (exactly 20
  characters). Without a salt the caching_sha2_password hash cannot be compared, so
  the root-password and user tasks reported changed on every run; providing one makes
  them idempotent.
- **meta:** Molecule scenarios for the web stack roles: `nginx`, `apache2`, `ssl`,
  `certbot`, and `php_repo_sury`.
- **meta:** Molecule scenarios for the PHP stack roles: `php_cli`, `php_fpm`,
  `composer`, `phpbu`, `new_relic`, and `symfony_params`.
- **meta:** Molecule scenarios for the data service roles: `pgsql`, `pgsql_client`,
  `redis`, `rabbitmq` (bookworm only), `supervisor`, and `mailhog`.
- **meta:** Molecule scenarios for the tooling roles: `node`, `nvm`, `yarn`, `chrome`,
  `stripe_cli`, and `gpg`.
- **meta:** Molecule scenarios for the system config roles: `cron`, `sudo`, `dotenv`,
  `environment`, `apt_keys`, `rrsync`, and `exim4`.
- **meta:** Molecule scenarios for the cloud and heavy roles: `base`, `aws_cli`,
  `aws_config`, `do_agent`, and `dbcd` — every role in the collection now has one.
- **base:** `base_hosts_unsafe_writes` (default `false`) writes `/etc/hosts` in place
  for containers, where the bind mount breaks the default atomic rename.

### Fixed

- **do_agent:** refresh the apt cache after adding the repository; the install task's
  `cache_valid_time` treated the pre-repo cache as fresh, so a first install failed
  with `No package matching 'do-agent'`. Same fix the other repo roles received in
  v4.0.2.
- **base:** the default locale is written through the `/etc/default/locale` symlink
  the locales package creates; without `follow` the template replaced the symlink on
  the run after a fresh install, so the role was never idempotent on first provision.
- **aws_cli:** the role now creates `/etc/bash_completion.d` instead of assuming
  another package has installed it.

- **nvm:** version installs and the default alias reported changed on every run; they
  now detect the existing state. The installer's `creates:` guard checked the wrong
  user's home directory (and would have blocked nvm upgrades); the version check
  alone now guards it.
- **stripe_cli:** the version check probed `/usr/local/bin/stripe`, but the package
  installs `/usr/bin/stripe`, so the deb was re-downloaded on every run.
- **chrome:** `unzip` joined `chrome_system_dependencies` — `@puppeteer/browsers`
  cannot extract the chrome archive without it.
- **rrsync:** the symlink now points at `/usr/bin/rrsync`; since bookworm, rsync ships
  the binary there and no longer includes the doc-scripts copy the role linked to, so
  the role failed on every supported release.

- **apache2:** the `apache2_env_vars` default was a list, which broke the role's own
  envvars template (it iterates `.items()`); the default is now `{}`.
- **composer:** `composer_global_packages` entries failed because the composer module
  rejects arguments embedded in the command string; the task now uses the `arguments`
  and `global_command` parameters.
- **phpbu:** the phar and schema are fetched from GitHub; `phar.phpbu.de` fails its
  TLS handshake and `schema.phpbu.de` no longer resolves, so the role could not
  converge. The pinned version and checksum are unchanged.

- **mysql:** probe the root auth plugin with a fixed dummy password (the 1524 plugin
  error precedes password verification) so Ansible's `no_log` censoring can never
  rewrite the error message. Previously, when the real root password value appeared in
  the message text, the plugin name was masked and detection never fired, failing the
  role on the very error it was meant to handle.

## [5.0.0] - 2026-07-13

### Added

- **mysql:** `mysql_login_unix_socket` variable (default `/var/run/mysqld/mysqld.sock`);
  all role logins now connect via the unix socket so they always run as
  `root@localhost`.

### Fixed

- **mysql:** temporarily enable `mysql_native_password` (drop-in config + restart) when
  root still authenticates with it after a 5.x upgrade, migrate managed accounts to
  `caching_sha2_password`, then remove the drop-in and restart. Previously the role
  failed outright on MySQL 8.4, where the plugin is disabled by default. Accounts not
  listed in `mysql_users` remain on the disabled plugin once the drop-in is removed.
- **mysql:** remove legacy `root@'127.0.0.1'` and `root@'::1'` rows left over from 5.x
  installs; their grants break after upgrades and the default TCP login could match one
  of them, failing the role with `SELECT command denied to user 'root'@'localhost'`.
  `root@localhost` is the canonical superuser.

### Changed

- **base:** **BREAKING** — user authorized keys are now exclusive: any key present on
  the server that is not listed in the user's `authorized_keys` is removed.
- **dbcd:** **BREAKING** — server authorized keys are now exclusive: any key on the
  `dbcd_server_user` account not in `dbcd_server_client_public_key` /
  `dbcd_server_consumer_public_keys` is removed.
- **mysql:** the tuning config now sets `innodb_redo_log_capacity` (new variable
  `mysql_innodb_redo_log_capacity`, default `256M` — the same effective capacity as the
  old defaults) instead of the InnoDB parameters deprecated since MySQL 8.0.30. Also
  drop the obsolete `ib_logfile` deletion on fresh installs.

### Deprecated

- **mysql:** `mysql_innodb_log_file_size` and `mysql_innodb_log_files_in_group` — still
  honoured if set (redo capacity is derived as `size × group`, overriding
  `mysql_innodb_redo_log_capacity`), but they will be removed in a future major
  release; switch to `mysql_innodb_redo_log_capacity`.

## [4.0.3] - 2026-07-03

### Fixed

- **pgsql, pgsql_client, nginx, mysql, rabbitmq, yarn, new_relic, php_repo_sury, base:**
  run the post-repo-add apt cache refresh *after* the legacy `.list` cleanup instead of
  before it. On hosts still carrying a stale legacy source, the v4.0.2 refresh ran
  `apt-get update` while the conflicting file was still present, crashing with
  `Conflicting values set for option Signed-By` before the cleanup could remove it.
- **aws_cli:** stop leaking per-user AWS credentials in playbook output. The
  `Per-user config` loop used `with_items`, so Ansible printed each item's full
  dict — including `access_key_id` and `secret_access_key` — in the `included:`
  task line. A `loop_control` label now shows only the user name.

## [4.0.2] - 2026-06-26

### Changed

- **meta:** bump `actions/setup-python` to v6.3.0 (was v6.2.0) and pin it with the full
  `#.#.#` version in the SHA comment for consistency with the other workflow actions.

### Fixed

- **pgsql, pgsql_client, nginx, mysql, redis, rabbitmq, node, yarn, new_relic,
  php_repo_sury, base:** refresh the apt cache immediately after adding a third-party
  repository so freshly added repos are visible to the install step. The
  `deb822_repository` module does not update the cache, and the following install task's
  `cache_valid_time` could treat the cache refreshed seconds earlier (before the repo
  existed) as fresh and skip the update, producing errors such as
  `No package matching 'postgresql-15' is available` on first provision. The new refresh
  runs only when the repository file changes.

## [4.0.1] - 2026-06-24

### Fixed

- **mysql, rabbitmq, node, new_relic, nginx, pgsql, pgsql_client, yarn, php_repo_sury,
  base:** legacy apt-source cleanup now removes stale `.list` files whose repository URL
  is not at the start of the line. The `find`/`contains` filter defaulted to anchored
  matching (`re.match`), so source files left by pre-key-centralization installs were
  never deleted, producing `Conflicting values set for option Signed-By` errors during
  `apt-get update`. Added `read_whole_file: true` to the legacy-source `find` tasks.

## [4.0.0] - 2026-06-15

### Fixed

- **node:** refresh the NodeSource repository signing key; the previous copy used a
  SHA1-bound signature rejected by Debian trixie's apt (disallowed since 2026-02-01). The
  current upstream key verifies on both bookworm and trixie
- **new_relic:** refresh the New Relic repository signing key to the current `2DAD550E`
  key; the previous `548C16BF` key used SHA1-bound signatures rejected by trixie and no
  longer matches the key signing the repository's `Release`

### Added

- **apt_keys:** new role that installs every collection-managed apt repository signing
  key from a single manifest (`apt_keys_keyrings`) to `/etc/apt/keyrings/`; declared as a
  meta dependency by `base` and every repo role so keys are refreshed before any
  `apt-get update` (including `base`'s initial cache update), removing the need for manual
  per-playbook key fixes when a key has changed
- **meta:** scheduled `key-check` workflow and `scripts/check_apt_keys.py` that warn 30
  days before a signing key expires, fail on revoked/expired keys, and verify each key
  still validates its repository on Debian bookworm and trixie (catching upstream
  revocation, rotation, and signature policy failures); alerts post to Slack via a
  `SLACK_WEBHOOK_URL` secret, and `make check-keys` runs the same checks locally

### Changed

- **apt_keys, nginx, mysql, redis, yarn, php_repo_sury, node, rabbitmq, new_relic,
  do_agent, pgsql, pgsql_client:** BREAKING repository signing keys now install to
  `/etc/apt/keyrings/` (was `/usr/share/keyrings/`) and are managed centrally by the new
  `apt_keys` role; each repo role now depends on `apt_keys`
- **rabbitmq:** BREAKING migrate from the retired `www.rabbitmq.com/debian` repository (no
  longer served) to RabbitMQ's official `deb1.rabbitmq.com` repositories (`rabbitmq-erlang`,
  with its Erlang packages pinned, and `rabbitmq-server`), both signed by the Team RabbitMQ
  key; the suite is now the distribution codename

### Removed

- **nginx, mysql, redis, yarn, php_repo_sury:** BREAKING remove the per-role `add-key.yml`
  task files and their `Manage repo key` / `Import key task` tasks; key installation now
  happens in the `apt_keys` dependency
- **nginx:** remove the unused `fix-key.yml` expired-key workaround

## [3.0.0] - 2026-06-12

### Added

- **redis:** add `redis_version` to optionally pin a major or major.minor line (e.g.
  `8` or `7.4`) via an apt preferences file; empty (the default) installs the latest
- **redis:** add `redis_bind`, `redis_maxmemory`, `redis_maxmemory_policy`, and
  `redis_password` variables, written to an Ansible-managed `/etc/redis/redis-ansible.conf`
  that is included from the packaged `redis.conf`; the service is now restarted when
  the configuration changes

### Changed

- **redis:** BREAKING install Redis from the official `packages.redis.io` APT repository
  via `deb822_repository` (with the signing key shipped in the role) instead of the
  distribution's own repository, and explicitly enable and start the service
- **redis:** BREAKING bind to localhost (`127.0.0.1 -::1`) by default

## [2.0.1] - 2026-06-12

### Fixed

- **pgsql:** use the canonical `login_db` parameter for `postgresql_user`,
  `postgresql_schema`, `postgresql_privs`, and `postgresql_ext` tasks, replacing the
  deprecated `db`/`database` aliases (slated for removal in `community.postgresql` 5.0.0)

## [2.0.0] - 2026-06-12

### Added

- **pgsql:** add `postgres_schemas` variable to create schemas, and
  `schema`/`target_roles` keys in `postgres_privs` for schema-scoped and
  default-privilege grants
- **pgsql:** add `postgres_become_user` variable (default `postgres`) to configure the
  system user that database operations run as

### Changed

- **yarn:** update the bundled Yarn APT repository signing key
- **base, do_agent, mysql, new_relic, nginx, node, pgsql, pgsql_client,
  php_repo_sury, yarn:** stop using the deprecated `ansible.builtin.apt_repository` module
  (slated for removal in ansible-core 2.25). Repositories are now configured with
  `deb822_repository`, and these roles install the `python3-debian` package the
  new module requires. Stale `.list` files left by the old module are removed on the next
  run so apt no longer warns about duplicate sources.
- **chrome:** drop the deprecated `ansible.builtin.apt_repository` task that removed the
  Google Chrome repository during system-chrome cleanup.
- **rabbitmq:** replace the deprecated `ansible.builtin.apt_key` and `apt_repository` with
  a `/usr/share/keyrings` key and a `signed-by` `deb822_repository`.
- **tasks:** replace the deprecated `ansible.builtin.apt_key` in the shared `add-key`
  helper with a copy into `/etc/apt/trusted.gpg.d` (the `key_path` interface is unchanged).
- **mysql:** **BREAKING** switch from the deprecated `community.mysql` collection to the
  renamed `ansible.mysql` collection. Consumers of `tborealis.roles` must now install the
  `ansible.mysql` collection.

### Fixed

- **php_cli, php_fpm:** accept an unquoted YAML version (e.g. `php_version: 8.1`
  parsed as a float) in the supported-version check by coercing to a string before
  comparison

## [1.0.0] - 2026-06-11

### Added

- Initial release of the `tborealis.roles` collection: roles for provisioning local and
  production Debian/Ubuntu environments.
- **meta:** SemVer release policy, `CHANGELOG.md`, tag-triggered GitHub Actions release
  workflow, and a per-PR changelog gate.
