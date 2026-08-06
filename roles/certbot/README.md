# certbot

Installs certbot and requests Let's Encrypt certificates in one of four modes:
`nginx`, `apache`, `dns-digitalocean` or `dns-cloudflare`.

## Requirements

- `nginx` mode should run **before** nginx is installed, as nginx fails to start
  when configured certificates are missing.
- `apache` mode expects apache2 to be installed already (e.g. via the `apache2` role).
- `dns-digitalocean` mode needs a DigitalOcean API token with all `domain` scopes,
  generated at https://cloud.digitalocean.com/account/api/tokens/new.
- `dns-cloudflare` mode needs a Cloudflare API token with Zone → DNS → Edit
  permission on every zone the certificates cover, created at
  https://dash.cloudflare.com/profile/api-tokens.
- The DNS modes are standalone — they need nothing from a webserver, so the role
  can run before the webserver role, which then installs SSL vhosts against
  certificates that already exist.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `certbot_mode` | Required | Mode: `nginx`, `apache`, `dns-digitalocean` or `dns-cloudflare` |
| `certbot_certs` | `[]` | Certificates to request; see below |
| `certbot_dry_run` | `false` | Pass `--dry-run` to certbot: exercises the request without creating certificates, avoiding rate limits |
| `certbot_apache2_ssl_vhosts` | `apache2_vhosts` if defined, else `[]` | `apache` mode only: SSL vhosts to install once certificates exist; see below |
| `certbot_digitalocean_dns_token` | Required in `dns-digitalocean` mode | DigitalOcean API token with all `domain` scopes |
| `certbot_cloudflare_dns_token` | Required in `dns-cloudflare` mode | Cloudflare API token with Zone → DNS → Edit on the certificates' zones |
| `certbot_digitalocean_dns_propagation_seconds` | `10` | Seconds to wait for DNS propagation before asking the ACME server to verify the record |
| `certbot_cloudflare_dns_propagation_seconds` | `30` | Seconds to wait for DNS propagation before asking the ACME server to verify the record |

## Certificates

Each `certbot_certs` item requests one certificate:

```yaml
certbot_certs:
  - name: example.com                # certificate directory: /etc/letsencrypt/live/<name>/
    admin_email: admin@example.com   # registration and expiry email address
    domains:                         # domains covered by the single certificate
      - example.com
      - test.example.com
    deploy_hook: /path/to/deploy.sh  # optional: script run after issue/renewal
```

See https://certbot.eff.org/docs/using.html#where-are-my-certificates for which keys
and certificates to reference in webserver config.

In the DNS modes, nothing reloads the webserver after a renewal — set
`deploy_hook` (e.g. `systemctl reload apache2`) or renewed certificates are
never served.

## Apache SSL vhosts

`apache` mode installs the vhosts in `certbot_apache2_ssl_vhosts` after the
certificates exist. Templates are resolved relative to the consuming playbook, so they
must live in `templates/apache2/vhosts/` next to it:

```yaml
certbot_apache2_ssl_vhosts:
  - src: example-ssl.conf.j2  # template in templates/apache2/vhosts/
    dest: example-ssl.conf    # filename under /etc/apache2/sites-available/, symlinked into sites-enabled
```
