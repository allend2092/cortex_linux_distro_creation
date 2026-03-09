# Cortex Linux: Technical Specification

**Version:** 0.1 (Pre-Alpha)  
**Base OS:** Ubuntu 24.04 LTS  
**Target Release:** Q1 2027  

---

## Table of Contents

1. [Overview](#overview)
2. [System Architecture](#system-architecture)
3. [Installation Approach](#installation-approach)
4. [Hardware Requirements](#hardware-requirements)
5. [Tool Catalog](#tool-catalog)
6. [Security Model](#security-model)
7. [Modular Profiles](#modular-profiles)
8. [Agent Safety Rails](#agent-safety-rails)
9. [Known Challenges](#known-challenges)
10. [Development Roadmap](#development-roadmap)

---

## Overview

Cortex Linux is an AI-first Linux distribution designed to empower agentic AI workflows while maintaining comprehensive tooling for security, finance, career development, and personal productivity.

### Key Differentiators

| Feature | Cortex Linux | Ubuntu | Kali Linux | Pop!_OS |
|---------|--------------|--------|------------|---------|
| AI Agent Focus | ✅ Primary Design Goal | ❌ | ❌ | ❌ |
| Security Isolation | ✅ Distrobox containers | ❌ | ✅ Global install | ❌ |
| Modular Profiles | ✅ Yes | ❌ | ❌ | ❌ |
| Life Management | ✅ Integrated | ❌ | ❌ | ❌ |
| GPU Acceleration | ✅ Optimized | ✅ | ❌ | ✅ |

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     CORTEX LINUX ARCHITECTURE               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           USER APPLICATIONS & WORKLOADS              │   │
│  │  • Financial tools (GnuCash, ledger)                 │   │
│  │  • Career tools (Git, pandoc, Anki)                  │   │
│  │  • Productivity (tmux, calcurse, taskwarrior)        │   │
│  │  • AI agents (opencode, claude-code)                 │   │
│  └──────────────────────────────────────────────────────┘   │
│                            │                                 │
│  ┌────────────────────────┼───────────────────────────────┐  │
│  │        TOOLCHAIN LAYER (Meta-Packages)                │  │
│  │  • ai-dev-meta         • security-meta                │  │
│  │  • finance-meta        • general-meta                 │  │
│  └────────────────────────┼───────────────────────────────┘  │
│                            │                                 │
│  ┌────────────────────────┼───────────────────────────────┐  │
│  │     CONTAINER & VIRTUALIZATION LAYER                  │  │
│  │  • Distrobox (security tools in Kali container)       │  │
│  │  • Docker (optional workload isolation)               │  │
│  └────────────────────────┼───────────────────────────────┘  │
│                            │                                 │
│  ┌────────────────────────┼───────────────────────────────┐  │
│  │        AI/ML RUNTIME LAYER                            │  │
│  │  • Ollama (local LLM inference)                       │  │
│  │  • vLLM (high-throughput inference)                   │  │
│  │  • CUDA, cuDNN, NCCL (GPU acceleration)               │  │
│  └────────────────────────┼───────────────────────────────┘  │
│                            │                                 │
│  ┌────────────────────────┼───────────────────────────────┐  │
│  │          BASE SYSTEM LAYER                            │  │
│  │  • Ubuntu 24.04 LTS (kernel 6.6+)                     │  │
│  │  • systemd init system                                │  │
│  │  • UFW firewall pre-configured                        │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Installation Approach

### Meta-Installer Design

The meta-installer is a bash script that transforms an existing Ubuntu 24.04 system into Cortex Linux. This approach:

- **Avoids ISO complexity** during development
- **Enables testing on existing hardware**
- **Simplifies iterative development**
- **Reduces maintenance burden**

### Installer Features

1. **Hardware Detection** - Detects CPU, GPU, RAM before proceeding
2. **Profile Selection** - User chooses which tool profiles to install
3. **Dry-Run Mode** - Preview changes without applying them
4. **Rollback Capability** - Backup & restore point creation

### Installation Steps

```
1. Download meta-installer
2. Run with profile selection
3. Hardware detection & compatibility check
4. Add required repositories (CUDA, Ollama, Distrobox)
5. Install base system packages
6. Install profile-specific packages
7. Configure system defaults
8. Set up Distrobox containers for security tools
9. Configure AI agent integration
10. First-boot wizard for user configuration
```

---

## Hardware Requirements

### Minimum Requirements

| Component | Minimum | Recommended | Notes |
|-----------|---------|-------------|-------|
| **CPU** | Dual-core | Quad-core+ | x86_64 or ARM64 |
| **RAM** | 8 GB | 16 GB+ | AI workloads need more |
| **Storage** | 30 GB | 100 GB+ | NVMe recommended |
| **GPU** | Integrated | NVIDIA RTX 3060+ | For local LLM inference |

### Hardware Detection Logic

```bash
# CPU Architecture
case $(uname -m) in
    x86_64) ARCH="x86_64" ;;
    aarch64) ARCH="ARM64" ;;
    riscv64) ARCH="RISC-V" ;;
    *) ARCH="unknown" ;;
esac

# GPU Detection
if lspci | grep -i nvidia >/dev/null 2>&1; then
    GPU_TYPE="NVIDIA"
    GPU_DRIVER="proprietary"
elif lspci | grep -i amd >/dev/null 2>&1; then
    GPU_TYPE="AMD"
    GPU_DRIVER="open-source"
else
    GPU_TYPE="integrated"
    GPU_DRIVER="none"
fi

# Memory Check
MIN_RAM=8
AVAILABLE_RAM=$(free -g | awk '/^Mem:/{print $2}')
if [ "$AVAILABLE_RAM" -lt "$MIN_RAM" ]; then
    echo "WARNING: Less than $MIN_RAM GB RAM available"
fi
```

---

## Tool Catalog

### AI & LLM Runtime

| Tool | Purpose | Version | Notes |
|------|-------|-------|------|
| Ollama | Local LLM inference | Latest | Primary LLM runner |
| vLLM | High-throughput inference | Latest | For multi-agent workloads |
| Python 3.11+ | Scripting & development | 3.11+ | With pyenv support |
| PyTorch | ML framework | Latest | With CUDA support |
| Transformers | Hugging Face models | Latest | AI agent development |

### Security Tools (Containerized)

| Tool | Purpose | Container | Notes |
|------|-------|---------|------|
| Nmap | Network discovery | Distrobox | Only in security profile |
| Wireshark | Protocol analysis | Distrobox | Only in security profile |
| Metasploit | Penetration testing | Distrobox | Only in security profile |
| Hashcat | Password cracking | Distrobox | Only in security profile |
| Burp Suite | Web application testing | Distrobox | Only in security profile |

### Financial Tools

| Tool | Purpose | Profile |
|------|-------|---------|
| GnuCash | GUI financial management | Finance |
| Ledger | CLI double-entry accounting | Finance |
| hledger | Alternative ledger format | Finance |
| pandas | Financial data analysis | Finance |
| matplotlib | Financial visualization | Finance |

### Career & Development Tools

| Tool | Purpose |
|------|-------|
| Git | Version control |
| GitHub CLI (gh) | GitHub integration |
| Pandoc | Document conversion |
| vdirsyncer | Calendar/contact sync |
| tmux | Terminal multiplexing |

### Productivity Utilities

| Tool | Purpose |
|------|-------|
| tmux | Multi-window terminal |
| htop | System monitoring |
| glances | Comprehensive system stats |
| calcurse | Calendar & scheduling |
| taskwarrior | Task management |
| ranger | Terminal file manager |

---

## Security Model

### Defense in Depth

```
┌─────────────────────────────────────────────────────────┐
│                   CORTEX LINUX SECURITY                 │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. NETWORK SECURITY                                    │
│  • UFW firewall pre-configured                          │
│  • fail2ban for brute-force protection                  │
│  • Auditd for system logging                            │
│                                                         │
│  2. DATA ISOLATION                                      │
│  • /home/user/FinancialDocuments/                       │
│  • /home/user/HealthData/                               │
│  • /home/user/Projects/                                 │
│  • /home/user/SecurityTests/ (encrypted)                │
│                                                         │
│  3. CONTAINER ISOLATION                                 │
│  • Distrobox for security tools                         │
│  • Docker for optional isolation                        │
│  • SELinux/AppArmor profiles                            │
│                                                         │
│  4. AGENT SAFETY                                        │
│  • Dry-run mode for destructive operations              │
│  • Command confirmation prompts                         │
│  • Rate limiting for AI agents                          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Distrobox Security Containers

Security tools run in isolated containers:

```bash
# User can launch security container
distrobox create kali-security
distrobox enter kali-security
# Now have full Kali toolset, isolated from main system

# Or use existing container
distrobox enter kali-security -- nmap -sV 192.168.1.0/24
```

### Ethical Usage Requirements

1. **Consent** - Only test systems you own or have written permission to test
2. **Scope** - Stay within authorized testing boundaries
3. **Reporting** - Report vulnerabilities to appropriate parties
4. **Compliance** - Adhere to all applicable laws and regulations

---

## Modular Profiles

### Profile: AI/Dev

For AI/ML development, coding, and data science.

**Included Tools:**
- Ollama (with models: llama3, mistral, deepseek-coder)
- Python 3.11+ with common ML libraries
- PyTorch/TensorFlow with CUDA support
- Jupyter, TensorBoard
- Docker, GitHub CLI
- Git, GitLab CLI
- IDE support (VS Code, JetBrains)

**Excluded Tools:**
- Security tools (run in Distrobox if needed)

### Profile: Security

For penetration testing, network security, and ethical hacking.

**Included Tools:**
- All security tools in Distrobox container
- Network monitoring (Wireshark, tshark)
- Vulnerability scanning (Nmap, Nikto)
- Password auditing (Hydra, Hashcat)
- Exploitation framework (Metasploit)

**Excluded Tools:**
- Financial tools (run in container if needed)

### Profile: Finance

For financial management, accounting, and tax preparation.

**Included Tools:**
- GnuCash (GUI accounting)
- Ledger, hledger (CLI accounting)
- Python financial libraries (pandas, matplotlib)
- PDF processing (pdftk, poppler-utils)
- LibreOffice for reports

**Excluded Tools:**
- Heavy development tools

### Profile: General

For productivity, career management, and general use.

**Included Tools:**
- tmux, htop, glances
- calcurse, taskwarrior
- vdirsyncer for sync
- mutt, neomutt for email
- ranger for file management
- Git, pandoc

**Excluded Tools:**
- AI/ML frameworks
- Security tools

---

## Agent Safety Rails

### Problem Statement

AI agents can autonomously execute commands. Without safeguards, they could:
- Delete critical files
- Accidentally expose sensitive data
- Execute destructive operations without human oversight

### Safety Mechanisms

1. **Dry-Run Mode**
```bash
# Instead of: rm -rf /tmp/data
# Use: rm -rf /tmp/data --dry-run
```

2. **Confirmation Prompts**
```bash
# Destructive operations require confirmation
Would you like to delete 2.3 GB of temporary files? [y/N]
```

3. **Operation Limits**
- Maximum execution time per command
- Maximum data modification size
- Blacklisted commands (e.g., `mkfs`, `dd if=/dev/zero`)

4. **Audit Logging**
```json
{
  "timestamp": "2024-03-15T10:30:00Z",
  "command": "git push origin main",
  "agent": "code-curator",
  "success": true,
  "data_modified": "2 files, 15 lines"
}
```

### Configuration

```bash
# ~/.config/cortex/agent-safety.conf
[limits]
max_execution_time = 300  # seconds
max_data_modification = 1000  # files
blacklisted_commands = ["mkfs", "dd if=/dev/zero", "rm -rf /"]

[prompts]
require_confirmation = true
dry_run_default = true
```

---

## Known Challenges

### Challenge 1: Package Conflicts

**Problem:** Kali security tools often require newer libraries than Ubuntu LTS provides.

**Solution:** 
- Use Distrobox to containerize security tools
- Each container can have its own package versions
- Main system stays stable

### Challenge 2: NVIDIA Driver Compatibility

**Problem:** CUDA toolkit requires specific NVIDIA driver versions.

**Solution:**
- Hardware detection before installation
- Provide multiple CUDA version options
- Document compatibility matrix

### Challenge 3: AI Model Size

**Problem:** LLMs can be 4GB+ each.

**Solution:**
- Don't bundle models in ISO/installer
- Provide curated model list post-installation
- Support multiple model providers (Ollama, LM Studio, vLLM)

### Challenge 4: Maintenance Burden

**Problem:** Keeping 100+ tools updated is time-consuming.

**Solution:**
- Modular profiles reduce installed package count
- Use meta-packages for easy group updates
- Automate dependency tracking

---

## Development Roadmap

### Phase 1: Meta-Installer (Q2 2026)

**Goal:** Working meta-installer with hardware detection

**Tasks:**
- [ ] Hardware detection script (GPU, RAM, architecture)
- [ ] Profile selection logic
- [ ] Repository configuration (CUDA, Ollama, etc.)
- [ ] Dry-run mode implementation
- [ ] Documentation for installation

**Success Criteria:**
- Meta-installer runs on clean Ubuntu 24.04
- Profile selection works correctly
- Hardware detection prevents incompatible installs

### Phase 2: Containerized Security (Q3 2026)

**Goal:** Isolate security tools in Distrobox containers

**Tasks:**
- [ ] Distrobox integration
- [ ] Security container templates
- [ ] User-friendly container launch scripts
- [ ] Documentation for ethical use

**Success Criteria:**
- Security tools run in isolated containers
- Container setup is automated
- Users can easily launch security tools

### Phase 3: Agent Safety Rails (Q4 2026)

**Goal:** Prevent AI agents from performing destructive operations

**Tasks:**
- [ ] Dry-run mode implementation
- [ ] Confirmation prompts
- [ ] Operation limits and blacklists
- [ ] Audit logging system

**Success Criteria:**
- All potentially destructive operations require confirmation
- Audit log captures all agent actions

### Phase 4: First-Boot Wizard (Q1 2027)

**Goal:** User-friendly configuration wizard

**Tasks:**
- [ ] Interactive setup wizard
- [ ] AI key configuration
- [ ] Security settings
- [ ] Documentation integration

**Success Criteria:**
- First-boot experience is guided and simple
- Users can configure all key settings in <15 minutes

### Phase 5: Full ISO Build (Q2 2027)

**Goal:** Complete ISO installer for redistribution

**Tasks:**
- [ ] Live CD/DVD creation
- [ ] ISO signing and verification
- [ ] Hardware testing (various GPUs, network cards)
- [ ] Documentation site

**Success Criteria:**
- ISO boots on multiple hardware platforms
- Installation completes successfully
- Post-install system is functional

---

## Conclusion

Cortex Linux represents a new paradigm for Linux distributions: AI-first, secure by isolation, and comprehensive for professional workflows.

This specification document will evolve as the project matures. Feedback and contributions are welcome.

---

**Next Steps:**
1. Implement meta-installer (Phase 1)
2. Test on multiple hardware configurations
3. Gather user feedback
4. Iterate on design

---

*This document is version-controlled. Last updated: March 2026*
