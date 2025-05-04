# 📦 Pi-hole Helm Chart

Deploy [Pi-hole](https://pi-hole.net/) 📡 — a network-wide ad blocker — on your Kubernetes cluster using Helm 🚀.

---

## 🧰 Prerequisites

- 🐳 Kubernetes cluster
- 🛠️ Helm 3+
- 💾 A storage class (if persistence is enabled)
- 🌐 Ingress controller (if using ingress)

---

## 🚀 Installation

1. **Add the Helm repository** (if available):

   ```bash
   helm repo add morrismusumi-helm-charts https://morrismusumi.github.io/helm-charts
   helm repo update
   ```

2. **Install the chart with your custom values:**

   ```bash
   helm install pi-hole morrismusumi-helm-charts/pi-hole -n pi-hole --create-namespace
   ```

---

## ⚙️ Configuration

Customize your deployment via the `values.yaml`. Below is a summary of the default values used in this chart:

### 🔁 Replica Count

| Parameter      | Description               | Default |
|----------------|---------------------------|---------|
| `replicaCount` | Number of Pi-hole pods    | `1`     |

---

### 🐳 Image

| Parameter          | Description            | Default         |
|--------------------|------------------------|-----------------|
| `image.repository` | Docker image repo      | `pihole/pihole` |
| `image.tag`        | Image tag              | `2025.04.0`     |
| `image.pullPolicy` | Image pull policy      | `IfNotPresent`  |

---

### 🌐 Service

| Parameter               | Description               | Value      |
|-------------------------|---------------------------|------------|
| `service.type`          | Kubernetes service type   | `NodePort` |
| `service.ports`         | Exposed ports             | See below  |
| `service.ports[0]`      | DNS (UDP 53)              | `31101`    |
| `service.ports[1..3]`   | DNS (TCP 53), HTTP, HTTPS | Dynamic    |

---

### 🚪 Ingress

| Parameter               | Description             | Value                 |
|-------------------------|-------------------------|-----------------------|
| `ingress.enabled`       | Enable ingress          | `true`                |
| `ingress.className`     | Ingress class           | `""` (empty)          |
| `ingress.hosts[0].host` | Hostname                | `chart-example.local` |
| `ingress.paths`         | Path routing            | `/`                   |
| `ingress.tls`           | TLS support             | ❌ Disabled by default |

---

### 🌍 Environment Variables

| Name                              | Description             | Value           |
|-----------------------------------|-------------------------|-----------------|
| `TZ`                              | Time zone               | `Europe/Berlin` |
| `FTLCONF_webserver_api_password`  | Admin password          | `piholeadmin`   |
| `FTLCONF_dns_upstreams`           | Upstream DNS server     | `127.0.0.1`     |

---

### 📦 Persistence

| Parameter                   | Description              | Value      |
|-----------------------------|--------------------------|------------|
| `persistence.enabled`       | Enable persistent volume | `true`     |
| `persistence.storageClassName` | Storage class         | `standard` |
| `persistence.storageSize`   | Volume size              | `1G`       |

---

### 📈 Resources

```yaml
resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

---

## ❌ Uninstallation

To remove the Pi-hole release:

```bash
helm uninstall pihole
```

---

## ⚠️ Notes

- 🚫 Ensure ports 53, 80, and 443 are not already in use by other services.
- 🔐 Change the default admin password for production use.
- 🧠 Confirm DNS traffic flows correctly in your cluster (especially if you use custom upstreams).

---

## 📚 More Info

For detailed configuration, refer to the [Pi-hole documentation](https://docs.pi-hole.net/).

---

🛡️ Happy Ad-Blocking with Pi-hole + Kubernetes!%