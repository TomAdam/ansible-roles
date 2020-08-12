Certbot role
============

This role should be run before the Nginx role.


Config
------

```yaml
certbot_certs: # array of certificates to request
- admin_email: admin@example.com # administration email address
  domains: # array of domains in single certificate
  - example.com
  - test.example.com 

certbot_dry_run: false # test mode that prevents creation or download of certs (default: false)
```

Notes
-----

The first domain in the array will form the name/dir of the certificate. In the example above the cert will be
stored at `/etc/letsencrypt/live/example.com/cert.pem`.

See https://certbot.eff.org/docs/using.html#where-are-my-certificates for more details on which keys and certs
should be used in the Nginx vhost config.
