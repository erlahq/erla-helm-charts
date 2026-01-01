# Contributing to ERLA Helm Charts

Thank you for your interest in contributing to ERLA Helm Charts! We welcome contributions from the community.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Chart Guidelines](#chart-guidelines)
- [Testing](#testing)
- [Submitting Changes](#submitting-changes)

## Code of Conduct

We are committed to providing a welcoming and inclusive environment. Please be respectful and professional in all interactions.

## Getting Started

### Prerequisites

- Kubernetes cluster (1.23+) for testing
- Helm 3.8+
- kubectl configured with cluster access
- Git

### Setting Up Your Development Environment

1. Fork the repository on GitHub
2. Clone your fork:
   ```bash
   git clone https://github.com/YOUR_USERNAME/erla-helm-charts.git
   cd erla-helm-charts
   ```
3. Add upstream remote:
   ```bash
   git remote add upstream https://github.com/erlahq/erla-helm-charts.git
   ```

## Development Workflow

### Creating a New Chart

1. Create a new directory under `charts/`:
   ```bash
   helm create charts/my-new-chart
   ```

2. Follow the [Chart Guidelines](#chart-guidelines) below

3. Add a comprehensive README.md to your chart directory

4. Test thoroughly

### Updating an Existing Chart

1. Create a feature branch:
   ```bash
   git checkout -b feature/improve-ib-gateway
   ```

2. Make your changes

3. Bump the chart version in `Chart.yaml`:
   - Patch version (x.y.Z) for bug fixes and minor improvements
   - Minor version (x.Y.0) for new features (backward compatible)
   - Major version (X.0.0) for breaking changes

4. Update the CHANGELOG in the chart's README.md

## Chart Guidelines

### Chart Structure

All charts must follow the standard Helm chart structure:

```
charts/my-chart/
├── Chart.yaml           # Chart metadata (required)
├── values.yaml          # Default values (required)
├── README.md           # Documentation (required)
├── .helmignore         # Ignored files (recommended)
├── templates/          # Kubernetes manifests (required)
│   ├── NOTES.txt      # Post-install notes (recommended)
│   ├── _helpers.tpl   # Template helpers (recommended)
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ...
└── templates/tests/    # Chart tests (recommended)
```

### Best Practices

#### Chart.yaml

- Use semantic versioning for `version`
- Include meaningful `description`
- Specify `appVersion` to track upstream application version
- Add `keywords` for discoverability
- Include `maintainers` information
- Add `sources` links to upstream projects

Example:
```yaml
apiVersion: v2
name: my-chart
description: A Helm chart for MyApplication
type: application
version: 1.0.0
appVersion: "2.1.0"
keywords:
  - trading
  - market-data
maintainers:
  - name: ERLA Team
    email: opensource@erlahq.com
sources:
  - https://github.com/upstream/project
```

#### values.yaml

- Provide sensible defaults
- Document every value with comments
- Group related values
- Use camelCase for keys
- Avoid hardcoding values that users might want to change

#### Templates

- Use `_helpers.tpl` for common template functions
- Include resource limits and requests
- Add health checks (liveness/readiness probes)
- Support configurable image repositories and tags
- Use named ports in Services
- Include security contexts
- Support node affinity and tolerations
- Add proper labels and annotations

#### Documentation

Every chart must include a README.md with:

- Overview and features
- Prerequisites
- Installation instructions
- Configuration parameters table
- Usage examples
- Upgrading guide
- Troubleshooting section
- License information

### Security

- Never include secrets or credentials in charts
- Use Kubernetes Secrets for sensitive data
- Run containers as non-root when possible
- Set appropriate security contexts
- Validate user inputs
- Follow principle of least privilege

### Resource Management

- Always define resource requests and limits
- Provide reasonable defaults based on expected workload
- Document resource requirements in README
- Consider autoscaling where appropriate

## Testing

### Linting

Before submitting, lint your charts:

```bash
# Lint a specific chart
helm lint charts/my-chart

# Lint with custom values
helm lint charts/my-chart -f charts/my-chart/values-production.yaml
```

### Template Validation

Verify rendered templates:

```bash
# Render templates
helm template test-release charts/my-chart

# Render with custom values
helm template test-release charts/my-chart -f my-values.yaml

# Dry run install
helm install test-release charts/my-chart --dry-run --debug
```

### Integration Testing

Test installation on a real cluster:

```bash
# Install
helm install test-release charts/my-chart

# Check resources
kubectl get all -l app.kubernetes.io/instance=test-release

# Check logs
kubectl logs -l app.kubernetes.io/instance=test-release

# Cleanup
helm uninstall test-release
```

### Automated Testing

Our CI pipeline automatically:
- Lints all changed charts
- Validates templates
- Tests installation on kind cluster

Ensure your changes pass all checks before submitting.

## Submitting Changes

### Pull Request Process

1. Ensure all tests pass locally

2. Commit your changes:
   ```bash
   git add .
   git commit -m "feat(ib-gateway): add support for custom timeouts"
   ```

   Use conventional commit format:
   - `feat(chart-name):` for new features
   - `fix(chart-name):` for bug fixes
   - `docs(chart-name):` for documentation
   - `refactor(chart-name):` for refactoring
   - `test(chart-name):` for tests
   - `chore(chart-name):` for maintenance

3. Push to your fork:
   ```bash
   git push origin feature/improve-ib-gateway
   ```

4. Open a Pull Request on GitHub

5. Fill out the PR template with:
   - Description of changes
   - Related issue numbers
   - Testing performed
   - Breaking changes (if any)

6. Wait for review and address feedback

### Review Process

- All PRs require at least one approval
- CI checks must pass
- Charts must be properly tested
- Documentation must be updated
- Version must be bumped appropriately

## Versioning

We follow [Semantic Versioning](https://semver.org/):

- **PATCH** (0.0.x): Bug fixes, documentation updates, minor improvements
- **MINOR** (0.x.0): New features, backward compatible changes
- **MAJOR** (x.0.0): Breaking changes, incompatible API changes

When updating a chart:
1. Increment the version in `Chart.yaml`
2. Document changes in the chart's README
3. Note breaking changes clearly

## Questions?

- Open an [Issue](https://github.com/erlahq/erla-helm-charts/issues) for bugs or feature requests
- Start a [Discussion](https://github.com/erlahq/erla-helm-charts/discussions) for questions
- Join our community (link TBD)

## License

By contributing, you agree that your contributions will be licensed under the Apache License 2.0.

Thank you for contributing to ERLA Helm Charts!