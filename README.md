# Node.js Builder & Deployer Action

A reusable GitHub Action for building, testing, auditing, and deploying Node.js applications to Kubernetes with multi-platform Docker support.

## Features

- 🧪 Automated unit testing with npm test
- 🔒 Security vulnerability scanning with npm audit
- 🐳 Multi-platform Docker builds (amd64/arm64)
- 📦 Pushes to GitHub Container Registry
- ☸️ Kubernetes deployment integration
- 🔐 Vault integration for secure credential management
- 🏷️ Automatic tagging (staging/latest)
- 🚀 Separate staging and production deployments

## Usage

### Basic Example

```yaml
name: Build, Test & Deploy

on:
  push:
    branches: [ "*" ]
  pull_request:
    branches: [ "*" ]

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build, Test and Deploy
        uses: cfindlayisme/nodejs-builder@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          repository: ${{ github.repository }}
          ref: ${{ github.ref }}
          vault-addr: ${{ secrets.VAULT_ADDR }}
          vault-role-id: ${{ secrets.VAULT_ROLE_ID }}
          vault-secret-id: ${{ secrets.VAULT_SECRET_ID }}
          deployment-name: 'my-app'
```

### Advanced Example with Custom Configuration

```yaml
- name: Build, Test and Deploy
  uses: cfindlayisme/nodejs-builder@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    repository: ${{ github.repository }}
    ref: ${{ github.ref }}
    vault-addr: ${{ secrets.VAULT_ADDR }}
    vault-role-id: ${{ secrets.VAULT_ROLE_ID }}
    vault-secret-id: ${{ secrets.VAULT_SECRET_ID }}
    deployment-name: 'my-app'
    node-version: '20'
    audit-level: 'moderate'
    staging-enabled: 'true'
    production-enabled: 'true'
    staging-namespace: 'staging'
    production-namespace: 'production'
    docker-platforms: 'linux/amd64,linux/arm64'
```

## Inputs

### Required Inputs

| Input | Description |
|-------|-------------|
| `github-token` | GitHub token for container registry |
| `repository` | GitHub repository (owner/name) |
| `ref` | Git reference being built |
| `vault-addr` | Vault server address |
| `vault-role-id` | Vault AppRole role ID |
| `vault-secret-id` | Vault AppRole secret ID |
| `deployment-name` | Kubernetes deployment name |

### Optional Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `node-version` | Node.js version to use | `22` |
| `vault-kubeconfig-path` | Vault path to kubeconfig secret | `kv/data/pipeline/k3s` |
| `staging-enabled` | Enable staging deployment | `true` |
| `production-enabled` | Enable production deployment | `true` |
| `staging-namespace` | Kubernetes namespace for staging | `staging` |
| `production-namespace` | Kubernetes namespace for production | `production` |
| `docker-platforms` | Docker platforms to build for | `linux/amd64,linux/arm64` |
| `dockerfile-path` | Path to Dockerfile | `./Dockerfile` |
| `docker-context` | Docker build context | `.` |
| `audit-level` | npm audit severity level | `high` |
| `run-tests` | Whether to run npm test | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `audit-result` | Security audit result |
| `test-result` | Unit test result |
| `build-result` | Docker build result |
| `deploy-result` | Deployment result |

## Workflow Behavior

### All Branches
1. **Install Dependencies**: Runs `npm install`
2. **Security Audit**: Runs `npm audit` at the specified level
3. **Unit Tests**: Runs `npm test` (if enabled)
4. **Docker Build**: Builds and pushes `:staging` tag
5. **Deploy to Staging**: Restarts staging deployment (if enabled)

### Main Branch Only
- Additionally builds and pushes `:latest` tag
- Additionally deploys to production namespace (if enabled)

## Deployment Strategy

- **Staging**: Deploys on all builds when `staging-enabled: 'true'`
- **Production**: Only deploys on main branch when `production-enabled: 'true'` and `ref == 'refs/heads/main'`
- Both use `kubectl rollout restart` to trigger a deployment update

## Prerequisites

### Node.js Project Requirements
- `package.json` with dependencies defined
- `npm test` script configured (if `run-tests: 'true'`)
- Dockerfile for containerization

### Vault Setup
1. HashiCorp Vault server with AppRole authentication enabled
2. Kubeconfig stored in Vault at the specified path
3. GitHub secrets configured:
   - `VAULT_ADDR`: Your Vault server URL
   - `VAULT_ROLE_ID`: AppRole role ID
   - `VAULT_SECRET_ID`: AppRole secret ID

### Kubernetes
- Kubernetes cluster configured
- Namespaces created (`staging`, `production`, or custom)
- Deployment resources defined

## Security Audit Levels

The `audit-level` input accepts the following values:
- `low`: Fail on low severity and above
- `moderate`: Fail on moderate severity and above
- `high`: Fail on high severity and above (default)
- `critical`: Fail only on critical severity

## Example Project Structure

```
my-nodejs-project/
├── .github/
│   └── workflows/
│       └── build-deploy.yml
├── Dockerfile
├── package.json
├── package-lock.json
├── src/
│   └── ...
└── test/
    └── ...
```

## License

LGPL v3

## Author

cfindlayisme
