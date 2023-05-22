# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"
    config.vm.define "server" do |server| # конфиг сервера
      server.vm.hostname = "server.loc"
      server.vm.network "private_network", ip: "192.168.56.10"
    end

    config.vm.define "client" do |client| # конфиг клиента
      client.vm.hostname = "client.loc"
      client.vm.network "private_network", ip: "192.168.56.20"
    end
      
    config.vm.provision "ansible" do |ansible|  # вызываем playbook для развертывания нужной инфраструктуры на машинах
      ansible.verbose = "v"
      ansible.playbook = "vpn.yml" # файл playbook
    end

  end
