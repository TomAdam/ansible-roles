# AWS CLI with config

Installs the AWS CLI and configures it with the given credentials. Uses the EC ecgalaxy.aws-cli role. To install this 
role add the following to the roles section of the project requirements.yml:

```yaml
roles:
  - name: ecgalaxy.aws_cli
    version: <current version>
```

See https://galaxy.ansible.com/ecgalaxy/aws_cli for the current version.
