# Vagrant Rancher Test Environment (KVM-based)
This Vagrant environment should help you to deploy a Rancher Setup playground within a few minutes. There is a manual and an automated setup. Both setups have their own `Vagrantfile` but both will spin up the following VMs:

```bash
$ vagrant status
Current machine states:

ctl-node                  running (libvirt)
lb-node                   running (libvirt)
rancher-node-0            running (libvirt)
rancher-node-1            running (libvirt)
rancher-node-2            running (libvirt)
k8s-node-0                running (libvirt)
k8s-node-1                running (libvirt)
k8s-node-2                running (libvirt)
```

Nevertheless, of course you can customize the `rancher-node-*`/`k8s-node-*?` VM count directly via the regarding variable inside the `manual_setup/Vagrantfile` or `ansible_setup/Vagrantfile` files.

## Prerequisites for both Setups

### Install Vagrant & Libvirt Plugin
```bash
wget https://releases.hashicorp.com/vagrant/2.2.9/vagrant_2.2.9_x86_64.deb
sudo dpkg -i vagrant_2.2.9_x86_64.deb
sudo apt install -y libvirt-dev libvirt-bin qemu-utils qemu-kvm
sudo vagrant plugin install vagrant-libvirt
```

### SSH Keypair for passwordless Access (VM to VM)
Generate the SSH key pair:
```bash
ssh-keygen -t rsa -f id_rsa -b 4096 -C 'vagrant@rancher-lab' -q -N ''
```

## Getting Started

### VM Setup
1. `cd` into the `manual_setup/` or `ansible_setup/` sub directory depending on your wished installation method.
2. Copy the SSH key pair into the sub directory:
```bash
cp ../id_rsa* .
```
3. Spin up the Vagrant VMs:
```bash
vagrant up --provision --no-parallel
```
4. To verify the VM setup, access the `ctl-node` VM using `vagrant ssh ctl-node`.
5. Test if passwordless SSH access is working:
```bash
$ ssh rancher-node-0
$ ssh k8s-node-0
```

### Rancher Cluster Setup
In order to continue with the Rancher setup navigate to the regarding `README.md`.

- **Manual** Rancher cluster setup: [manual_setup/README.md](manual_setup/README.md)
- **Automated** Rancher cluster setup using Ansible: [ansible_setup/README.md](ansible_setup/README.md)

## Teardown
Run the following command inside the `./manual_setup/` or `./ansible_setup/` sub directory to shutdown and delete the Vagrant environment:
```bash
vagrant destroy -f
```

## Credit
- https://github.com/puzzle/ansible-rancher
- https://medium.com/@jazz.twk/learn-docker-swarm-with-vagrant-47dd52b57bcc
- https://gist.github.com/kikitux/86a0bd7b78dca9b05600264d7543c40d