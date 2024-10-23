Node Version Manager Role
=========================

Installs NVM on a user by user basis. Can co-exist with a system Node version. 

Config
------

```yaml
nvm_config:
- user: vagrant # username to install NVM for
  default_version: lts # any version NVM understands to set as default
  versions: # array of versions to install for user
  - version: 10.7 # any version NVM understands
    global_packages: # array of global packages specific to user and version 
    - gulp
  - version: lts
    global_packages:
    - grunt
```

System Node
-----------

`default_version` can be set to `system` to default to the system installed Node version. Don't
try to use `system` as a version under the `versions` key as any global packages installed will
be attempted using `user` but need root permissions, resulting in a failure.


Updating
--------

When updating the NVM version (see defaults/main.yml) you need to all update the checksum of the
archive.
