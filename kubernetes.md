# Environment K8S Mac M1
Create a local server for k8s to playground

# Pre Req
* https://canonical.com/multipass 
** to create ubuntu VMs on demand  
*** https://canonical.com/multipass/docs/install-multipass#p-113607-check-prerequisites

#####
    brew install --cask multipass

# on multipass

# Create Master

#####
    multipass networks

    check all the networks available, remember the name of the network on your host machine.

get any random MAC address to put in your ubuntu VM: https://www.browserling.com/tools/random-mac

#####
    multipass launch --disk 10G --memory 3G --cpus 2 --name k8smaster --network name=en0,mode=manual,mac="7c:38:2c:b8:c2:cb" jammy

##### expected message
    Launched: k8smaster

##### set public IP
    multipass exec -n k8smaster -- sudo bash -c 'cat << EOF > /etc/netplan/10-custom.yaml
    network:
    version: 2
    ethernets:
        extra0:
        dhcp4: no
        match:
            macaddress: "7c:38:2c:b8:c2:cb"
        addresses: [192.168.64.101/24]
    EOF'

#####
    multipass exec -n k8smaster -- sudo netplan apply

##### get your IP: 
    multipass info k8smaster | grep IPv4 -A1


# 192.168.68.93 host ip
# k8s master
       192.168.64.4 
       192.168.64.101 static setted

##### test ping from host and from master
    multipass exec -n k8smaster -- ping 192.168.68.93

# Create Worker 01

##### create worker
    multipass launch --disk 10G --memory 3G --cpus 2 --name k8sworker01 --network name=en0,mode=manual,mac="7a:2b:5c:2c:aa:c8" jammy

##### set public ip
    multipass exec -n k8sworker01 -- sudo bash -c 'cat << EOF > /etc/netplan/10-custom.yaml  
    network:
    version: 2
    ethernets:
        extra0:
        dhcp4: no
        match:
            macaddress: "7a:2b:5c:2c:aa:c8"
        addresses: [192.168.64.102/24]
    EOF'

##### apply 
    multipass exec -n k8sworker01 -- sudo netplan apply

# Create Worker 02

##### create worker
    multipass launch --disk 10G --memory 3G --cpus 2 --name k8sworker0 --network name=en0,mode=manual,mac="52:54:00:4b:cd:ab" jammy

##### set public ip
    multipass exec -n k8sworker02 -- sudo bash -c 'cat << EOF > /etc/netplan/10-custom.yaml  
    network:
    version: 2
    ethernets:
        extra0:
        dhcp4: no
        match:
            macaddress: "52:54:00:4b:cd:ab"
        addresses: [192.168.64.103/24]
    EOF'

##### apply 
    multipass exec -n k8sworker02 -- sudo netplan apply

##### set /etc/host/ to easy work with
    multipass shell k8smaster
    multipass shell k8sworker01
    multipass shell k8sworker02

##### put all itens according your ip 
    192.168.64.101 k8smaster
    192.168.64.102 k8sworker01
    192.168.64.103 k8sworker02








    kubeadm join 192.168.64.101:6443 --token 6jug5o.rv8d3itmuibp2zma \
	--discovery-token-ca-cert-hash sha256:04a684cd8e5dade875e3af966fd907ce68bbb73b6cd950f6d4f8d8057755bf6b





# troubleshooting
##### if any VM was stucked some commands on multipass: 
    # list all VM's with status
    multipass ls .
##### fix any unknown status or VM stucked 
    ps -ef | grep -i multipass | awk '{print "sudo kill -9 "$2}' | sh
