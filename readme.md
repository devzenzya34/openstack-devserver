# CREATE A PRIVATE CLOUD WITH OPENSTACK
    Deploy on Virtualbox with vagrant and ansible for testing environment
## DEVSTACK INSTALL on VirtualBOX
### 1 - check gateway:
    ip route show
    ex : [default via 192.168.1.1 dev enp0s3] 
### 2 - Configure network:
    sudo nano /etc/netplan/01-netcfg.yaml
    network:
    version: 2
    renderer: networkd
    ethernets:
        enp0s3:  # Remplacez par le nom de votre interface réseau
        dhcp4: no
        addresses:
            - 192.168.1.100/24  # Adresse IP statique
        gateway4: 192.168.1.1  # Passerelle par défaut
        nameservers:
            addresses:
            - 8.8.8.8  # Serveur DNS
            - 8.8.4.4
### 3 - sudo netplan apply
### 4 - Launch ubuntu Serveur
    ip a 
### 5 - Check connection:
    from server: ping <ADRESSE_IP_HOTE>
    from hote: ping <ADRESSE_IP_HOTE>

### 6 - Open Vm console:
    new user stack
    sudo useradd -s /bin/bash -d /opt/stack -m stack
    sudo chmod +x /opt/stack
### 7 - change to stack user
    echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
    sudo -u stack -i
### 8 - clone devstack repo 
    sudo git clone https://opendev.org/openstack/devstack
    cd devstack
### 9 -  create a local.conf at root of devstack folder (no sudo)
    [[local|localrc]]
    ADMIN_PASSWORD=secret
    DATABASE_PASSWORD=$ADMIN_PASSWORD
    RABBIT_PASSWORD=$ADMIN_PASSWORD
    SERVICE_PASSWORD=$ADMIN_PASSWORD
    HOST_IP=<VOTRE_IP_PUBLIQUE_OU_PRIVEE>
    FLAT_INTERFACE=eth0
    FLOATING_RANGE=192.168.1.224/27
    FIXED_RANGE=10.11.12.0/24
    FIXED_NETWORK_SIZE=256
### 10 - Install
    ./stack.sh


