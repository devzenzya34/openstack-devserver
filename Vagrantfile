# vi: set ft=ruby :

# Load environment variables if .env file exists
if File.exist?('.env')
    require 'dotenv'
    Dotenv.load
  end
  
  # Get environment variables with defaults
  BRIDGE_INTERFACE = ENV['BRIDGE_INTERFACE']
  PUBLIC_IP = ENV['PUBLIC_IP']
  PUBLIC_NETMASK = ENV['PUBLIC_NETMASK']
  PUBLIC_GATEWAY = ENV['PUBLIC_GATEWAY']
  
  Vagrant.configure("2") do |config|
    config.vm.box = "bento/ubuntu-22.04"
    config.vm.box_check_update = false
    
    # Disable the default synced folder
    config.vm.synced_folder ".", "/vagrant", disabled: true
  
    config.vm.define "devstack" do |devstack|
      devstack.vm.hostname = "devstack"
  
      # Bridge network configuration
      devstack.vm.network "public_network",
        bridge: BRIDGE_INTERFACE,
        ip: PUBLIC_IP,
        netmask: PUBLIC_NETMASK,
        gateway: PUBLIC_GATEWAY,
        auto_config: true
  
      # Add a second network adapter in NAT mode
      devstack.vm.network "private_network", type: "dhcp"
  
      # VirtualBox specific configuration
      devstack.vm.provider "virtualbox" do |vb|
        vb.name = "devstack_vm"
        vb.memory = 16384  # 16GB RAM
        vb.cpus = 8
        
        # Disable USB
        vb.customize ["modifyvm", :id, "--usb", "off"]
        vb.customize ["modifyvm", :id, "--usbehci", "off"]
        
        # Enable promiscuous mode on the bridge interface for OpenStack networking
        vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
        
        # Disable audio
        vb.customize ["modifyvm", :id, "--audio", "none"]
        
        # Enable nested virtualization
        vb.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
      end
  
      # Provision SSH key
      devstack.vm.provision "shell" do |s|
        ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
        s.inline = <<-SHELL
          mkdir -p /home/vagrant/.ssh
          echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
          chmod 600 /home/vagrant/.ssh/authorized_keys
        SHELL
      end
  
      # Configure network interfaces
      devstack.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y bridge-utils net-tools
  
        # Configure network interface
        cat > /etc/netplan/01-netcfg.yaml <<EOL
          network:
            version: 2
            renderer: networkd
            ethernets:
              enp0s3:
                dhcp4: no
                addresses: [#{PUBLIC_IP}/24]
                gateway4: #{PUBLIC_GATEWAY}
                nameservers:
                  addresses: [8.8.8.8, 8.8.4.4]
              enp0s8:
                dhcp4: yes
EOL
  
        # Apply netplan configuration
        netplan apply
  
        # Enable IP forwarding
        echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
        sysctl -p
  
        # Verify network configuration
        ip addr show
        route -n
      SHELL
    end
  end