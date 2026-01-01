# IB Gateway Helm Chart

Deploy Interactive Brokers Gateway on Kubernetes for automated trading via the TWS API.

## Overview

This Helm chart deploys the Interactive Brokers Gateway, enabling automated trading applications to connect to IBKR's trading platform through a stable, containerized environment. The chart is based on the excellent [gnzsnz/ib-gateway-docker](https://github.com/gnzsnz/ib-gateway-docker) project.

## Features

- Automated deployment of IB Gateway with TWS API access
- Support for both paper and live trading modes
- VNC access for two-factor authentication (2FA) handling
- Health checks and liveness probes for reliability
- Configurable resource limits for production workloads
- Node affinity and tolerations for dedicated trading infrastructure

## Prerequisites

- Kubernetes 1.23+
- Helm 3.8+
- Interactive Brokers account (paper or live)
- Kubernetes secret containing IBKR credentials

## Installation

### Create IBKR Credentials Secret

First, create a Kubernetes secret with your Interactive Brokers credentials:

```bash
kubectl create secret generic ibkr-credentials \
  --from-literal=username='YOUR_IBKR_USERNAME' \
  --from-literal=password='YOUR_IBKR_PASSWORD'
```

### Install the Chart

```bash
# Add the ERLA Helm repository
helm repo add erla https://erlahq.github.io/erla-helm-charts
helm repo update

# Install for paper trading
helm install ib-gateway erla/ib-gateway \
  --set credentials.secretName=ibkr-credentials \
  --set ibgateway.tradingMode=paper

# Install for live trading
helm install ib-gateway erla/ib-gateway \
  --set credentials.secretName=ibkr-credentials \
  --set ibgateway.tradingMode=live
```

### Install from Source

```bash
git clone https://github.com/erlahq/erla-helm-charts.git
cd erla-helm-charts

helm install ib-gateway ./charts/ib-gateway \
  --set credentials.secretName=ibkr-credentials
```

## Configuration

### Key Parameters

| Parameter | Description | Default                     |
|-----------|-------------|-----------------------------|
| `ibgateway.tradingMode` | Trading mode: `paper` or `live` | `paper`                     |
| `ibgateway.twsPort` | TWS API port (4001 for live, 4002 for paper) | `4002`                      |
| `ibgateway.vncPassword` | VNC password for remote access | `changeme`                  |
| `ibgateway.javaHeapSize` | JVM heap size in MB | `2048`                      |
| `credentials.secretName` | Name of Kubernetes secret with IBKR credentials | `ibkr-credentials`          |
| `image.repository` | IB Gateway Docker image | `ghcr.io/gnzsnz/ib-gateway` |
| `image.tag` | Image tag | `latest`                    |
| `resources.requests.cpu` | CPU request | `3000m`                     |
| `resources.requests.memory` | Memory request | `4Gi`                       |
| `resources.limits.cpu` | CPU limit | `4000m`                     |
| `resources.limits.memory` | Memory limit | `6Gi`                       |

### Example: Custom Configuration

Create a `values.yaml` file:

```yaml
# Custom values for production live trading
ibgateway:
  tradingMode: live
  twsPort: 4001
  javaHeapSize: 3072
  vncPassword: "changeme"

credentials:
  secretName: ibkr-prod-credentials

resources:
  requests:
    cpu: 4000m
    memory: 6Gi
  limits:
    cpu: 6000m
    memory: 8Gi

# Schedule on specific nodes
nodeSelector:
  workload: trading
  zone: us-east-1a

# Production-grade affinity
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: workload
              operator: In
              values:
                - trading
```

Install with custom values:

```bash
helm install ib-gateway erla/ib-gateway -f values.yaml
```

## Usage

### Connecting Your Trading Application

Once deployed, your trading applications can connect to IB Gateway via the Kubernetes service:

**Service DNS Name:** `ib-gateway.your-namespace.svc.cluster.local`

**Ports:**
- TWS API: 4002 (paper) or 4001 (live)
- VNC: 5900

**Example connection in your trading app:**

```python
from ibapi.client import EClient
from ibapi.wrapper import EWrapper

class IBApp(EWrapper, EClient):
    def __init__(self):
        EClient.__init__(self, self)

# Connect to IB Gateway service
app = IBApp()
app.connect("ib-gateway", 4002, clientId=1)  # Paper trading
```

### Accessing VNC for 2FA

IB Gateway requires VNC access for handling two-factor authentication. Use port-forwarding to access VNC:

```bash
# Forward VNC port to localhost
kubectl port-forward service/ib-gateway 5900:5900

# Connect with VNC client
# macOS: open vnc://localhost:5900
# Linux: vncviewer localhost:5900
# Windows: Use TightVNC or RealVNC
# Password: Value from ibgateway.vncPassword (default: changeme)
```

### Health Monitoring

The chart includes health checks:

```bash
# Check pod status
kubectl get pods -l app.kubernetes.io/name=ib-gateway

# View logs
kubectl logs -l app.kubernetes.io/name=ib-gateway -f

# Check service endpoints
kubectl get endpoints ib-gateway
```

## Upgrading

```bash
# Update repository
helm repo update

# Upgrade release
helm upgrade ib-gateway erla/ib-gateway \
  --set credentials.secretName=ibkr-credentials
```

## Uninstalling

```bash
helm uninstall ib-gateway
```

## Troubleshooting

### Gateway Not Starting

**Check logs:**
```bash
kubectl logs -l app.kubernetes.io/name=ib-gateway --tail=100
```

**Common issues:**
- Incorrect IBKR credentials
- Two-factor authentication not completed (access via VNC)
- Port conflicts

### Connection Refused from Trading App

**Verify service:**
```bash
kubectl get service ib-gateway
kubectl describe service ib-gateway
```

**Test connectivity:**
```bash
kubectl run -it --rm debug --image=busybox --restart=Never -- telnet ib-gateway 4002
```

### Health Check Failures

**Check probe status:**
```bash
kubectl describe pod -l app.kubernetes.io/name=ib-gateway
```

The liveness probe waits 120 seconds for IB Gateway to fully start. Increase `livenessProbe.initialDelaySeconds` if needed.

## Architecture

### Deployment Strategy

The chart uses `Recreate` strategy to ensure only one IB Gateway instance runs at a time, as IBKR limits concurrent connections per account.

### Resource Allocation

Default resources:

- **CPU Request:** 3000m (3 cores) for stable TWS API
- **Memory Request:** 4Gi (2GB JVM heap + 2GB overhead)
- **CPU Limit:** 4000m (can burst for market data spikes)
- **Memory Limit:** 6Gi (hard limit to prevent leaks)

Adjust based on your workload and cluster capacity.

### Network Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 4002 | TCP | TWS API (Paper Trading) |
| 4001 | TCP | TWS API (Live Trading) |
| 5900 | TCP | VNC Server (Remote Access) |

## Security Considerations

### Credentials Management

- Always use Kubernetes secrets for IBKR credentials
- Never commit credentials to version control
- Rotate VNC password from default value
- Use RBAC to restrict secret access

### Network Security

- IB Gateway service is ClusterIP by default (not exposed externally)
- Trading applications should run in the same cluster
- For external access, use secure tunnels (VPN, port-forward, or ingress with mTLS)

### Pod Security

- Runs as root user (UID 0) by default (required by the IB Gateway Docker image)
- Configure `podSecurityContext` and `securityContext` as needed
- Consider using Pod Security Standards/Admission

## Upstream Project

This chart deploys the Docker image from [gnzsnz/ib-gateway-docker](https://github.com/gnzsnz/ib-gateway-docker), which is licensed under the MIT License.

**Credits:**
- Docker Image: [gnzsnz/ib-gateway-docker](https://github.com/gnzsnz/ib-gateway-docker)
- IBC (IB Controller): [IbcAlpha/IBC](https://github.com/IbcAlpha/IBC)

## Contributing

Contributions are welcome! Please see the main [repository README](../../README.md) for contribution guidelines.

## License

This Helm chart is licensed under the Apache License 2.0. See [LICENSE](../../LICENSE) for details.

The upstream Docker image (gnzsnz/ib-gateway-docker) is licensed under the MIT License.

## Support

- **Issues:** [GitHub Issues](https://github.com/erlahq/erla-helm-charts/issues)
- **Discussions:** [GitHub Discussions](https://github.com/erlahq/erla-helm-charts/discussions)
- **Upstream Issues:** [gnzsnz/ib-gateway-docker Issues](https://github.com/gnzsnz/ib-gateway-docker/issues)
