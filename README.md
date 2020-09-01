# Vagrant Rancher Test Environment
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

## Minimum Recommended Available Resources
In order to run a full minimal HA environment (`K8S_NODES` and `RANCHER_NODES` set to `3`) on your machine, we recommend a system with at least 6 physical CPU cores (12 with Hyper-Threading) and 20GB memory.

If you want to run this setup on your local development machine or any other less powerful system, it's recommended to scale the node count of the `K8S_NODES` and `RANCHER_NODES` to `1`. In this case its also recommended to set `MEMORY_RANCHER_NODES` to `4096` and `MEMORY_K8S_NODES` to at least `2048` (depending on the workload you are going to deploy).

Relevant configuration settings (see inside the `Vagrantfile`s):
```
# Resources
CPU_CORES_RANCHER_NODES = 2
MEMORY_RANCHER_NODES = 4096
CPU_CORES_K8S_NODES = 2
MEMORY_K8S_NODES = 2048
CPU_CORES_CTL_LB = 1
MEMORY_CTL_LB = 512

# Node Count
K8S_NODES = 1
RANCHER_NODES = 1
```

## Prerequisites for both Setups
Choose between libvirt (KVM) or VirtualBox.

### Install Vagrant & Libvirt Plugin (optional)
```bash
wget https://releases.hashicorp.com/vagrant/2.2.10/vagrant_2.2.10_linux_amd64.zip
sudo dpkg -i vagrant_2.2.10_linux_amd64.zip
sudo apt install -y libvirt-dev libvirt-bin qemu-utils qemu-kvm
sudo vagrant plugin install vagrant-libvirt
```

### VirtualBox
Install Virtualbox according to https://www.virtualbox.org/wiki/Linux_Downloads.

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
3.  Choose your Vagrant provider inside the `Vagrantfile` (variable `PROVIDER`).
4. Spin up the Vagrant VMs:
```bash
vagrant box update
vagrant up --provision --no-parallel --provider=virtualbox
# or
vagrant up --provision --no-parallel --provider=libvirt
```
5. To verify the VM setup, access the `ctl-node` VM using `vagrant ssh ctl-node`.
6. Test if passwordless SSH access is working:
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