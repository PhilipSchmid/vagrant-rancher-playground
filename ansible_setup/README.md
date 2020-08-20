# Vagrant Rancher Test Environment (KVM-based)
This Vagrant environment should help you to automatically deploy a Rancher Setup playground within a few minutes using the [Ansible Playbook from Puzzle ITC](https://github.com/puzzle/ansible-rancher).

**Make sure you already completed the steps documented in [../README.md](../README.md).**

## Access the Control Node
1. Access the `ctl-node` VM using `vagrant ssh ctl-node`.
2. Run the following commands:
```bash
cd /opt/rancher-setup
```

## TLS Certificate

**Important:** 
- Replace `192.168.121.254.xip.puzzle.ch` with the actual public vIP/domainname you would like to use for the keepalived VRRP (the same as specified in `PUBLIC_LB_NODE_IP` inside `Vagrantfile`).

Generate a self-signed CA and server certificate (optional, inside `/opt/rancher-setup` on `ctl-node`):
```
# Generate root CA cert and key
openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout ca.key -out ca.crt -days 3650 -subj "/C=CH/ST=Bern/L=Bern/O=Puzzle ITC/OU=mid/CN=Vagrant Rancher Root CA"
# Generate server cert request and key
openssl req -new -newkey rsa:4096 -sha256 -nodes -keyout tls.key -out tls.csr -days 3650 -subj "/C=CH/ST=Bern/L=Bern/O=Puzzle ITC/OU=mid/CN=192.168.121.254.xip.puzzle.ch"
# Sign server cert request with CA
openssl x509 -req -in tls.csr -set_serial 1000 -CA ca.crt -CAkey ca.key -days 3650 -out tls.crt
# Validate the signature
openssl verify -verbose -CAfile ca.crt tls.crt
# Copy the content from ca.crt to cacerts.pem (required for the later setup)
cp ca.crt cacerts.pem
```

## Configuration

### Ansible Playbook Configuation
1. Get the [ansible-rancher](https://github.com/puzzle/ansible-rancher) from Github and install pip modules.
```bash
git clone https://github.com/puzzle/ansible-rancher.git
cd ansible-rancher/
pip3 install --user pipenv
pipenv shell --three
pipenv install
```
**Note**: Please ensure the Python dependencies from `Pipfile` are installed properly by using `pipenv graph`.

2. Copy the preconfigured Ansible inventory files to the just "git cloned" `ansible-rancher` directory. Before that, ensure all configuration values inside these preconfigured Ansible inventory files are set according to your setup/needs.
**Important:** Especially also adjust the Rancher and K8s node lists inside `/opt/rancher-setup/ansible-rancher/inventories/site` according to your Vagrant node setup!
```bash
cp -rf /vagrant/ansible-rancher/* .
```

3. Set/Replace some configuration values:
```bash
sed -i ./inventories/host_vars/cluster_rancher.yml -e 's/certmanager_enabled.*/certmanager_enabled: false/g'
cat ../tls.crt | base64 -w 0 | xargs --replace=INSERTED -- sed -i ./inventories/host_vars/cluster_rancher.yml -e 's/rke_rancher_tls_crt.*/rke_rancher_tls_crt: "INSERTED"/g'
cat ../tls.key | base64 -w 0 | xargs --replace=INSERTED -- sed -i ./inventories/host_vars/cluster_rancher.yml -e 's/rke_rancher_tls_key.*/rke_rancher_tls_key: "INSERTED"/g'
cat ../cacerts.pem | base64 -w 0 | xargs --replace=INSERTED -- sed -i ./inventories/host_vars/cluster_rancher.yml -e 's/rke_rancher_tls_cacerts.*/rke_rancher_tls_cacerts: "INSERTED"/g'
```

### Installation
```bash
ansible-playbook -i ./inventories/site ./plays/site.yml
```

Visit the Rancher UI at https://192.168.121.254.xip.puzzle.ch (or whatever your URL is) and have fun ;).