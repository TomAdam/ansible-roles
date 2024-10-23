The add-key task allows installation of a repository key without running an APT update. This is used when a repo key
is changed and the APT update in the base role is failing. Add a pre-task to the provision playbook as shown below:

```yaml
pre_tasks:
- import_tasks: tasks/add-key.yml
  vars:
    key_path: roles/chrome/files/google-repo.asc
```

The base directory of the key_path is the ansible folder. 
