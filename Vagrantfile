# -*- mode: ruby -*-
# vi: set ft=ruby :

# Setup subscription manager using environment variables to login with
# This elimates hard coding passwords in the code
user = ENV['RHSM_USER']
password = ENV['RHSM_PASS']

ENV["LC_ALL"] = "en_US.UTF-8"

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

  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  config.vm.box = "generic/rhel7"
  config.hostmanager.enabled = true
  config.hostmanager.include_offline = true

  config.vm.provider :virtualbox do |v|
    v.memory = 4096
    v.cpus = 2
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
  end

  config.vm.define "controller" do |controller|
    controller.vm.box = "roboxes/alma8"
    controller.vm.hostname = "controller.localdev"
    controller.vm.network "private_network", ip: "192.168.56.10"
      #virtualbox__intnet: "ClientNetwork"
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
    server1.vm.network "private_network", ip: "192.168.56.45"
    #server1.vm.network "forwarded_port", guest: 3306, host: 3306
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

  config.vm.define "dbserver1" do |dbserver1|
    dbserver1.vm.hostname = "dbserver1.localdev"
    dbserver1.vm.network "private_network", ip: "192.168.56.47"
    dbserver1.vm.network "forwarded_port", guest: 3306, host: 3306
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


  # config.vm.provision "shell", privileged: false, inline: <<-SHELL
  #   echo 'Installing required packages...'
  #   sudo yum upgrade -y > /dev/null
  #   sudo yum install gcc openssl-devel bzip2-devel libffi-devel make zlib-devel ncurses-devel sqlite-devel readline-devel -y > /dev/null
  #   echo 'Installing git ...'
  #   sudo yum install git -y
  #   echo 'Installing pyenv...'
  #   curl -s https://pyenv.run | bash > /dev/null 2>&1
  #   echo 'export PATH="$HOME/.pyenv/bin:$PATH"' >> ~/.bashrc
  #   echo 'eval "$(pyenv init -)"' >> ~/.bashrc
  #   echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc
  #   source ~/.bashrc
  #   echo 'Installing Python 3.8.0...'
  #   pyenv install 3.8.6 > /dev/null 2>&1
  #   pyenv global 3.8.6 > /dev/null 2>&1
  #   echo 'Installing bpython - a fancy interface to the Python interpreter...'
  #   pip install --upgrade pip > /dev/null
  #   pip install bpython > /dev/null 2>&1
  # SHELL

