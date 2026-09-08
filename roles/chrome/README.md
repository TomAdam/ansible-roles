# chrome

Installs Google Chrome and optionally ChromeDriver using Puppeteer's browser installer.

## Requirements

Node.js must already be installed (e.g. via the `node` role) — the role fails
fast if `npx` is not available. The role deliberately does not install Node.js
itself, since the Node version is site-specific.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `chrome_version` | `stable` | Release channel (`stable`, `beta`, `dev`, `canary`), major milestone (e.g. `127`) or exact version to install |
| `chrome_chromedriver_version` | `{{ chrome_version }}` | ChromeDriver channel, major milestone or exact version to install |
| `chrome_install_chromedriver` | `false` | Whether to install ChromeDriver |
| `chrome_remove_system_chrome` | `false` | Remove system-installed Chrome first |
| `chrome_system_dependencies` | See defaults | System dependencies for Chrome (packages renamed per Debian release are appended automatically) |
