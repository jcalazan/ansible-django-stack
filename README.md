ansible-django-stack
====================

Ansible Playbook designed for environments running a Django app.  It can install and configure these applications that are commonly used in production Django deployments:

- Nginx
- Gunicorn
- PostgreSQL
- Supervisor
- Virtualenv
- Memcached
- Celery
- RabbitMQ

Default settings are stored in ```roles/role_name/vars/main.yml```.  Environment-specific settings are in the ```env_vars``` directory.

A `certbot` role is also included for automatically generating and renewing trusted SSL certificates with [Let's Encrypt](https://letsencrypt.org/). 

**Tested with OS:** Ubuntu 14.04 LTS x64

**Tested with Cloud Providers:** [Digital Ocean](https://www.digitalocean.com/?refcode=5aa134a379d7), [Amazon](https://aws.amazon.com), [Rackspace](http://www.rackspace.com/)

## Getting Started

A quick way to get started is with Vagrant and VirtualBox.

### Requirements

- [Ansible](http://docs.ansible.com/intro_installation.html)
- [Vagrant](http://www.vagrantup.com/downloads.html)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

The main settings to change here is in the **env_vars/base** file, where you can configure the location of your Git project, the project name, and application name which will be used throughout the Ansible configuration.

Note that the default values in the playbooks assume that your project structure looks something like this:

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

The main things to note are the locations of the `manage.py` and `wsgi.py` files.  If your project's structure is a little different, you may need to change the values in these 2 files:

- `roles/web/tasks/setup_django_app.yml`
- `roles/web/templates/gunicorn_start.j2`

Also, if your app needs additional system packages installed, you can add them in `roles/web/tasks/install_additional_packages.yml`.

I set some default values in the `env_vars` based on my open-source app, [YouTube Audio Downloader](https://github.com/jcalazan/youtube-audio-dl), so all you really have to do is type in this one command in the project root:

```
vagrant up
```

Wait a few minutes for the magic to happen.  Access the app by going to this URL: http://192.168.33.15

Yup, exactly, you just provisioned a completely new server and deployed an entire Django stack in 5 minutes with _two words_ :).

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

## Security

###NOTE: Do not run the Security role without understanding what it does. Improper configuration could lock you out of your machine.

**Security role tasks**

The security module performs several basic server hardening tasks. Inspired by [this blog post](http://www.codelitt.com/blog/my-first-10-minutes-on-a-server-primer-for-securing-ubuntu/):

* Updates apt
* Performs `aptitude safe-upgrade`
* Adds a user specified by the `server_user` variable, found in `roles/base/defaults/main.yml`
* Adds authorized key for the new user
* Installs sudo and adds the new user to sudoers with the password specified by the `server_user_password` variable found in `roles/security/defaults/main.yml`
* Installs and configures various security packages:
 * [Unattended upgrades](https://help.ubuntu.com/lts/serverguide/automatic-updates.html)
 * [Uncomplicated Firewall](https://wiki.ubuntu.com/UncomplicatedFirewall)
 * [Fail2ban](http://www.fail2ban.org/) (NOTE: Fail2ban is disabled by default as it is the most likely to lock you out of your server. Handle with care!)
* Restricts connection to the server to SSH and http(s) ports
* Limits su access to the sudo group
* Disallows password authentication (be careful!)
* Disallows root SSH access (you will only SSH to your machine as your new user and use a password for `sudo` access)
* Restricts SSH access to the new user specified by the `server_user` variable
* Deletes the `root` password

**Security role configuration**

* Change the `server_user` from `root` to something else in `roles/base/defaults/main.yml`
* Change the sudo password in `roles/security/defaults/main.yml`
* Change variables in `./roles/security/vars/` per your desired configuration

**Running the Security role**

* The security role can be run by running `security.yml` via:

```
ansible-playbook -i development security.yml
```

## Running the Ansible Playbook to provision servers

**NOTE:** to enable the Security module you can use the steps above prior to following the steps below.

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

Next, create a playbook for the server type. See [webservers.yml](webservers.yml) for an example.

Run the playbook:

```
ansible-playbook -i development webservers.yml [-K]
```

You can also provision an entire site by combining multiple playbooks.  For example, I created a playbook called `site.yml` that includes both the `webservers.yml` and `dbservers.yml` playbook.

A few notes here:

- The `dbservers.yml` playbook will only provision servers in the `[dbservers]` section of the inventory file.
- The `webservers.yml` playbook will only provision servers in the `[webservers]` section of the inventory file.
- An inventory var called `env` is also set which applies to `all` hosts in the inventory.  This is used in the playbook to determine which `env_var` file to use.
- The `-K` flag is for adding the sudo password you created for a new sudoer in the Security role (if applicable)

You can then provision the entire site with this command:

```
ansible-playbook -i development site.yml [-K]
```

If you're testing with vagrant, you can use this command:

```
ansible-playbook -i vagrant_ansible_inventory_default --private-key=~/.vagrant.d/insecure_private_key vagrant.yml [-K]
```

## Using Ansible for Django Deployments

When doing deployments, you can simply use the `--tags` option to only run those tasks with these tags.

For example, you can add the tag `deploy` to certain tasks that you want to execute as part of your deployment process and then run this command:

```
ansible-playbook -i stage webservers.yml --tags="deploy"
```

This repo already has `deploy` tags specified for tasks that are likely needed to run during deployment in most Django environments.

## Advanced Options

### Using Python 3.5

Python 3.5 is already installed and to use this version in the `virtualenv`, just override the value of the `virtualenv_python_version` variable in [roles/web/defaults/main.yml](roles/web/defaults/main.yml).

### Creating a swap file

By default, the playbook won't create a swap file.  To create/enable swap, simply change the values in [roles/base/vars/main.yml](roles/base/vars/main.yml).

You can also override these values in the main playbook, for example:

```
---

...

  roles:
    - { role: base, create_swap_file: yes, swap_file_size_kb: 1024 }
    - db
    - rabbitmq
    - web
    - celery
```

This will create and mount a 1GB swap.  Note that block size is 1024, so the size of the swap file will be 1024 x `swap_file_size_kb`.

### Automatically generating and renewing Let's Encrypt SSL certificates with the certbot client

A `certbot` role has been added to automatically install the `certbot` client and generate a Let's Encrypt SSL certificate.

**Requirements:**

- A DNS "A" or "CNAME" record must exist for the host to issue the certificate to.
- The `--standalone` option is being used, so port 80 or 443 must not be in use (the playbook will automatically check if Nginx is installed and will stop and start the service automatically).

In `roles/nginx/defaults.main.yml`, you're going to want to override the `nginx_use_letsencrypt` variable and set it to yes/true to reference the Let's Encrypt certificate and key in the Nginx template. 

In `roles/certbot/defaults/main.yml`, you may want to override the `certbot_admin_email` variable.

A cron job to automatically renew the certificate will run daily.  Note that if a certificate is due for renewal (expiring in less than 30 days), Nginx will be stopped before the certificate can be renewed and then started again once renewal is finished.  Otherwise, nothing will happen so it's safe to leave it running daily.

## Useful Links

- [Ansible - Getting Started](http://docs.ansible.com/intro_getting_started.html)
- [Ansible - Best Practices](http://docs.ansible.com/playbooks_best_practices.html)
- [Setting up Django with Nginx, Gunicorn, virtualenv, supervisor and PostgreSQL](http://michal.karzynski.pl/blog/2013/06/09/django-nginx-gunicorn-virtualenv-supervisor/)
- [How to deploy encrypted copies of your SSL keys and other files with Ansible and OpenSSL](http://www.calazan.com/how-to-deploy-encrypted-copies-of-your-ssl-keys-and-other-files-with-ansible-and-openssl/)
