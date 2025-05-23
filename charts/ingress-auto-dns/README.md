# ingress-auto-dns Helm Chart

## Overview

`ingress-auto-dns` is a Helm chart designed to rapidly bootstrap an automated internal DNS system for newly created Kubernetes clusters. It provisions and configures **CoreDNS**, **etcd**, **ExternalDNS**, and **NGINX Ingress Controller**, enabling seamless internal ingress DNS resolution in a fully automated and repeatable way.

All components in this chart are included as Helm chart dependencies for modularity and maintainability.

---

## Requirements

- Kubernetes cluster with access to specified NodePorts (31100, 31101, 31200)
- Helm 3+

---

## Components

### 1. etcd

A distributed key-value store used as a backend for CoreDNS and ExternalDNS. This chart deploys `bitnami/etcd`.

**Key Settings:**
- Image: `bitnami/etcd:3.5.21-debian-12-r4`
- RBAC authentication: **disabled**

### 2. CoreDNS

DNS server configured to handle internal DNS resolution for ingress resources.

**Key Settings:**
- Image: `coredns/coredns:1.12.0`
- Runs as a non-cluster DNS service (not replacing kube-dns)
- Service Type: `NodePort` (Port 31101)
- Custom Corefile support via `coredns-custom` ConfigMap
- Domain name: `home.lab`
- Plugins included:
  - `errors`, `health`, `ready`, `import`, `forward`, `cache`, `loop`, `reload`, `loadbalance`

### 3. ExternalDNS

ExternalDNS is configured to automatically generate CoreDNS records from Kubernetes ingress resources.

**Key Settings:**
- Image: `registry.k8s.io/external-dns/external-dns:v0.16.1`
- Provider: `coredns`
- Sources: `ingress`
- Registry: `noop`
- Policy: `sync`
- Uses `ETCD_URLS` from a Kubernetes Secret (`etcd-urls`) for etcd connectivity

### 4. ingress-nginx

NGINX Ingress Controller configured for internal use with predictable NodePort exposure.

**Key Settings:**
- Image: `ingress-nginx/controller:v1.12.2`
- Exposed on:
  - HTTP: `31100`
  - HTTPS: `31200`
- Static external IP: `192.168.0.73`

---
## Create a kind cluster
```bash
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 31100
    hostPort: 80
    protocol: TCP
  - containerPort: 31200
    hostPort: 443
    protocol: TCP
  - containerPort: 31101
    hostPort: 5153
    protocol: UDP
  - containerPort: 31102
    hostPort: 5253
    protocol: TCP
- role: worker
EOF
```

## Installation

To install this chart:

```bash
helm repo add morrismusumi-helm-charts https://morrismusumi.github.io/helm-charts
helm repo update

helm install ingress-auto-dns morrismusumi-helm-charts/ingress-auto-dns
```

---

## Usage with Dnsmasq

Once installed you can forward DNS queries for your domain to any nodeIP on the chosen port. 

Here is an example using dnsmasq on MacOS, but the same proceedure can be adapted to your OS of choice.

```bash
# install dnsmasq
sudo brew install dnsmasq

# configure dnsmasq to forward dns queries for 
# internal.lab to coredns port 5153 exposed on the kind cluster
sudo  echo "server=/internal.lab/127.0.0.1#5153" > /opt/homebrew/etc/dnsmasq.d/internal.conf

# ensure dnsmasq is started
sudo brew services start dnsmasq

OR

sudo brew services restart dnsmasq

# check that dnsmasq is running
sudo brew services list

# test dns resolution
dig @127.0.0.1 nginx-app.internal.lab
```

Add 127.0.0.1 as a dns server in network settings.

---

## Custom Configuration

Customize the deployment by editing `values.yaml` or using `--set` parameters. For example:

```bash
helm install ingress-auto-dns morrismusumi-helm-charts/ingress-auto-dns \
  --set coredns.domainName=internal.lab \
  --set ingress-nginx.controller.service.externalIPs={192.168.0.100}
```

---

## Notes

- This chart is ideal for homelabs, internal environments or test clusters in the cloud where quick DNS automation is required.
- Designed for use with trusted internal networks; no public DNS is configured.
- CoreDNS can be extended via `coredns-custom` ConfigMap mounted at `/opt/coredns`.

