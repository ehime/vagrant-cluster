# vi: set ft=ruby :
# Builds Puppet Master and multiple Puppet Agent Nodes using JSON config file
# Requires triggers, vagrant plugin install vagrant-triggers


class GuestFix < VagrantVbguest::Installers::Linux
  def install(opts=nil, &block)
    communicate.sudo('yum update', opts, &block)
    communicate.sudo('yum purge -y virtualbox-guest-dkms virtualbox-guest-utils virtualbox-guest-x11', opts, &block)
    super
    communicate.sudo('( [ -d /opt/VBoxGuestAdditions-5.0.14/lib/VBoxGuestAdditions ] && sudo ln -s /opt/VBoxGuestAdditions-5.0.14/lib/VBoxGuestAdditions /usr/lib/VBoxGuestAdditions ) || true', )
  end
end

# read vm and chef configurations from JSON files
nodes_config = (JSON.parse(File.read("config/cluster.json")))['nodes']

Vagrant.configure(2) do | config |
  config.vbguest.installer = GuestFix

  config.vm.box = "bento/centos-7.1"
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope  = :box
    config.cache.enable   :yum
    config.cache.enable   :puppet
  end

  # List with `vagrant status`
  nodes_config.each do | node |
    node_name   = node[0] # name of node
    node_values = node[1] # content of node

    config.vm.define node_name do | config |

      # configures all forwarding ports in JSON array
      config.ssh.forward_agent = true

      ports = node_values['ports']
      ports.each do | port |
        config.vm.network :forwarded_port,
          host:  port[':host'],
          guest: port[':guest'],
          id:    port[':id']
      end

      config.vm.hostname = node_name

      config.vm.network :private_network,
        ip: node_values[':ip'],
        auto_config: false

      config.vm.provider :virtualbox do | vb |
        #vb.gui = true
        vb.customize ["modifyvm", :id, "--memory",  node_values[':memory']]
        vb.customize ["modifyvm", :id, "--name",    node_name]
      end

      config.vm.provision :shell, :path => node_values[':bootstrap']
    end
  end
end
