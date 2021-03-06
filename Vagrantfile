# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'socket'

module Vagrant
  # TODO machines.each do ...
  @default_machines = {
    web: { primary: true,  active: true,  memory: '1024', hostname: 'vagrant-web.test', subdomains: []},
    db:  { primary: false, active: false, memory: '512',  hostname: 'vagrant-db.test',  subdomains: []}
  }

  def self.configure_machines?
    @configure_machines
  end

  def self.configure_machines(machines = {})
    @configure_machines = true

    @default_machines.each do |type, values|
      values.merge(machines[type] || {}).each do |name, value|
        accessor_name = name == :active ? :"#{type}?" : :"#{type}_#{name}"
        define_singleton_method accessor_name do
          value
        end
      end
    end

    if block_given?
      Vagrant.configure(2) do |config|
        yield config
      end
    end
  end

  def self.free_ip(type)
    ip_file = ".vagrant/#{type}_ip"
    return File.readlines(ip_file).first.strip if File.exist?(ip_file)

    network_file = '.vagrant/network'
    network = File.exist?(network_file) ? File.readlines(network_file).first.strip : ''
    networks = [''].concat Socket.getifaddrs.map{ |i| i.addr.ip_address.sub(/\.\d+$/, '') if i.addr.ipv4? }.compact
    loop do
      break unless networks.include?(network)
      network = "192.168.#{rand(4..254)}"
    end
    File.open(network_file, 'w') { |f| f.write(network) }
    ip = "#{network}.#{rand(2..254)}"
    File.open(ip_file, 'w') { |f| f.write(ip) }
    ip
  end
end

Vagrant.configure(2) do |config|
  if Vagrant.configure_machines?
    config.vm.box = 'bento/ubuntu-16.04'

    config.ssh.forward_agent = true

    if Vagrant.has_plugin? 'vagrant-hostmanager'
      config.hostmanager.enabled = true
      config.hostmanager.manage_host = true
    end

    if Vagrant.web?
      config.vm.define :web, primary: true do |node|
        node.vm.hostname = Vagrant.web_hostname
        node.vm.network :private_network, ip: (ip = Vagrant.free_ip(:web))
        if Vagrant.has_plugin?('vagrant-hostmanager') && Vagrant.web_subdomains.any?
          node.hostmanager.aliases = Vagrant.web_subdomains.map{ |name| "#{name}.#{Vagrant.web_hostname}" }
        end

        config.vm.provider :virtualbox do |vb|
          vb.memory = Vagrant.web_memory
          vb.name = "web-#{ip}"
        end
      end
    end

    if Vagrant.db?
      config.vm.define :db do |node|
        node.vm.hostname = Vagrant.db_hostname
        node.vm.network :private_network, ip: (ip = Vagrant.free_ip(:db))
        if Vagrant.has_plugin?('vagrant-hostmanager') && Vagrant.db_subdomains.any?
          node.hostmanager.aliases = Vagrant.db_subdomains.map{ |name| "#{name}.#{Vagrant.db_hostname}" }
        end

        config.vm.provider :virtualbox do |vb|
          vb.memory = Vagrant.db_memory
          vb.name = "db-#{ip}"
        end
      end
    end
  end
end
