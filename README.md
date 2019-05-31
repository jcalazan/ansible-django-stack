
ansible-django-stack
====================

[![Build Status](https://travis-ci.org/jcalazan/ansible-django-stack.svg?branch=master)](https://travis-ci.org/jcalazan/ansible-django-stack)

Ansible Playbook designed for environments running a Django app.
It can install and configure these applications that are commonly used in
production Django deployments:

- Nginx
- Gunicorn
- PostgreSQL
- Supervisor
- Virtualenv
- Memcached
- Celery
- RabbitMQ

Default settings are stored in `roles/role_name/defaults/main.yml`.
Environment-specific settings are in the `env_vars` directory.

A `certbot` role is also included for automatically generating and renewing
trusted SSL certificates with [Let's Encrypt][lets-encrypt].

**Tested with OS:** Ubuntu 18.04 LTS (64-bit), Ubuntu 16.04 LTS (64-bit).

**Tested with Cloud Providers:** [Digital Ocean][digital-ocean], [AWS][aws], [Rackspace][rackspace]

## Getting Started

A quick way to get started is with Vagrant.

### Requirements

- [Ansible][ansible-installation_guide]
- [Vagrant][vagrant-downloads]
- [VirtualBox][virtual-box_downloads] or [Docker][docker-get_started]

It's recommended to use the version of Ansible specified in `requirements.txt`,
although any version greater than Ansible 2.4 will work with this repository.
When choosing an Ansible version, consider:

- Ansible only issues security fixes for the [last three major releases][ansible-release_cycle].
- The included version of `molecule` has requirements on the Ansible version
  (currently, Molecule requires Ansible 2.4 or later)

Ansible has been configured to use Python 3 inside the remote machine when
provisioning it. In Ubuntu 16.04 LTS, compatible Ansible versions are not
in the main package repositories, but can be installed from the Ansible PPA by
running these commands:

```
sudo add-apt-repository ppa:ansible/ansible
sudo apt-get update
```

### Configuring your application

The main settings to change are in the [`env_vars/base.yml`](env_vars/base.yml)
file, where you can configure the location of your Git project, the project
name, and the application name which will be used throughout the Ansible
configuration. I set some default values based on my open-source app, [YouTube
Audio Downloader][youtube-audio-dl].

Note that the default values in the playbooks assume that your project
structure looks something like this:

```
myproject
├── manage.py
├── myapp
│   ├── apps
│   │   └── __init__.py
│   ├── __init__.py
│   ├── settings
│   │   ├── base.py
│   │   ├── __init__.py
│   │   ├── local.py
│   │   └── production.py
│   ├── templates
│   │   ├── 403.html
│   │   ├── 404.html
│   │   ├── 500.html
│   │   └── base.html
│   ├── urls.py
│   └── wsgi.py
├── README.md
└── requirements.txt
```

The main things to note are the locations of the `manage.py` and `wsgi.py`
files. If your project's structure is a little different, you may need to
change the values in these 2 files:

- `roles/web/tasks/setup_django_app.yml`
- `roles/web/templates/gunicorn_start.j2`

Also, if your app needs additional system packages installed, you can add them
in `roles/web/tasks/install_additional_packages.yml`.

### Creating the machine

Type this command from the project root directory:

```
vagrant up
```

(To use Docker instead of VirtualBox, add the flag `--provider=docker` to the
command above. Note that extra configuration may be required first on your host
for Docker to run systemd in a container.)

Wait a few minutes for the magic to happen. Access the app by going to this
URL: [https://my-cool-app.local](https://my-cool-app.local)

Yup, exactly, you just provisioned a completely new server and deployed an
entire Django stack in 5 minutes with _two words_ :).

### Additional vagrant commands

**SSH to the box**

```
vagrant ssh
```

**Re-provision the box to apply the changes you made to the Ansible configuration**

```
vagrant provision
```

**Reboot the box**

```
vagrant reload
```

**Shutdown the box**

```
vagrant halt
```

## Pulling from a private git repository using SSH agent forwarding

If your code is in a private repository you must use an SSH connection along
with a key so ansible can checkout the code. HTTPS connections and SSH 
connections with a username and password do not work because ansible cannot 
deal with interactive logins.

Using SSH agent forwarding we can get the authentication request from the 
repository sent to the machine where the playbook is run. That keeps the 
private key to the repository as safe as possible. To set this up you need to:

- Add a public key to the server hosting your repo. 
- Make sure ssh-agent is running on your local machine
- Add the public key to ssh-agent

[Connecting to GitHub with SSH](https://help.github.com/articles/connecting-to-github-with-ssh/) 
has all the information on key generation, adding keys to the server, setting up ssh-agent and
troubleshooting any problems.

Your server SSH configuration should work out-of-the-box. The "Server-Side 
Configuration Options" section in [SSH Essentials: Working with SSH Servers, Clients, and
Keys](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys)
has good advice on locking down who can access the server over SSH
connections.

### Getting the playbook to use agent forwarding

The first thing you need to do is set the ssh_agent_forwarding flag in 
env_vars/base.yml to `true`:
```
ssh_forward_agent: true
```

This flag is used when configuring sudoers so that any user you become on the
remote server will also use the same socket connection when requesting to
unlock keys.

To enable SSH agent forwarding on the Vagrant box, change the following flag in
VagrantFile and set it to `true`:
```
config.ssh.forward_agent = true
```

When running a playbook to provision a server, you enable SSH agent forwarding
using the `--ssh-extra-args` option on the command line:
```
ansible-playbook --ssh-extra-args=-A -i production site.yml
```

This is a little bit clunky but it does not restrict you from setting other
SSH options if you need to.




## Security

*NOTE: Do not run the Security role without understanding what it does.
Improper configuration could lock you out of your machine.*

**Security role tasks**

The security module performs several basic server hardening tasks. Inspired by
[this blog post][securing-ubuntu]:

- Updates `apt`
- Performs `aptitude safe-upgrade`
- Adds a user specified by the `server_user` variable, found in `roles/base/defaults/main.yml`
- Adds authorized key for the new user
- Installs sudo and adds the new user to sudoers with the password specified by
  the `server_user_password` variable found in `roles/security/defaults/main.yml`
- Installs and configures various security packages:
     - [Unattended upgrades](https://help.ubuntu.com/lts/serverguide/automatic-updates.html)
     - [Uncomplicated Firewall](https://wiki.ubuntu.com/UncomplicatedFirewall)
     - [Fail2ban](http://www.fail2ban.org/) (NOTE: Fail2ban is disabled by default
       as it is the most likely to lock you out of your server. Handle with care!)
- Restricts connection to the server to SSH and http(s) ports
- Limits `su` access to the sudo group
- Disallows password authentication (be careful!)
- Disallows root SSH access (you will only SSH to your machine as your new user
  and use a password for `sudo` access)
- Restricts SSH access to the new user specified by the `server_user` variable
- Deletes the `root` password

**Security role configuration**

- Change the `server_user` from `root` to something else in `roles/base/defaults/main.yml`
- Change the sudo password in `roles/security/defaults/main.yml`
- Change variables in `./roles/security/vars/` per your desired configuration

**Running the Security role**

- The security role can be run by running `security.yml` via:

```
ansible-playbook -i development security.yml
```

## Running the Ansible Playbook to provision servers

**NOTE:** to enable the Security module you can use the steps above prior to
following the steps below.

Create an inventory file for the environment, for example:

```
# development

[all:vars]
env=dev

[webservers]
webserver1.example.com
webserver2.example.com

[dbservers]
dbserver1.example.com
```

Next, create a playbook for the server type. See
[webservers.yml](webservers.yml) for an example.

Run the playbook:

```
ansible-playbook -i development webservers.yml [-K]
```

You can also provision an entire site by combining multiple playbooks. For
example, I created a playbook called `site.yml` that includes both the
`webservers.yml` and `dbservers.yml` playbook.

A few notes here:

- The `dbservers.yml` playbook will only provision servers in the `[dbservers]`
  section of the inventory file.
- The `webservers.yml` playbook will only provision servers in the
  `[webservers]` section of the inventory file.
- An inventory var called `env` is also set which applies to `all` hosts in the
  inventory. This is used in the playbook to determine which `env_var` file to use.
- The `-K` flag is for adding the sudo password you created for a new sudoer in
  the Security role (if applicable)

You can then provision the entire site with this command:

```
ansible-playbook -i development site.yml [-K]
```

If you're testing with vagrant, you can use this command:

```
ansible-playbook -i vagrant_ansible_inventory_default --private-key=~/.vagrant.d/insecure_private_key vagrant.yml [-K]
```

## Using Ansible for Django Deployments

When doing deployments, you can simply use the `--tags` option to only run
those tasks with these tags.

For example, you can add the tag `deploy` to certain tasks that you want to
execute as part of your deployment process and then run this command:

```
ansible-playbook -i stage webservers.yml --tags="deploy"
```

This repo already has `deploy` tags specified for tasks that are likely needed
to run during deployment in most Django environments.

## Advanced Options

### Changing the Ubuntu release

The [Vagrantfile](Vagrantfile) uses the Ubuntu 18.04 LTS Vagrant box for a
64-bit PC that is published by Canonical in HashiCorp Atlas. To use Ubuntu
16.04 LTS instead, change the `config.vm.box` setting to `ubuntu/xenial64`.

### Changing the Python version used by your application

Python 3 is used by default in the `virtualenv`. To use Python 2 instead, just
override the `virtualenv_python_version` variable and set it to `python`.

It is possible to install other versions of Python from an
[unofficial PPA by Felix Krull (see disclaimer)][deadsnakes].
To use this PPA, override the `enable_deadsnakes_ppa` variable and set it to
`yes`. Then the `virtualenv_python_version` variable can be set to the name of
a Python package from this PPA, such as `python3.6`.

### Changing the Python version used by Ansible

To use Python 2 as the interpreter for Ansible, override the
`ansible_python_interpreter` variable and set it to `/usr/bin/python`. This
allows a machine without Python 3 to be provisioned.

### Creating a swap file

By default, the playbook won't create a swap file. To create/enable swap,
simply change the values in [roles/base/defaults/main.yml](roles/base/defaults/main.yml).

You can also override these values in the main playbook, for example:

```
---

...

  roles:
    - { role: base, create_swap_file: true, swap_file_size_kb: 1024 }
    - db
    - rabbitmq
    - web
    - celery
```

This will create and mount a 1GB swap. Note that block size is 1024, so the
size of the swap file will be 1024 x `swap_file_size_kb`.

### Automatically generating and renewing Let's Encrypt SSL certificates with the certbot client

A `certbot` role has been added to automatically install the `certbot` client
and generate a Let's Encrypt SSL certificate.

**Requirements:**

- A DNS "A" or "CNAME" record must exist for the host to issue the certificate to.
- The `--standalone` option is being used, so port 80 or 443 must not be in use
  (the playbook will automatically check if Nginx is installed and will stop
  and start the service automatically).

In `roles/nginx/defaults.main.yml`, you're going to want to override the
`nginx_use_letsencrypt` variable and set it to yes/true to reference the Let's
Encrypt certificate and key in the Nginx template.

In `roles/certbot/defaults/main.yml`, you may want to override the
`certbot_admin_email` variable.

A cron job to automatically renew the certificate will run daily. Note that if
a certificate is due for renewal (expiring in less than 30 days), Nginx will be
stopped before the certificate can be renewed and then started again once
renewal is finished. Otherwise, nothing will happen so it's safe to leave it
running daily.

## Useful Links

- [Ansible - Getting Started][ansible-installation_guide]
- [Ansible - Best Practices][ansible-best_practices]
- [Setting up Django with Nginx, Gunicorn, virtualenv, supervisor and PostgreSQL][michal-ansible_guide]
- [How to deploy encrypted copies of your SSL keys and other files with Ansible and OpenSSL][deploy-encrypted-copies]
- [Using SSH agent forwarding - GitHub developer guide](https://developer.github.com/guides/using-ssh-agent-forwarding/)

## Contributing

Contributions are welcome! Please make sure any PR passes the TravisCI test suite.

### Running the test suite locally:

The test suite uses a Docker container - make sure Docker is installed and
configured before running the following commands:

```
pip install -r requirements-dev.txt
molecule test
```

[aws]: https://aws.amazon.com
[digital-ocean]: https://www.digitalocean.com/?refcode=5aa134a379d7
[rackspace]: http://www.rackspace.com/

[youtube-audio-dl]: https://github.com/jcalazan/youtube-audio-dl
[ansible-installation_guide]: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
[ansible-best_practices]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html
[ansible-release_cycle]: https://docs.ansible.com/ansible/latest/reference_appendices/release_and_maintenance.html?highlight=release
[docker-get_started]: https://www.docker.com/get-started
[lets-encrypt]: https://letsencrypt.org/
[vagrant-downloads]: https://www.vagrantup.com/downloads.html
[virtual-box_downloads]: https://www.virtualbox.org/wiki/Downloads

[securing-ubuntu]: https://www.codelitt.com/blog/my-first-10-minutes-on-a-server-primer-for-securing-ubuntu/
[deadsnakes]: https://launchpad.net/~fkrull/+archive/ubuntu/deadsnakes/?field.series_filter=xenial
[michal-ansible_guide]: http://michal.karzynski.pl/blog/2013/06/09/django-nginx-gunicorn-virtualenv-supervisor/
[deploy-encrypted-copies]: https://www.calazan.com/how-to-deploy-encrypted-copies-of-your-ssl-keys-and-other-files-with-ansible-and-openssl/
