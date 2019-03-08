Config
----------------

```yaml
gpg_config:
- user: # unix username
  private_keys:
  - id: # full key id
    key: # base64 encoded armoured key
    passphrase: # key passphrase
  public_keys:
  - id: # full key id
    key: # base 64 encoded armoured key
    owner_trust_level: # owner trust level (see `man gpg2`, 5 for ultimately trusted)
```
