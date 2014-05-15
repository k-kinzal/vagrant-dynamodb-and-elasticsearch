# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  config.vm.hostname = "vagrant"

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "CentOS 6.5 x86_64"

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  config.vm.box_url = "https://github.com/2creatives/vagrant-centos/releases/download/v6.5.1/centos65-x86_64-20131205.box"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network :forwarded_port, guest: 80, host: 8080
  config.vm.network :forwarded_port, guest: 8000, host: 8000 # DynamoDB local
  config.vm.network :forwarded_port, guest: 9200, host: 9200 # ElasticSearch

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.network :public_network

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  # config.ssh.forward_agent = true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider :virtualbox do |vb|
  #   # Don't boot with headless mode
  #   vb.gui = true
  #
  #   # Use VBoxManage to customize the VM. For example to change memory:
  #   vb.customize ["modifyvm", :id, "--memory", "1024"]
  # end
  #
  # View the documentation for the provider you're using for more
  # information on available options.

  config.vm.provision :shell, :inline => <<-EOS
    chef-solo --version | grep "Chef: 11.*.*" || curl -L https://www.opscode.com/chef/install.sh | sudo bash
  EOS

  # The path to the Berksfile to use with Vagrant Berkshelf
  # config.berkshelf.berksfile_path = "./Berksfile"

  # Enabling the Berkshelf plugin. To enable this globally, add this configuration
  # option to your ~/.vagrant.d/Vagrantfile file
  config.berkshelf.enabled = true

  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  config.vm.provision :chef_solo do |chef|
    chef.json = {
      :ark => {
        :package_dependencies => ["libtool", "autoconf"]
      },
      :java => {
        :install_flavor => "openjdk",
        :jdk_version => "7"
      },
      :elasticsearch => {
        :version => "1.1.0",
        :path => {
          :conf => "/etc/elasticsearch",
          :data => "/var/data/elasticsearch",
          :logs => "/var/log/elasticsearch",
        },
        :pid_path => "/var/run",
        :plugins => {
          :'elasticsearch/elasticsearch-analysis-kuromoji' => { :version => "2.0.0" },
          :'royrusso/elasticsearch-HQ' => {},
          :'com.github.kzwang/elasticsearch-river-dynamodb' => { :version => "1.0.0" },
          :'elasticsearch/marvel' => { :version => "latest"},
          :'elasticsearch/elasticsearch-lang-javascript' => { :version => "2.1.0" }
        }
      },
      :dynamodb_local => {
        :log_dir => "/var/log/dynamodb"
      }
    }

    chef.run_list = [
        "recipe[yum]",
        "recipe[yum-epel]",
        "recipe[yum-remi]",
        "recipe[java]",
        "recipe[elasticsearch]",
        "recipe[elasticsearch::plugins]",
        "recipe[dynamodb_local]",
        "recipe[selinux::disabled]",
        "recipe[iptables::disabled]"
    ]
  end

  config.vm.provision :shell, :inline => <<-EOS
    sudo service elasticsearch restart
  EOS
end
