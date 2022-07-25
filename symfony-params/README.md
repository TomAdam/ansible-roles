To set the `vagrant_host_os` var you need to add the following to the Vagrantfile:

```ruby
    if Vagrant::Util::Platform.darwin?
        os = 'darwin'
    else
        os = 'linux'
    end

    ansible.extra_vars = {
        vagrant_host_os: os
    }
```
