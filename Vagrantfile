# -*- mode: ruby -*-
# vi: set ft=ruby

# VAGRANTFILE_API_VERSION = "2"
Vagrant.require_version ">= 1.5.2"

#Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
Vagrant.configure("2") do |config|
  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.http = "http://proxyserver.fqdn.com:80"
    config.proxy.https = "http://proxyserver.fqdn.com:80"
    config.proxy.no_proxy = "localhost, 127.0.0.1,.example.com,.test.om"
  end

  config.hostmanager.enabled = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  config.vm.define "devopsmain" do |devopsmain|
    # puppetmaster, ansible main, hadoop main, autotest main
    devopsmain.vm.synced_folder ".", "/vagrant"
    devopsmain.vm.synced_folder "code/", "/puppet_code"
    devopsmain.vm.synced_folder "puppetserver/", "/puppet_puppetserver"
    devopsmain.vm.box  = "centos/7"
    devopsmain.vm.hostname = "devopsmain.test.om"
    devopsmain.vm.network :private_network, ip: "192.168.250.100"
    devopsmain.hostmanager.alias = %w(devopsmain)
    devopsmain.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "4096"]
      vb.customize ["modifyvm", :id, "--cpus", "2"]
    end
    devopsmain.vm.provision "shell", inline: <<-SHELL
       sudo yum update -y
       sudo rpm -ivh  http://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
       sudo yum --disableplugin=fastestmirror install puppetserver -y
       cp -pr /etc/puppetlabs/code /puppet_code
       cp -pr /etc/puppetlabs/puppetserver /puppet_puppetserver
       rm -rf /etc/puppetlabs/code
       ln -s /puppet_code /etc/puppetlabs/code
       rm -rf /etc/puppetlabs/puppetserver
       ln -s /puppet_puppetserver /etc/puppetlabs/puppetserver
       sudo sed -i 's/2g/512m/g' /etc/sysconfig/puppetserver
       echo "*.test.om" | sudo tee /etc/puppetlabs/puppet/autosign.conf
       sudo /opt/puppetlabs/bin/puppetserver gem install hiera-eyaml
       sudo cp /vagrant/keys/* /etc/puppetlabs/keys/
       sudo chown puppet:puppet /etc/puppetlabs/keys/*
       sudo service puppetserver start
    SHELL
  end

  config.vm.define "devopsc1" do |devopsc1|
    # Puppet agent, ansible agent and secondary name node, autotest test machine.
    devopsc1.vm.synced_folder ".", "/vagrant"
    devopsc1.vm.box = "centos/7"
    devopsc1.vm.hostname = "devopsc1.test.om"
    devopsc1.vm.network :private_network, ip: "192.168.250.101"
    devopsc1.hostmanager.alias = %w(devopsc1)
    devopsc1.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "2048"]
      vb.customize ["modifyvm", :id, "--cpus", "2"]
    end
    devopsc1.vm.provision "shell", inline: <<-SHELL
       sudo yum update -y
       sudo rpm -ivh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
       sudo yum install puppet-agent -y
       sudo service puppet start
    SHELL
  end

  config.vm.define "devopsc2" do |devopsc2|
    # puppet agents, ansible agent, job tracker, autotest test machin
    devopsc2.vm.synced_folder ".", "/vagrant"
    devopsc2.vm.box = "ubuntu/trusty64"
    devopsc2.vm.hostname = "devopsc2.test.om"
    devopsc2.vm.network :private_network, ip: "192.168.250.102"
    devopsc2.hostmanager.alias = %w(devopsc2)
    devopsc2.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "2048"]
      vb.customize ["modifyvm", :id, "--cpus", "2"]
    end
    devopsc2.vm.provision "shell", inline: <<-SHELL
       wget https://apt.puppetlabs.com/puppetlabs-release-pc1-trusty.deb
       sudo dpkg -i puppetlabs-release-pc1-trusty.deb
       sudo apt-get update
       sudo apt-get install puppet-agent -y
       sudo /opt/puppetlabs/bin/puppet agent --enable
       sudo service puppet start
    SHELL
  end
end
