# Vagrant Rancher Test Environment (KVM-based)
This Vagrant environment should help you to deploy a Rancher Setup playground within a few minutes.

## Install Vagrant & Libvirt Plugin
```bash
wget https://releases.hashicorp.com/vagrant/2.2.9/vagrant_2.2.9_x86_64.deb
sudo dpkg -i vagrant_2.2.9_x86_64.deb
sudo apt install -y libvirt-dev libvirt-bin qemu-utils qemu-kvm
sudo vagrant plugin install vagrant-libvirt
```

## VM Setup
Generate the SSH key pair:
```bash
ssh-keygen -t rsa -f id_rsa -b 4096 -C 'vagrant@rancher-lab' -q -N ''
```

**Note:** By default there are 3 Rancher K8s cluster nodes and 3 user K8s cluster nodes configured. It's possible to scale these counters down to 1 node per cluster. Simply edit the `K8S_NODES` and `RANCHER_NODES` value inside the `Vagrantfile`.

Start the VMs:
```bash
vagrant up
```

Check the status from the VMs:
```bash
$ vagrant status
Current machine states:

ctl-node                  running (libvirt)
rancher-node-0            running (libvirt)
rancher-node-1            running (libvirt)
rancher-node-2            running (libvirt)
k8s-node-0                running (libvirt)
k8s-node-1                running (libvirt)
k8s-node-2                running (libvirt)
```

## Getting Started
1. Access the `ctl-node` VM using `vagrant ssh ctl-node`.
2. Test if passwordless SSH access is working:
```bash
$ ssh rancher-node-0
$ ssh k8s-node-0
```

## Teardown
```bash
vagrant destroy -f
```

## Credit
- https://medium.com/@jazz.twk/learn-docker-swarm-with-vagrant-47dd52b57bcc
- https://gist.github.com/kikitux/86a0bd7b78dca9b05600264d7543c40d