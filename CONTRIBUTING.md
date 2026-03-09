# Contributing to Cortex Linux

Thank you for your interest in contributing to Cortex Linux! This document provides guidelines for contributing to this AI-first Linux distribution.

---

## Code of Conduct

All contributors must abide by our [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md). Please read it before contributing.

---

## How Can I Contribute?

### Reporting Bugs

Before creating a bug report, please check:
- The [documentation](https://github.com/your-org/cortex-linux/docs)
- Existing [issues](https://github.com/your-org/cortex-linux/issues)

When creating a bug report, include:
- A clear, descriptive title
- Steps to reproduce the issue
- Expected behavior
- Actual behavior
- Environment details (OS, version, hardware)
- Screenshots if applicable

### Suggesting Enhancements

Enhancement suggestions are welcome! When submitting:
- Use a clear, descriptive title
- Explain the problem this solves
- Describe the proposed solution
- Include any alternatives you've considered
- Explain why this enhancement would be useful to most users

### Contributing Code

1. **Fork the repository**
   ```bash
   git clone https://github.com/your-org/cortex-linux.git
   cd cortex-linux
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes**
   - Follow the existing code style
   - Add tests for new functionality
   - Update documentation as needed

3. **Test your changes**
   ```bash
   # Run linter
   make lint
   
   # Run tests
   make test
   ```

4. **Commit your changes**
   ```bash
   git commit -m "feat: Add new feature"
   ```

5. **Push and create a pull request**
   ```bash
   git push origin feature/your-feature-name
   ```

---

## Development Process

### Branching Strategy

- `main` - Stable, production-ready code
- `develop` - Integration branch for features
- `feature/*` - New features (derived from `develop`)
- `bugfix/*` - Bug fixes (derived from `main` or `develop`)
- `release/*` - Release preparation (derived from `develop`)

### Pull Request Process

1. Fill out the PR template
2. Ensure all checks pass
3. Request review from maintainers
4. Address review comments promptly
5. Squash commits if requested

---

## Coding Standards

### Bash Scripting
- Use `set -euo pipefail` at the top of scripts
- Use descriptive variable names
- Comment complex logic
- Follow existing style

### Documentation
- Update README.md for user-facing changes
- Update SPEC.md for architectural changes
- Add comments for non-obvious code

### Testing
- Test on multiple Ubuntu versions
- Test different hardware configurations
- Test all installation profiles

---

## Community

### Getting Help
- Join our [Discord server](https://discord.gg/cortex-linux)
- Ask questions in [GitHub Discussions](https://github.com/your-org/cortex-linux/discussions)
- Email: community@cortex-linux.org

### Staying Updated
- Star the repository
- Watch for releases
- Follow us on social media

---

## Acknowledgments

Thank you for helping make Cortex Linux better!
