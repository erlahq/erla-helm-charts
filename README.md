# ERLA Helm Charts

Official Helm charts for ERLA's infrastructure and open-source tools.

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

## Overview

This repository contains Helm charts for deploying infrastructure components on Kubernetes.

## Available Charts

| Chart | Description | Status |
|-------|-------------|--------|
| [ib-gateway](./charts/ib-gateway) | Interactive Brokers Gateway for automated trading via TWS API | Stable |

## Usage

### Add Helm Repository

```bash
helm repo add erla https://erlahq.github.io/erla-helm-charts
helm repo update
```

### Install a Chart

```bash
# Install IB Gateway
helm install ib-gateway erla/ib-gateway \
  --set credentials.secretName=ibkr-credentials \
  --set ibgateway.tradingMode=paper
```

### Development Installation

Clone this repository and install from source:

```bash
git clone https://github.com/erlahq/erla-helm-charts.git
cd erla-helm-charts

# Install from local chart
helm install ib-gateway ./charts/ib-gateway -f ./charts/ib-gateway/values.yaml
```

## Contributing

We welcome contributions! Please see our [contributing guidelines](CONTRIBUTING.md) for details.

### Reporting Issues

If you encounter any issues or have feature requests, please [open an issue](https://github.com/erlahq/erla-helm-charts/issues).

### Pull Requests

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Test thoroughly with `helm lint` and `helm template`
5. Commit your changes (`git commit -m 'Add amazing feature'`)
6. Push to the branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

## License

This repository is licensed under the Apache License 2.0. See [LICENSE](LICENSE) for details.

Individual charts may reference or depend on third-party software with different licenses. Please refer to each chart's README for specific licensing information.


## Support

- **Documentation**: See individual chart READMEs
- **Issues**: [GitHub Issues](https://github.com/erlahq/erla-helm-charts/issues)
- **Discussions**: [GitHub Discussions](https://github.com/erlahq/erla-helm-charts/discussions)

## Related Projects

- [gnzsnz/ib-gateway-docker](https://github.com/gnzsnz/ib-gateway-docker) - Docker image for Interactive Brokers Gateway
- [IBC](https://github.com/IbcAlpha/IBC) - Interactive Brokers Controller