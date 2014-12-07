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

**Tested with OS:** Ubuntu 12.04.4 LTS x64

**Tested with Cloud Providers:** Amazon, Rackspace, Digital Ocean

## Getting Started
A quick way to get started is with Vagrant and VirtualBox.

### Requirements
- [Ansible (use v1.7.1 as there's currently a bug in v1.7.2, see issues, you can do ```sudo pip install ansible==1.7.1``` if you're using pip)](http://docs.ansible.com/intro_installation.html)
- [Vagrant](http://www.vagrantup.com/downloads.html)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

The main settings to change here is in the **env_vars/base** file, where you can configure the location of your Git project, the project name, and application name which will be used throughout the Ansible configuration.

I set some default values here using my open-source app, [GlucoseTracker](https://github.com/jcalazan/glucose-tracker), so all you really have to do is type in this one command in the project root:
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

## Running the Ansible Playbook on existing servers
Create a Playbook for that environment and specify the user accounts to use. See **development.yml** for an example.

If you only have 1 server, you can skip the other steps below and simply run the Playbook with this command:

```
ansible-playbook -i server_ip_address_or_hostname, -v playbook.yml --ask-sudo-pass
```

If you have multiple servers, create an inventory file in the root folder for your environment and add your servers' hostnames or IP addresses there.

For example:

```
# development

[webservers]
webserver1.example.com
webserver2.example.com

[dbservers]
dbserver1.example.com
```

Run the Playbook:

```
ansible-playbook -i development -v development.yml --ask-sudo-pass
```

If you're testing with vagrant, you can use this command:

```
ansible-playbook -i vagrant_ansible_inventory_default --private-key=~/.vagrant.d/insecure_private_key -v vagrant.yml
```

## Useful Links
- [Ansible - Getting Started](http://docs.ansible.com/intro_getting_started.html)
- [Ansible - Best Practices](http://docs.ansible.com/playbooks_best_practices.html)
- [Setting up Django with Nginx, Gunicorn, virtualenv, supervisor and PostgreSQL](http://michal.karzynski.pl/blog/2013/06/09/django-nginx-gunicorn-virtualenv-supervisor/)
- [How to deploy encrypted copies of your SSL keys and other files with Ansible and OpenSSL](http://www.calazan.com/how-to-deploy-encrypted-copies-of-your-ssl-keys-and-other-files-with-ansible-and-openssl/)
