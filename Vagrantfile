# -*- mode: ruby -*-
# vi: set ft=ruby :

# load configs
require 'yaml'
current_dir    = File.dirname(File.expand_path(__FILE__))
configs        = YAML.load_file("#{current_dir}/config.yaml")
vagrant_config = configs['configs']

Vagrant.configure(2) do |config|

  config.vagrant.plugins = ["vagrant-vbguest", "vagrant-hostmanager"]

  # activate hostmanager plugin
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = true

  config.vm.define "box" do |b|

    b.vm.box = "ubuntu/jammy64" # 22.04

    b.vm.provider "virtualbox" do |vb|
      vb.memory = vagrant_config['memory']
      vb.cpus = vagrant_config['cpus']
      vb.customize ["modifyvm", :id, "--audio", "none"]
    end

    # vagrant-hostmanager is necessary to update /etc/hosts on hosts and guests
    b.vm.network "private_network", ip: vagrant_config['ip']
    b.vm.hostname = vagrant_config['domain']
    b.hostmanager.aliases = vagrant_config['aliases']

    b.vm.synced_folder ".", "/vagrant", disabled: true
    b.vm.synced_folder "ansible_vagrant", "/vagrant/ansible_vagrant", create: true, owner: "vagrant", group: "vagrant", mount_options: ["dmode=775,fmode=775"]
    b.vm.synced_folder "custom-odoo-addons", "/opt/custom-odoo-addons", create: true, owner: "vagrant", group: "vagrant", mount_options: ["dmode=777,fmode=777"]

    # auto update guest additions
    b.vbguest.auto_update = true

    # Run Ansible from the Vagrant VM
    b.vm.provision "shell", inline: "apt update && apt install python3-pip tree -yqq"

    if vagrant_config['ansible_version'] == "latest"
      b.vm.provision "shell", inline: "pip install --upgrade --no-warn-script-location ansible-core"
    else
      # if you need to test with a specific version
      # e.g. Ansible Version 2.11 is important for kubespray
      # b.vm.provision "shell", inline: "pip install --upgrade ansible-core~=2.11.0"
      b.vm.provision "shell", inline: "pip install --upgrade --no-warn-script-location ansible-core~=#{vagrant_config['ansible_version']}"
    end

    b.vm.provision "ansible_local" do |ansible|
      ansible.install = false
      ansible.playbook = "ansible_vagrant/playbook.yml"
      ansible.galaxy_role_file = "ansible_vagrant/requirements.yml"
      #Uncomment when ansible 2.10 is available
      #ansible.galaxy_command = "sudo ansible-galaxy install -r %{role_file} --force; sudo ansible-galaxy collection install -r %{role_file} --force"
      ansible.extra_vars = {
        ansible_python_interpreter:"/usr/bin/python3"
      }
    end
  end
end
