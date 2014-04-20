ansible-django-stack
====================

Ansible Playbook designed for environments running a Django app.  It installs and configures these applications that are commonly used in production Django deployments:
- Nginx
- Gunicorn
- PostgreSQL
- Supervisor
- Virtualenv

## Getting Started
A quick way to get started is with Vagrant and VirtualBox.

### Requirements
- [Ansible](http://docs.ansible.com/intro_installation.html)
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
```ansible-playbook -i 162.243.85.40, -v base.yml --ask-sudo-pass```

If you have multiple servers, create an inventory file in the **inventory/** folder for your environment and add your servers' hostnames or IP addresses there.

For example:
```
# inventory/development

[webservers]
webserver1.example.com
webserver2.example.com

[dbservers]
dbserver1.example.com
```

Run the Playbook:
```
ansible-playbook -i inventory/development -v development.yml --ask-sudo-pass
```
