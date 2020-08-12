# -*- mode: ruby -*-
# vi: set ft=ruby :

RKE_VERSION = "v1.1.4"
KUBECTL_VERSION = "v1.18.0"
HELM_VERSION = "v3.2.4"
# Check https://rancher.com/docs/rke/latest/en/os/#installing-docker for the Rancher Docker version:
RANCHER_DOCKER_VERSION = "18.09.2"
CPU_CORES = 2
MEMORY = 2048
K8S_NODES = 3
RANCHER_NODES = 3
CONTROL_NODE_IP = "172.20.20.5"
# Specify only the first 3 octets
IP_RANGE_FOR_NODES = "172.20.20"
# Place Rancher nodes inside .10 - .19
RANCHER_NODES_BASE_IP = "172.20.20.1"
# Place K8s nodes inside .20 - .29
K8S_NODES_BASE_IP = "172.20.20.2"

$prepare_ssh_access = <<SCRIPT

# Deploy key
[ -f /home/vagrant/.ssh/id_rsa ] || {
    cp /vagrant/id_rsa /home/vagrant/.ssh/id_rsa
    cp /vagrant/id_rsa.pub /home/vagrant/.ssh/id_rsa.pub
    chmod 0600 /home/vagrant/.ssh/id_rsa
    chmod 0600 /home/vagrant/.ssh/id_rsa.pub
}
# Allow ssh passwordless
grep 'vagrant@rancher-lab' ~/.ssh/authorized_keys &>/dev/null || {
  cat /vagrant/id_rsa.pub >> ~/.ssh/authorized_keys
  chmod 0600 ~/.ssh/authorized_keys
}
# Exclude all nodes from host checking
cat > ~/.ssh/config <<EOF
Host *
   StrictHostKeyChecking no
   UserKnownHostsFile=/dev/null
EOF
chmod -R 600 ~/.ssh/config
# Allow PublicKeyAuthentication
sudo sed -i 's|#PubkeyAuthentication.*|PubkeyAuthentication yes|g' /etc/ssh/sshd_config
sudo sed -i 's|#AuthorizedKeysFile.*|AuthorizedKeysFile	.ssh/authorized_keys|g' /etc/ssh/sshd_config
# Populate /etc/hosts
for x in {10..#{9+RANCHER_NODES}}; do
  grep #{IP_RANGE_FOR_NODES}.${x} /etc/hosts &>/dev/null || {
      echo #{IP_RANGE_FOR_NODES}.${x} rancher-node-${x##?} | sudo tee -a /etc/hosts &>/dev/null
  }
done
for x in {20..#{19+K8S_NODES}}; do
  grep #{IP_RANGE_FOR_NODES}.${x} /etc/hosts &>/dev/null || {
      echo #{IP_RANGE_FOR_NODES}.${x} k8s-node-${x##?} | sudo tee -a /etc/hosts &>/dev/null
  }
done

SCRIPT

$rke_installation = <<SCRIPT

curl -LOsS https://github.com/rancher/rke/releases/download/$1/rke_linux-amd64
mv rke_linux-amd64 rke
chmod +x rke
mv rke /usr/local/bin/rke

SCRIPT

$kubectl_installation = <<SCRIPT

curl -LOsS https://storage.googleapis.com/kubernetes-release/release/$1/bin/linux/amd64/kubectl
chmod +x kubectl
mv kubectl /usr/local/bin/kubectl
yum install bash-completion -y
echo 'source /usr/share/bash-completion/bash_completion' >> /home/vagrant/.bashrc
echo 'source <(kubectl completion bash)' >> /home/vagrant/.bashrc
echo 'alias k=kubectl' >> /home/vagrant/.bashrc
echo 'complete -F __start_kubectl k' >> /home/vagrant/.bashrc

SCRIPT

$ansible_installation = <<SCRIPT

yum check-update
yum install epel-release git -y
yum check-update
yum install ansible -y

SCRIPT

$helm_installation = <<SCRIPT

curl -LOsS https://get.helm.sh/helm-$1-linux-amd64.tar.gz
tar xzf helm-$1-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
rm -rf ./linux-amd64/
rm -f helm-$1-linux-amd64.tar.gz

SCRIPT

$docker_installation = <<SCRIPT

yum check-update
curl https://releases.rancher.com/install-docker/$1.sh | sh
usermod -aG docker vagrant

SCRIPT

Vagrant.configure("2") do |config|
    #Common setup
    config.vm.box = "centos/7"
    config.vm.synced_folder ".", "/vagrant"
    config.vm.provision "prepare_ssh_access", type: "shell", inline: $prepare_ssh_access, privileged: false
    config.vm.provider :libvirt do |libvirt|
        libvirt.cpus = CPU_CORES
        libvirt.memory = MEMORY
        libvirt.nested = false
    end
    # Control node
    config.vm.define "ctl-node" do |ctl_node|
        ctl_node.vm.network :private_network, ip: "#{CONTROL_NODE_IP}"
        ctl_node.vm.hostname = "ctl-node"
        ctl_node.vm.provision "rke_installation", type: "shell", inline: $rke_installation, args: [RKE_VERSION], privileged: true
        ctl_node.vm.provision "kubectl_installation", type: "shell", inline: $kubectl_installation, args: [KUBECTL_VERSION], privileged: true
        ctl_node.vm.provision "ansible_installation", type: "shell", inline: $ansible_installation, privileged: true
        ctl_node.vm.provision "helm_installation", type: "shell", inline: $helm_installation, args: [HELM_VERSION], privileged: true
    end
    # Setup Rancher nodes
    (0..RANCHER_NODES-1).each do |i|
        config.vm.define "rancher-node-#{i}" do |rancher_node|
            rancher_node.vm.network :private_network, ip: "#{RANCHER_NODES_BASE_IP}#{i}"
            rancher_node.vm.hostname = "rancher-node-#{i}"
            rancher_node.vm.network :forwarded_port, guest: 80, host: 80, host_ip: "127.0.0.1"
            rancher_node.vm.network :forwarded_port, guest: 443, host: 443, host_ip: "127.0.0.1"
            rancher_node.vm.provision "docker_installation", type: "shell", inline: $docker_installation, args: [RANCHER_DOCKER_VERSION], privileged: true
        end
    end
    # Setup K8s nodes
    (0..K8S_NODES-1).each do |i|
        config.vm.define "k8s-node-#{i}" do |k8s_node|
            k8s_node.vm.network :private_network, ip: "#{K8S_NODES_BASE_IP}#{i}"
            k8s_node.vm.hostname = "k8s-node-#{i}"
            k8s_node.vm.network :forwarded_port, guest: 80, host: 80, host_ip: "127.0.0.1"
            k8s_node.vm.network :forwarded_port, guest: 443, host: 443, host_ip: "127.0.0.1"
            k8s_node.vm.network :forwarded_port, guest: 6443, host: 6443, host_ip: "127.0.0.1"
            k8s_node.vm.provision "docker_installation", type: "shell", inline: $docker_installation, args: [RANCHER_DOCKER_VERSION], privileged: true
        end
    end
end