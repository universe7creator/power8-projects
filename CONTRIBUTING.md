# Contributing to Power8 Projects

Thank you for your interest in contributing to Power8 Projects! This guide will help you get started with contributing to our POWER8/POWER9 optimized Linux distribution and related tools.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Environment](#development-environment)
- [How to Contribute](#how-to-contribute)
- [Coding Standards](#coding-standards)
- [Testing](#testing)
- [Documentation](#documentation)
- [Commit Guidelines](#commit-guidelines)
- [Pull Request Process](#pull-request-process)

## Code of Conduct

This project and everyone participating in it is governed by our commitment to:

- Be respectful and inclusive
- Welcome newcomers and help them learn
- Focus on constructive feedback
- Respect different viewpoints and experiences

## Getting Started

### Prerequisites

To contribute to Power8 Projects, you'll need:

- **Git** (2.20+)
- **Bash** (4.0+) or compatible shell
- **Docker** (optional, for containerized builds)
- **QEMU** (optional, for POWER8 emulation)

### Hardware Requirements

While physical POWER8/POWER9 hardware is ideal, you can contribute using:

- **Physical Hardware**: IBM Power Systems (S822L, S924, AC922, etc.)
- **Cloud Instances**: IBM Cloud, AWS, or Azure POWER instances
- **QEMU Emulation**: ppc64le emulation for development and testing

### Clone the Repository

```bash
git clone https://github.com/Scottcjn/power8-projects.git
cd power8-projects
```

## Development Environment

### Setting Up QEMU for POWER8 Development

If you don't have physical POWER8 hardware, you can use QEMU for development:

```bash
# Install QEMU (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install qemu-system-ppc qemu-utils

# Download a POWER8 VM image
wget http://50.28.86.153/powerelyan/powerelyan-server-1.0-ppc64le.iso

# Create a virtual disk
qemu-img create -f qcow2 power8-dev.qcow2 20G

# Boot the VM
qemu-system-ppc64 \
  -machine pseries \
  -cpu POWER8 \
  -m 4096 \
  -drive file=power8-dev.qcow2,format=qcow2 \
  -cdrom powerelyan-server-1.0-ppc64le.iso \
  -netdev user,id=net0,hostfwd=tcp::2222-:22 \
  -device virtio-net-pci,netdev=net0 \
  -nographic
```

Default login: `powerelyan` / `powerelyan`

### Cloud Development Environment

For cloud-based POWER8 development:

```bash
# IBM Cloud - Create a POWER virtual server
ibmcloud sl vs create \
  --hostname=power8-dev \
  --domain=example.com \
  --cpu=4 \
  --memory=8192 \
  --os=UBUNTU_20_64 \
  --datacenter=dal13 \
  --flavor=A1_4x8

# AWS - Launch POWER instance
aws ec2 run-instances \
  --image-id ami-xxxxxxxx \
  --instance-type a1.xlarge \
  --key-name my-key
```

## How to Contribute

### Reporting Bugs

Before creating a bug report:

1. Check existing issues to avoid duplicates
2. Use the latest version to verify the bug still exists
3. Collect information about the bug

When reporting bugs, include:

- **Power8 Projects version** (e.g., 1.0.0)
- **Hardware/Environment** (physical POWER8, QEMU, cloud instance)
- **Steps to reproduce**
- **Expected behavior**
- **Actual behavior**
- **Logs or error messages**

### Suggesting Enhancements

Enhancement suggestions are tracked as GitHub issues. Include:

- **Use case**: Why is this enhancement needed?
- **Proposed solution**: How should it work?
- **Alternatives**: What else was considered?
- **Additional context**: Screenshots, mockups, references

### Contributing Code

1. **Fork the repository**
2. **Create a branch**: `git checkout -b feature/your-feature-name`
3. **Make your changes**
4. **Test your changes** (see Testing section)
5. **Commit your changes** (see Commit Guidelines)
6. **Push to your fork**: `git push origin feature/your-feature-name`
7. **Open a Pull Request**

## Coding Standards

### Shell Scripts

Power8 Projects uses shell scripts extensively for build automation:

```bash
#!/bin/bash
set -euo pipefail

# Use descriptive variable names
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly BUILD_DIR="${SCRIPT_DIR}/build"

# Functions should be lowercase with underscores
build_package() {
    local package_name="$1"
    local output_dir="$2"
    
    echo "Building ${package_name}..."
    # Implementation
}

# Main execution
main() {
    build_package "powerelyan-server" "./output"
}

main "$@"
```

### Build Script Guidelines

- Use `set -euo pipefail` for strict error handling
- Quote all variables: `"${var}"` not `${var}`
- Use `readonly` for constants
- Provide informative error messages
- Support `--help` and `--version` flags

### POWER8-Specific Optimizations

When contributing code that runs on POWER8:

```bash
# Detect POWER8/POWER9 architecture
if grep -q "POWER8" /proc/cpuinfo 2>/dev/null; then
    ARCH_OPTS="-mcpu=power8 -mtune=power8"
elif grep -q "POWER9" /proc/cpuinfo 2>/dev/null; then
    ARCH_OPTS="-mcpu=power9 -mtune=power9"
fi

# Use AltiVec/VSX when available
if grep -q "altivec" /proc/cpuinfo 2>/dev/null; then
    ARCH_OPTS="${ARCH_OPTS} -maltivec -mvsx"
fi
```

### Package Management

For package list contributions:

```bash
# packages-server.txt format
# One package per line
# Comments start with #
# Groups separated by blank lines

# Core system
glibc
systemd

# Development
gcc
git
make
```

## Testing

### Local Testing

Test your changes before submitting:

```bash
# Validate shell scripts
shellcheck build_powerelyan.sh

# Test package lists
./scripts/validate-packages.sh packages-server.txt

# Build test ISO (requires POWER8 or QEMU)
./build_powerelyan.sh --edition server --test
```

### Testing on POWER8 Hardware

If you have access to POWER8 hardware:

```bash
# Run full test suite
./scripts/run-tests.sh --hardware power8

# Test specific component
./scripts/test-component.sh --component miner
```

### Testing with QEMU

For QEMU-based testing:

```bash
# Start QEMU VM
./scripts/start-qemu.sh --image power8-dev.qcow2

# Run tests in VM
ssh -p 2222 powerelyan@localhost './run-tests.sh'
```

## Documentation

### README Updates

When adding features, update relevant documentation:

- `README.md` - Main project overview
- `POWERELYAN_ROADMAP.md` - Feature roadmap
- `BCOS.md` - Blockchain/Miner documentation
- Package-specific docs in `docs/`

### Code Comments

Document complex logic:

```bash
# Vec_Perm Non-Bijunctive Collapse
# Optimizes AI attention mechanisms using POWER8's vector permutation
# instruction for non-bijective transformations
vec_perm_optimization() {
    local input_data="$1"
    # Implementation details...
}
```

## Commit Guidelines

### Commit Message Format

```
type(scope): subject

body (optional)

footer (optional)
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, semicolons, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Build process or auxiliary tool changes

**Examples:**

```
feat(miner): add mftb hardware entropy support

Implements true randomness extraction from POWER8 timebase register
for cryptographic operations in the RustChain miner.

Closes #123
```

```
fix(build): resolve package dependency order

Ensures glibc is installed before packages that depend on it.
```

```
docs(readme): update QEMU setup instructions

Adds detailed steps for setting up POWER8 development environment
using QEMU on x86_64 hosts.
```

## Pull Request Process

1. **Update documentation** for any changed functionality
2. **Add tests** if applicable
3. **Ensure all tests pass**
4. **Update CHANGELOG.md** with your changes
5. **Request review** from maintainers
6. **Address feedback** promptly
