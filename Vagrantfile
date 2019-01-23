# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/xenial64"
  config.ssh.forward_agent = false

  config.vm.define "my-cool-app.local", primary: true do |app|
    app.vm.hostname = "my-cool-app"

    app.vm.network "private_network", type: "dhcp"
  end

  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--name", "MyCoolApp", "--memory", "1024"]
  end

  config.vm.provider "docker" do |d, override|
    override.vm.box = nil

    d.name = "MyCoolApp"
    d.build_dir = "docker"
    d.create_args = ["--publish-all", "--security-opt=seccomp:unconfined",
                     "--tmpfs=/run", "--tmpfs=/run/lock", "--tmpfs=/tmp",
                     "--volume=/sys/fs/cgroup:/sys/fs/cgroup:ro"]
    d.has_ssh = true
  end

  # For local development, uncommenting and editing the line below will enable
  # a folder in the host machine containing your local git repo to be synced to
  # the guest machine. Ensure the Ansible playbook variable "setup_git_repo" is
  # set to "no" (in env_vars/vagrant.yml) when enabling this.
  #config.vm.synced_folder "../../../my-cool-app", "/webapps/mycoolapp/my-cool-app"

  # Ansible provisioner.
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "vagrant.yml"
    ansible.host_key_checking = false
    ansible.verbose = "vv"
  end
end
