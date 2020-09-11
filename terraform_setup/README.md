# Vagrant Rancher Test Environment (KVM-based)
This Vagrant environment should help you to automatically deploy an (optionally HA) testing/development Rancher Setup playground within a few minutes using the [Terraform resources from Puzzle ITC](https://github.com/puzzle/terraform-rancher).

**Make sure you already completed the steps documented in [../README.md](../README.md).**

## Access the Control Node
1. Access the `ctl-node` VM using `vagrant ssh ctl-node`.
2. Run the following commands:
```bash
cd /opt/rancher-setup
```

## TLS Certificate

**Important:** 
- Replace `172.20.20.254.xip.puzzle.ch` with the actual public vIP/domainname you would like to use for the keepalived VRRP. For libvirt the default public bridge IP subnet probably is `192.168.121.0/24`. So if you choose to run it on libvirt you could use `192.168.121.254` instead of `172.20.20.254`.

Generate a self-signed CA and server certificate (optional, inside `/opt/rancher-setup` on `ctl-node`):
```
# Generate root CA cert and key
openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout ca.key -out ca.crt -days 3650 -subj "/C=CH/ST=Bern/L=Bern/O=Puzzle ITC/OU=mid/CN=Vagrant Rancher Root CA"
# Generate server cert request and key
openssl req -new -newkey rsa:4096 -sha256 -nodes -keyout tls.key -out tls.csr -days 3650 -subj "/C=CH/ST=Bern/L=Bern/O=Puzzle ITC/OU=mid/CN=172.20.20.254.xip.puzzle.ch"
# Sign server cert request with CA
openssl x509 -req -in tls.csr -set_serial 1000 -CA ca.crt -CAkey ca.key -days 3650 -out tls.crt
# Validate the signature
openssl verify -verbose -CAfile ca.crt tls.crt
# Copy the content from ca.crt to cacerts.pem (required for the later setup)
cp ca.crt cacerts.pem
```

## Configuration

### Terraform Configuation / Initialization
1. Get the [terraform-rancher](https://github.com/puzzle/terraform-rancher) files from Github.
```bash
git clone https://github.com/puzzle/terraform-rancher.git
cd terraform-rancher/
```

### Terraform Apply
```bash
terraform apply
```

Visit the Rancher UI at https://172.20.20.254.xip.puzzle.ch (or whatever your URL is) and have fun ;).