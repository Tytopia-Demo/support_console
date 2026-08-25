# GitHub Actions Runner Selection Guidelines

This document provides standardized guidelines for selecting appropriate GitHub Actions runners based on workflow resource requirements. Following these guidelines ensures optimal cost efficiency and performance.

## Table of Contents

- [Runner Types](#runner-types)
- [Resource Tier Classification](#resource-tier-classification)
- [Workflow Pattern Guidelines](#workflow-pattern-guidelines)
- [Best Practices](#best-practices)
- [Monitoring and Optimization](#monitoring-and-optimization)

## Runner Types

### GitHub-Hosted Runners

| Runner Label | vCPU | RAM | Storage | Use Case |
|--------------|------|-----|---------|----------|
| `ubuntu-latest` | 2 | 7 GB | 14 GB SSD | Default - suitable for most workflows |
| `ubuntu-latest-4-core` | 4 | 16 GB | 14 GB SSD | Medium workloads with parallel processing |
| `ubuntu-latest-8-core` | 8 | 32 GB | 14 GB SSD | Heavy computation or build tasks |
| `ubuntu-latest-16-core` | 16 | 64 GB | 14 GB SSD | Intensive parallel builds or large compilations |

_Note: Similar options exist for `windows-latest` and `macos-latest`._

### Self-Hosted Runners

If using self-hosted runners, use labels that clearly indicate resource specifications:
- `self-hosted-small` (2 vCPU, 4-8 GB RAM)
- `self-hosted-medium` (4 vCPU, 8-16 GB RAM)
- `self-hosted-large` (8+ vCPU, 16+ GB RAM)

## Resource Tier Classification

### Small Tier - Use `ubuntu-latest` (2-core)

**Characteristics:**
- Execution time: < 10 minutes
- Memory usage: < 4 GB
- CPU utilization: < 50% average
- Primarily I/O-bound operations

**Workflow Types:**
- Linting and code formatting (ESLint, Prettier, RuboCop)
- Security scanning (dependency checks, SAST tools)
- Unit tests (small test suites < 100 tests)
- Documentation generation
- Simple build tasks (static sites, small projects)

**Example Workflows:**
- Dependency vulnerability scanning (Frogbot, Dependabot)
- Code quality checks
- Pre-commit hooks validation
- License compliance checks

### Medium Tier - Use `ubuntu-latest-4-core`

**Characteristics:**
- Execution time: 10-30 minutes
- Memory usage: 4-12 GB
- CPU utilization: 50-80% average
- Mix of I/O and CPU operations

**Workflow Types:**
- Integration tests (moderate test suites 100-500 tests)
- Application builds (medium-sized projects)
- Docker image builds (multi-stage builds)
- End-to-end tests (small number of browsers)
- Moderate parallel processing (matrix builds with 2-4 jobs)

**Example Workflows:**
- Rails application with asset compilation
- Node.js builds with webpack/rollup
- Python application with multiple dependencies
- Container builds with caching

### Large Tier - Use `ubuntu-latest-8-core` or higher

**Characteristics:**
- Execution time: > 30 minutes
- Memory usage: > 12 GB
- CPU utilization: > 80% sustained
- CPU-intensive operations

**Workflow Types:**
- Large test suites (> 500 tests)
- Complex compilation (C++, Rust, large Java projects)
- Multiple parallel matrix builds (8+ concurrent jobs)
- Heavy Docker builds (multiple large images)
- Data processing or analysis workflows
- Cross-platform builds requiring emulation

**Example Workflows:**
- Full system tests with multiple services
- Monorepo builds affecting multiple services
- Machine learning model training or inference
- Large-scale integration tests

## Workflow Pattern Guidelines

### Security Scanning Workflows

**Recommended Configuration:**
```yaml
jobs:
  security-scan:
    runs-on: ubuntu-latest  # 2-core sufficient for most scanning tools
    timeout-minutes: 15     # Typical duration: 3-10 minutes
```

**Rationale:** Security scanning tools (SAST, dependency checkers, container scanners) are typically I/O-bound, spending most time downloading databases and dependencies. The 2-core runner provides adequate performance at optimal cost.

**Tools in this category:**
- Frogbot, Snyk, Dependabot
- Trivy, Grype
- SonarQube, CodeQL
- Bandit, Brakeman, Gosec

### Test Workflows

**Small Test Suites:**
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 10
```

**Medium Test Suites (with parallelization):**
```yaml
jobs:
  test:
    runs-on: ubuntu-latest-4-core
    timeout-minutes: 20
    strategy:
      matrix:
        test-group: [unit, integration, functional]
```

**Large Test Suites:**
```yaml
jobs:
  test:
    runs-on: ubuntu-latest-8-core
    timeout-minutes: 45
    strategy:
      max-parallel: 8
      matrix:
        shard: [1, 2, 3, 4, 5, 6, 7, 8]
```

### Build and Deployment Workflows

**Simple Builds:**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 15
```

**Asset-Heavy Builds:**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest-4-core
    timeout-minutes: 30
```

**Multi-Platform or Complex Builds:**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest-8-core
    timeout-minutes: 60
```

### Lint and Format Workflows

**Recommended Configuration:**
```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    timeout-minutes: 5
```

**Rationale:** Linting is fast and CPU-light. Use the smallest runner to minimize costs.

## Best Practices

### 1. Always Set Timeout Values

Every job should have a `timeout-minutes` value to prevent runaway processes:

```yaml
jobs:
  my-job:
    timeout-minutes: 30  # Adjust based on typical duration + buffer
```

**Guidelines:**
- Set timeout to 150-200% of typical execution time
- Minimum timeout: 5 minutes (for fast jobs)
- Maximum timeout: 360 minutes (6 hours, GitHub maximum)

### 2. Implement Resource Monitoring

Add resource monitoring steps to track actual usage:

```yaml
steps:
  - name: Log resource usage (start)
    run: |
      echo "=== System Resources (Start) ==="
      echo "CPU cores: $(nproc)"
      echo "Memory: $(free -h)"
      echo "Disk: $(df -h /)"
      echo "Load: $(uptime)"

  # Your workflow steps here

  - name: Log resource usage (end)
    if: always()
    run: |
      echo "=== System Resources (End) ==="
      echo "Memory: $(free -h)"
      echo "Load: $(uptime)"
```

### 3. Optimize Job Parallelization

**Use Matrix Builds Strategically:**
- Only parallelize when jobs are independent
- Consider if parallel execution actually reduces total time
- Balance parallelization with runner availability

**Example - Good Use:**
```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node-version: [14, 16, 18]
```

**Example - Avoid:**
```yaml
strategy:
  matrix:
    branch: ["master"]  # Single value provides no benefit
```

### 4. Use Appropriate Concurrency Controls

Prevent unnecessary parallel runs:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true  # Cancel old runs when new ones start
```

### 5. Leverage Caching

Reduce execution time and resource usage:

```yaml
- uses: actions/cache@v3
  with:
    path: |
      ~/.cache
      node_modules
    key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
```

### 6. Consider Job Dependencies

Structure jobs to run in parallel where possible:

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest

  test:
    runs-on: ubuntu-latest

  build:
    needs: [lint, test]  # Only run after dependencies complete
    runs-on: ubuntu-latest-4-core
```

## Monitoring and Optimization

### Initial Assessment

When creating a new workflow:

1. Start with `ubuntu-latest` (2-core) as default
2. Add resource monitoring steps
3. Run the workflow 3-5 times
4. Analyze the logs for:
   - Actual execution time
   - Memory usage peaks
   - CPU utilization
   - Disk I/O patterns

### Optimization Indicators

**Upgrade to larger runner if:**
- ✓ Execution time > 20 minutes on 2-core
- ✓ Memory usage consistently > 5 GB
- ✓ CPU utilization > 80% sustained
- ✓ Job timing out frequently

**Downgrade to smaller runner if:**
- ✓ Execution time < 5 minutes
- ✓ Memory usage < 2 GB
- ✓ CPU utilization < 30% average
- ✓ Workflow is I/O-bound (waiting on network/disk)

### Regular Review

Conduct quarterly reviews of:
- Workflow execution times (trends over time)
- Runner costs per workflow
- Failed/timed-out jobs
- Resource utilization patterns

### Cost Optimization Checklist

- [ ] All workflows have appropriate `runs-on` specifications
- [ ] All jobs have `timeout-minutes` configured
- [ ] Resource monitoring is implemented
- [ ] Unnecessary workflows are disabled/removed
- [ ] Caching is used where applicable
- [ ] Job dependencies are optimized for parallelization
- [ ] Concurrency controls prevent redundant runs
- [ ] Matrix strategies are used effectively
- [ ] Workflows use appropriate runner sizes for workload

## Examples

### Example 1: Frogbot Security Scan

```yaml
name: "Frogbot Security Scan"
on:
  pull_request_target:
    types: [opened, synchronize]
  schedule:
    - cron: "0 0 * * *"

jobs:
  frogbot-scan:
    runs-on: ubuntu-latest  # 2-core sufficient for security scanning
    timeout-minutes: 15     # Typical: 3-10 minutes
    steps:
      - name: Log resource usage (start)
        run: |
          echo "CPU cores: $(nproc)"
          echo "Memory: $(free -h)"

      - uses: jfrog/frogbot@v2
        env:
          JF_URL: ${{ secrets.JF_URL }}
          JF_ACCESS_TOKEN: ${{ secrets.JF_ACCESS_TOKEN }}

      - name: Log resource usage (end)
        if: always()
        run: echo "Memory: $(free -h)"
```

### Example 2: Rails Test Suite

```yaml
name: "Rails Tests"
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest-4-core  # Medium runner for test suite
    timeout-minutes: 30
    services:
      postgres:
        image: postgres:13
    strategy:
      matrix:
        ruby-version: ['2.7', '3.0', '3.1']
    steps:
      - uses: actions/checkout@v3
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: ${{ matrix.ruby-version }}
          bundler-cache: true
      - run: bundle exec rake test
```

### Example 3: Multi-Platform Build

```yaml
name: "Build Application"
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest-8-core  # Large runner for complex builds
    timeout-minutes: 60
    strategy:
      matrix:
        target: [linux-amd64, linux-arm64, darwin-amd64, darwin-arm64, windows-amd64]
    steps:
      - uses: actions/checkout@v3
      - name: Build for ${{ matrix.target }}
        run: make build-${{ matrix.target }}
```

## Questions or Updates

For questions about runner selection or to suggest updates to these guidelines:
- Open an issue in this repository
- Contact the DevOps team
- Review GitHub's official [runner specifications](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners)

## Changelog

- **2024-01**: Initial guidelines created
  - Established runner tier classifications
  - Documented workflow patterns
  - Added monitoring and optimization best practices
