# -*- mode: ruby -*-
# vi: set ft=ruby :

# General project settings
#################################

# IP Address for the host only network, change it to anything you like
# but please keep it within the IPv4 private network range
ip_address = "172.22.22.22"

# The project name is base for directories, hostname and alike
project_name = "projectname"

# MySQL and PostgreSQL password - feel free to change it to something
# more secure (Note: Changing this will require you to update the index.php example file)
database_password = "root"



# Vagrant configuration
#################################

Vagrant.configure("2") do |config|
  # Enable Berkshelf support
  config.berkshelf.enabled = true

  # Use the omnibus installer for the latest Chef installation
  config.omnibus.chef_version = :latest

  # Define VM box to use
  config.vm.box = "precise32"
  config.vm.box_url = "http://files.vagrantup.com/precise32.box"

  # VB setup
  #################################
  # we will try to autodetect this path. 
  # However, if we cannot or you have a special one you may pass it like:
  # config.vbguest.iso_path = "#{ENV['HOME']}/Downloads/VBoxGuestAdditions.iso"
  # or
  # config.vbguest.iso_path = "http://company.server/VirtualBox/%{version}/VBoxGuestAdditions.iso"

  # set auto_update to false, if you do NOT want to check the correct 
  # additions version when booting this machine
  config.vbguest.auto_update = true

  # do NOT download the iso file from a webserver
  config.vbguest.no_remote = true




  # Set share folder
  config.vm.synced_folder "./public" , "/var/www/" + project_name + "/",  :nfs => true

  config.vm.provision :shell, :inline => "sed -i 's#us.archive.ubuntu.com#mirrors.163.com#' /etc/apt/sources.list "
  config.vm.provision :shell, :inline => "echo \"Asia/Shanghai\" | sudo tee /etc/timezone && dpkg-reconfigure --frontend noninteractive tzdata"

  config.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--memory", "1024"]
      v.customize ["modifyvm", :id, "--cpus", 2]
  end

  config.vm.usable_port_range = (10200..10500)


  # Use hostonly network with a static IP Address and enable
  # hostmanager so we can have a custom domain for the server
  # by modifying the host machines hosts file
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.vm.define project_name do |node|
    node.vm.hostname = project_name + ".local"
    node.vm.network :private_network, ip: ip_address
    node.hostmanager.aliases = [ "www." + project_name + ".local" ]
  end
  config.vm.provision :hostmanager

  # Enable and configure chef solo
  config.vm.provision :chef_solo do |chef|
    chef.add_recipe "runit"
    chef.add_recipe "openssl"
    #chef.add_recipe "ohai"
    chef.add_recipe "apt"
    chef.add_recipe "git"

    chef.add_recipe "php"
    chef.add_recipe "php-fpm"
    
    chef.add_recipe "nginx"
    
    chef.add_recipe "mysql::server"
    chef.add_recipe "mysql::client"
    chef.add_recipe "composer"


    chef.add_recipe "redis::install_from_package"
    chef.add_recipe "redis::client"

    chef.add_recipe "memcached"

    #chef.add_recipe "app::packages"
    #chef.add_recipe "app::web_server"
    chef.add_recipe "app::vhost"
    #chef.add_recipe "app::db"

    versions = {};
    versions['php5'] = '5.5.*'
    versions['php5-mysql'] = '5.5.*'
    versions['php5-pgsql'] = '5.5.*'
    versions['php5-curl'] = '5.5.*'
    versions['php5-mcrypt'] = '5.5.*'
    versions['php5-cli'] = '5.5.*'
    versions['php5-fpm'] = '5.5.*'
    versions['php-pear'] = '5.5.*'
    versions['php5-imagick'] = '3.*'

    chef.json = {
      :app => {
        :name           => project_name
      },
      :mysql => {
        :server_root_password   => database_password,
        :server_repl_password   => database_password,
        :server_debian_password => database_password,
        :bind_address           => ip_address,
        :allow_remote_root      => true
      },
      :php => {
              # Customize PHP modules here
              :packages                => %w{ php5 php5-dev php5-cli php-pear php5-apcu php5-mysql php5-curl php5-mcrypt php5-memcached php5-gd php5-json },

              # It is necessary to specify a custom conf dir as we are using Apache 2.4
              :ext_conf_dir            => "/etc/php5/mods-available"
      },
      'versions' => versions
    }
  end
end
