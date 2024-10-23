Certbot role
============

This role installs certbot and requests certificates for the specified domains. It can be used in two modes: `nginx` 
and `dns-digitalocean`.

Config
------

```yaml
certbot_mode: nginx|dns-digitalocean
certbot_certs: # array of certificates to request
- admin_email: admin@example.com # administration email address
  name: example.com # name of the certificate
  domains: # array of domains in single certificate
  - example.com
  - test.example.com 
certbot_dry_run: false # test mode that prevents creation or download of certs (default: false)
```

Notes
-----

The name key will be used for the dir of the certificate. In the example above the cert will be
stored at `/etc/letsencrypt/live/example.com/cert.pem`.

If using the `mginx` mode, this role should be run before nginx is installed as nginx will fail to start if the certs 
are not present.

If using the `dns-digitalocean` mode, you will need to set the `certbot_digitalocean_dns_token` var to an API token with
all `domain` scopes. This can be generated at https://cloud.digitalocean.com/account/api/tokens/new.

See https://certbot.eff.org/docs/using.html#where-are-my-certificates for more details on which keys and certs
should be used in the webserver config.

The `certbot_dry_run` option is useful for testing the role without actually requesting certificates, which avoids rate 
limit issues.
