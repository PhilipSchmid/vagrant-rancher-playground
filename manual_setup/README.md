# Vagrant Rancher Test Environment (KVM-based)
This Vagrant environment should help you to manually deploy a Rancher Setup playground within a few minutes. 

**Make sure you already completed the steps documented in [../README.md](../README.md).**

## Rancher RKE Cluster Installation
1. Access the `ctl-node` VM using `vagrant ssh ctl-node`.
2. Run the following commands:
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
172.20.20.10.xip.puzzle.ch   Ready    controlplane,etcd,worker   46s   v1.17.9
172.20.20.11.xip.puzzle.ch   Ready    controlplane,etcd,worker   46s   v1.17.9
172.20.20.12.xip.puzzle.ch   Ready    controlplane,etcd,worker   46s   v1.17.9
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
kubectl -n cert-manager rollout status deploy/cert-manager
kubectl get pods --namespace cert-manager
```

### Rancher
**Important:** 
- Replace `192.168.121.254.xip.puzzle.ch` with the actual public IP/domainname from the `lb-node` VM (the same as specified in `PUBLIC_LB_NODE_IP` inside `Vagrantfile`).
- Remove `--set privateCA=true` from the `helm upgrade` Rancher installation command if your certificate is officially signed.
- 

Generate a self-signed CA and server certificate (optional):
```
# Generate root CA cert and key
openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout ca.key -out ca.crt -days 3650 -subj "/C=CH/ST=Bern/L=Bern/O=Puzzle ITC/OU=mid/CN=Vagrant Rancher Root CA"
# Generate server cert request and key
openssl req -new -newkey rsa:4096 -sha256 -nodes -keyout tls.key -out tls.csr -days 3650 -subj "/C=CH/ST=Bern/L=Bern/O=Puzzle ITC/OU=mid/CN=192.168.121.254.xip.puzzle.ch"
# Sign server cert request with CA
openssl x509 -req -in tls.csr -set_serial 1000 -CA ca.crt -CAkey ca.key -days 3650 -out tls.crt
# Validate the signature
openssl verify -verbose -CAfile ca.crt tls.crt
```

Finally the Rancher installation:

```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
kubectl create namespace cattle-system
kubectl -n cattle-system create secret tls tls-rancher-ingress \
    --cert=tls.crt \
    --key=tls.key
cp ca.crt cacerts.pem
kubectl -n cattle-system create secret generic tls-ca \
    --from-file=cacerts.pem
helm upgrade --install  \
  --namespace cattle-system \
  --set hostname=192.168.121.254.xip.puzzle.ch \
  --set ingress.tls.source=secret \
  --set privateCA=true \
  --version 2.4.5 \
  rancher \
  rancher-latest/rancher
# Verify installation:
kubectl -n cattle-system rollout status deploy/rancher
kubectl -n cattle-system get pods
```

Visit the Rancher UI at https://192.168.121.254.xip.puzzle.ch (or whatever your URL is) and have fun ;).