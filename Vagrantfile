# -*- mode: ruby -*-
# vi: set ft=ruby :

# Setup subscription manager using environment variables to login with
# This elimates hard coding passwords in the code
user = ENV['RHSM_USER']
password = ENV['RHSM_PASS']

ansible_mode = "2.0"

if !user or !password
  puts 'Required environment variables not found. Please set RHSM_USER and RHSM_PASS'
  abort
end

register_script = %{
if ! subscription-manager status; then
  sudo subscription-manager register --username=#{user} --password=#{password} --auto-attach
fi
}

unregister_script = %{
if subscription-manager status; then
  sudo subscription-manager unregister
fi
}

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|

  ENV["LC_ALL"] = "en_US.UTF-8"

  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  config.vm.box = "generic/rhel7"
  config.hostmanager.enabled = true
  config.hostmanager.include_offline = true
  config.hostmanager.manager_guest = true

  config.vm.provider :virtualbox do |v|
    v.memory = 4096
    v.cpus = 2
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
  end

  config.vm.define "controller" do |controller|
    controller.vm.box = "roboxes/alma8"
    controller.vm.hostname = "controller.localdev"
    controller.vm.network "private_network", ip: "192.168.56.10",
      virtualbox__intnet: "TowerNetwork"
    controller.vm.network "forwarded_port", id: "ssh", guest: 22, host: 2235
    controller.hostmanager.aliases = %w(controller)

    controller.vm.synced_folder "provision/", "/vagrant/provision"
      controller.vm.provision "ansible_local" do |ansible|
        ansible.compatibility_mode = (ansible_mode)
        ansible.playbook = "provision/controller.yml"
        ansible.install_mode ="pip3"
        ansible.verbose = true
    end
  end

  config.vm.define "server1" do |server1|
    server1.vm.hostname = "server1.localdev"
    server1.vm.network "private_network", ip: "192.168.56.45",
      virtualbox__intnet: "TowerNetwork"
    server1.vm.network "forwarded_port", guest: 443, host: 8443
    server1.vm.provision "shell", inline: register_script
    server1.hostmanager.aliases = %w(server1)

    server1.vm.synced_folder "provision/", "/vagrant/provision"
      server1.vm.provision "ansible_local" do |ansible|
        ansible.compatibility_mode = (ansible_mode)
        ansible.playbook = "provision/clients.yml"
        ansible.verbose = true
    end

    server1.trigger.before :destroy do |trigger|
      trigger.info = "Unregistering from Red Hat subscription"
      trigger.run_remote = {inline: unregister_script}
    end
  end

  # Create servers 2-4 in a loop
  (2..4).each do |i|
    config.vm.define "server#{i}" do |node|
      node.vm.hostname = "server#{i}.localdev"
      node.vm.network "private_network", ip: "192.168.56.#{i}",
        virtualbox__intnet: "TowerNetwork"
      node.vm.provision "shell", inline: register_script
      node.hostmanager.aliases = "server#{i}"

      node.vm.synced_folder "provision/", "/vagrant/provision"
        node.vm.provision "ansible_local" do |ansible|
          ansible.compatibility_mode = (ansible_mode)
          ansible.playbook = "provision/clients.yml"
          ansible.verbose = true
      end

      node.trigger.before :destroy do |trigger|
        trigger.info = "Unregistering from Red Hat subscription"
        trigger.run_remote = {inline: unregister_script}
      end
    end
  end

  config.vm.define "dbserver1" do |dbserver1|
    dbserver1.vm.hostname = "dbserver1.localdev"
    dbserver1.vm.network "private_network", ip: "192.168.56.47",
      virtualbox__intnet: "TowerNetwork"
    dbserver1.vm.network "forwarded_port", guest: 5432, host: 5432
    dbserver1.vm.provision "shell", inline: register_script
    dbserver1.hostmanager.aliases = %w(dbserver1)

    dbserver1.vm.synced_folder "provision/", "/vagrant/provision"
    dbserver1.vm.provision "ansible_local" do |ansible|
      ansible.compatibility_mode = (ansible_mode)
      ansible.playbook = "provision/clients.yml"
      ansible.verbose = true
    end

    dbserver1.trigger.before :destroy do |trigger|
      trigger.info = "Unregistering from Red Hat subscription"
      trigger.run_remote = {inline: unregister_script}
    end
  end
end
