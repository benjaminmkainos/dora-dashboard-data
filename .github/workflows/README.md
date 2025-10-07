# GitHub Actions Workflows

This repository contains three GitHub Action pipelines for CI/CD automation using GitVersion for semantic versioning.

## Workflows Overview

### 1. Build and Version (`build-and-version.yml`)

**Purpose**: Performs build tasks and calculates version using GitVersion

**Triggers**:
- Push to `main` or `develop` branches
- Pull requests to `main` or `develop` branches

**Steps**:
1. Checkout code with full git history
2. Install GitVersion
3. Calculate semantic version based on git history
4. Display version information
5. Execute build task
6. Upload build artifacts

**Usage**:
This workflow runs automatically on every push or pull request. The build task placeholder can be replaced with actual build commands (npm, dotnet, maven, etc.).

---

### 2. Create Release (`release.yml`)

**Purpose**: Creates a Git tag and GitHub release using GitVersion

**Triggers**:
- Push to `main` branch
- Manual dispatch (workflow_dispatch)

**Steps**:
1. Checkout code with full git history
2. Install GitVersion
3. Calculate semantic version
4. Create and push Git tag (format: `v{semver}`)
5. Create GitHub Release with version details

**Usage**:
- **Automatic**: Runs on every push to main
- **Manual**: Go to Actions tab → Create Release → Run workflow

**Permissions Required**:
- `contents: write` (automatically provided)

---

### 3. Deploy Release (`deploy.yml`)

**Purpose**: Deploys a tagged release

**Triggers**:
- When a GitHub release is published
- Manual dispatch with tag input

**Steps**:
1. Checkout code at the specified tag
2. Get and validate release information
3. Download release assets (if available)
4. Run pre-deployment checks
5. Execute deployment task
6. Run post-deployment verification
7. Generate deployment summary

**Usage**:
- **Automatic**: Runs when a release is published via the release.yml workflow
- **Manual**: Go to Actions tab → Deploy Release → Run workflow → Enter tag (e.g., v1.0.0)

**Customization**:
Replace the deployment task placeholder with actual deployment commands:
- Kubernetes: `kubectl apply -f k8s/`
- Terraform: `terraform apply`
- Docker: `docker push`
- AWS: `aws deploy ...`
- Server: `scp files to server`

---

## GitVersion Configuration

The `GitVersion.yml` file in the repository root configures semantic versioning behavior:

- **Mode**: ContinuousDelivery
- **Main branch**: Patch increment, no tag
- **Develop branch**: Minor increment, `alpha` tag
- **Feature branches**: Minor increment, `beta` tag
- **Hotfix branches**: Patch increment, `hotfix` tag
- **Release branches**: No increment, `rc` tag

### Version Examples:
- Main: `1.0.1`, `1.0.2`, `1.0.3`
- Develop: `1.1.0-alpha.1`, `1.1.0-alpha.2`
- Feature: `1.1.0-beta.1`, `1.1.0-beta.2`

---

## Workflow Execution Order

For a typical release cycle:

1. **Development**:
   - Push to `develop` or feature branches
   - `build-and-version.yml` runs automatically

2. **Release**:
   - Merge to `main` or manually trigger
   - `release.yml` creates a tag and GitHub release

3. **Deployment**:
   - Release publication triggers `deploy.yml`
   - Or manually trigger with a specific tag

---

## Required GitHub Secrets

The workflows use `GITHUB_TOKEN` which is automatically provided by GitHub Actions. No additional secrets configuration is needed unless you customize the workflows.

---

## Customization Guide

### Build Task (build-and-version.yml)
Replace line 44-49 with your actual build commands:
```yaml
- name: Build task
  run: |
    npm install
    npm run build
    npm test
```

### Deploy Task (deploy.yml)
Replace line 59-70 with your actual deployment commands:
```yaml
- name: Deploy task
  run: |
    kubectl apply -f k8s/deployment.yml
    kubectl rollout status deployment/my-app
```

### Artifacts
Update the artifacts path in `build-and-version.yml` (line 55-56):
```yaml
path: |
  dist/
  build/
```

---

## Monitoring and Troubleshooting

- View workflow runs: Go to the "Actions" tab in your repository
- Each workflow run shows detailed logs for each step
- Failed workflows will send notifications to repository watchers
- Use the deployment summary in `deploy.yml` to track deployments

---

## Best Practices

1. **Always test in a feature branch first** before merging to main
2. **Review the calculated version** in build logs before releasing
3. **Use manual dispatch** for release.yml to control when releases are created
4. **Verify deployments** using the post-deployment verification step
5. **Keep GitVersion.yml** up to date with your versioning strategy

---

## DORA Metrics Integration

These workflows are designed to support DORA (DevOps Research and Assessment) metrics:

- **Deployment Frequency**: Tracked via `deploy.yml` executions
- **Lead Time for Changes**: Measured from commit to deployment
- **Change Failure Rate**: Monitor deployment success/failure in Actions
- **Time to Restore**: Track time between failure and successful deployment

Use GitHub Actions metrics and deployment logs to calculate these DORA metrics for your dashboard.
