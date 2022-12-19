# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "generic/rocky8"

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = "1024"
    vb.name = "master.puppet.vm"
  end

  config.vm.provision "shell", inline: <<-SHELL
    if ! [ -x "$(command -v) puppetserver" ]; then
        sudo yum update -y
        echo "This machine does not have puppetserver installed"
        echo "Attempting to install"
        sudo su -
        rpm -Uvh https://yum.puppet.com/puppet-release-el-8.noarch.rpm
        yum install -y puppetserver git
    fi
  SHELL
end
