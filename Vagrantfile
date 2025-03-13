Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.box_check_update = false
  config.vm.box = "bento/ubuntu-22.04"

  # Provision SSH key
  config.vm.provision "shell" do |s|
    ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
    s.inline = <<-SHELL
    mkdir -p /home/vagrant/.ssh
    echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
    chmod 600 /home/vagrant/.ssh/authorized_keys
    SHELL
  end

  config.vm.define "vbox" do |vbox|
    vbox.vm.hostname = "openstack-vm"
    vbox.vm.network "private_network", ip: "192.168.56.30"
    vbox.vm.network "public_network", ip: "192.168.1.30"
    vbox.vm.provider "virtualbox" do |vb|
      vb.memory = "8192"
      vb.cpus = 4
      vb.customize ["modifyvm", :id, "--uartmode1", "disconnected"]
    end
    
    # Add Ansible provisioning
    vbox.vm.provision "ansible" do |ansible|
      ansible.playbook = "devstack-deploy.yml"
    end
  end
end


