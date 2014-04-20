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

**Requirements**
- Ansible
- Vagrant
- VirtualBox

The main settings to change here is in the **env_vars/base** file, where you can configure the location of your Git project, the project name, and application name which will be use throughout the Ansible configuration.

I set some default values here for my open-source app, GlucoseTracker, so all you really have to do is type in this one command in the project root:
```vagrant up```

Wait a few minutes for the magic to happen.  Access the app by going to this URL: http://192.168.33.15

Yup, that's all you had to do.  You just provisioned an entire server with one command.

### Additional vagrant commands
**SSH to the box**
```vagrant ssh```

**Re-provision to apply the changes you made to the Ansible configuration**
```vagrant provision```

**Reboot the box**
```vagrant reload```

**Shutdown the box**
```vagrant halt```

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

Run the **ansible-playbook** command:
```
ansible-playbook -i inventory/development -v development.yml --ask-sudo-pass
```