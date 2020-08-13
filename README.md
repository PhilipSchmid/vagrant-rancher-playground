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
vagrant up --provision --no-parallel
```

Check the status from the VMs:
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

## Getting Started
1. Access the `ctl-node` VM using `vagrant ssh ctl-node`.
2. Test if passwordless SSH access is working:
```bash
$ ssh rancher-node-0
$ ssh k8s-node-0
```

## Rancher RKE Cluster Installation
```bash
cd /opt/rancher-setup
# Check if cluster.yml was generated properly:
cat cluster.yml
# Start the RKE cluster installation:
rke up
# Verify the cluster installation:
echo "export KUBECONFIG='/opt/rancher-setup/kube_config_cluster.yml'" >> /home/vagrant/.bashrc
source ~/.bashrc
kubectl get nodes
NAME                         STATUS   ROLES                      AGE   VERSION
172.20.20.10.xip.puzzle.ch   Ready    controlplane,etcd,worker   46s   v1.18.6
172.20.20.11.xip.puzzle.ch   Ready    controlplane,etcd,worker   46s   v1.18.6
172.20.20.12.xip.puzzle.ch   Ready    controlplane,etcd,worker   46s   v1.18.6
```

## Rancher Installation (via Helm Chart)
Source: https://rancher.com/docs/rancher/v2.x/en/installation/k8s-install/helm-rancher/

### Cert-Manager
```bash
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.16.0/cert-manager.crds.yaml
kubectl create namespace cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v0.16.0
# Verify installation:
kubectl get pods --namespace cert-manager
```

### Rancher
```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
kubectl create namespace cattle-system
helm upgrade --install  \
  --namespace cattle-system \
  --set hostname=127.0.0.1.xip.puzzle.ch \
  --version 2.4.5 \
  rancher \
  rancher-latest/rancher
# Verify installation:
kubectl -n cattle-system rollout status deploy/rancher
kubectl -n cattle-system get pods
```

Visit the Rancher UI at https://127.0.0.1.xip.puzzle.ch.

## Alternative: Puzzle ITC Rancher Setup
Source: https://github.com/puzzle/ansible-rancher

```bash
git clone https://github.com/puzzle/ansible-rancher.git
cd ansible-rancher/
cp -rf /vagrant/ansible-rancher/inventories .
ansible-playbook -i inventories/site plays/site.yaml
```

## Teardown
```bash
vagrant destroy -f
```

## Credit
- https://medium.com/@jazz.twk/learn-docker-swarm-with-vagrant-47dd52b57bcc
- https://gist.github.com/kikitux/86a0bd7b78dca9b05600264d7543c40d