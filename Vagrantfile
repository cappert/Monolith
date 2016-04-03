# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"
Vagrant.require_version ">= 1.8.1"

# Use rbconfig to determine if we're on a windows host or not.
require 'rbconfig'
is_windows = (RbConfig::CONFIG['host_os'] =~ /mswin|mingw|cygwin/)

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  options = {
    :box              => 'boxcutter/debian82',
    :box_version      => '~> 2.0.5',
    :domain_tld       => 'project',
    :memory           => 1024,
    :cpu              => 1,
    :debug            => false
  }

  # Box
  config.vm.box = options[:box]
  config.vm.box_version = options[:box_version]

  # Network
  config.vm.network :private_network, type: :dhcp

  # SSH
  config.ssh.forward_agent = true

  # Synced Folders
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Landrush
  config.landrush.enabled = true
  config.landrush.guest_redirect_dns = false

  # Cachier
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :machine

    config.cache.synced_folder_opts = {
      type: :nfs,
      mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
    }

    config.cache.enable :generic
  end

  # VMWare
  config.vm.provider :vmware_fusion do |v, override|
    v.vmx["memsize"] = options[:memory]
    v.vmx["numvcpus"] = options[:cpu]
  end

  # VirtualBox.
  config.vm.provider :virtualbox do |v|
    v.memory = options[:memory]
    v.cpus = options[:cpu]

    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
  end

  # Configuration for the front env
  config.vm.define "monolith" do |front|

    frontoptions = {
      :hostname => 'www',
      :domain   => 'monolith',
      :folders  => {
          'monolith' => '/var/www/'
      }
    }

    # Box
    front.vm.hostname = ((false === frontoptions[:hostname].empty?) ? (frontoptions[:hostname] + '.') : '') + frontoptions[:domain] + '.' + options[:domain_tld]
    config.landrush.tld = front.vm.hostname

    frontoptions[:folders].each do |host, guest|
      front.vm.synced_folder host, guest, create: true, type: 'nfs', mount_options: ['nolock', 'actimeo=1', 'fsc']
    end

    front.vm.provider :vmware_fusion do |v, override|
      v.vmx["displayName"] = front.vm.hostname
    end

    front.vm.provider :virtualbox do |v|
      v.name = front.vm.hostname
    end

    # Provisioniers
    front.vm.provision :ansible do |ansible|
      ansible.playbook       = 'ansible/init.yml'
      ansible.verbose        = options[:debug] ? 'vvvv' : false
      ansible.sudo           = true
      ansible.tags           = ["init"]
      ansible.groups         = {
        "dev" => [frontoptions[:domain]],
        "provision:children" => ["dev"],
        "provision:vars" => { "isFirstRun" => true }
      }
    end

    front.vm.provision :ansible do |ansible|
      ansible.playbook       = 'ansible/build.yml'
      ansible.verbose        = options[:debug] ? 'vvvv' : false
      ansible.sudo           = true
      ansible.tags           = ["build"]
      ansible.groups         = {
        "dev" => [frontoptions[:domain]],
        "provision:children" => ["dev"],
        "provision:vars" => { "isFirstRun" => true }
      }
    end
  end

end
