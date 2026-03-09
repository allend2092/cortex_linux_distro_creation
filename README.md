# Cortex Linux: README

[![License](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Status](https://img.shields.io/badge/Status-Pre--Alpha-orange.svg)](https://github.com/your-org/cortex-linux)
[![Ubuntu](https://img.shields.io/badge/Base-Ubuntu_24.04_LTS-green.svg)](https://ubuntu.com/)

**Cortex Linux** is an AI-agent optimized Linux distribution with integrated security tools and life management utilities, designed for professionals who want their operating system to be an extension of their workflow.

---

## Philosophy

Cortex Linux is built on three core principles:

1. **AI-First** - Designed for agentic AI workflows from the ground up
2. **Secure by Isolation** - Security tools containerized, personal data protected
3. **LifeOS** - Single system for finance, health, career, relationships, and development

---

## Features

### Agentic AI Workflows
- **Ollama integration** - Run local LLMs (Llama 3, Mistral, DeepSeek)
- **Multi-agent support** - opencode, claude-code, openclaw, codex-cli
- **GPU acceleration** - CUDA, cuDNN, NCCL pre-configured
- **Agent safety rails** - Dry-run mode, confirmation prompts for destructive operations

### Modular Installation Profiles
Choose your installation profile at setup time:

| Profile | Description | Tools Included |
|---------|-------------|----------------|
| **AI/Dev** | AI development, machine learning, coding | Ollama, PyTorch, Jupyter, Docker, Git |
| **Security** | Penetration testing, network security | Nmap, Metasploit, Wireshark, Hashcat |
| **Finance** | Financial management, accounting, tax | GnuCash, Ledger, Pandas, Matplotlib |
| **General** | Productivity, career, life management | Calcurse, TaskWarrior, vdirsyncer |

### Hardware Detection
The installer automatically detects:
- CPU architecture (x86_64, ARM64, RISC-V)
- GPU manufacturer (NVIDIA, AMD, integrated)
- Available system memory
- Network interface capabilities

### Security Isolation
- **Containerized security tools** - Kali tools run in Distrobox containers
- **User data partitioning** - Separate home directory structure
- **Pre-configured firewall** - UFW with sensible defaults
- **Intrusion detection** - Auditd, fail2ban

---

## Tech Stack

| Category | Tools |
|----------|-------|
| **Base** | Ubuntu 24.04 LTS, Linux 6.6+ LTS kernel |
| **Desktop** | GNOME/KDE/XFCE (user-selectable during install) |
| **AI/ML** | Ollama, vLLM, PyTorch, TensorFlow, Transformers |
| **GPU** | NVIDIA proprietary drivers, CUDA 12.x, cuDNN |
| **Container** | Docker, Distrobox (for security tool isolation) |
| **Security** | Nmap, Wireshark, Metasploit (via Distrobox) |
| **Productivity** | tmux, htop, glances, calcurse, taskwarrior |

---

## Installation

### Meta-Installer (Recommended for Testing)

The meta-installer converts an existing Ubuntu 24.04 system into Cortex Linux:

```bash
curl -fsSL https://raw.githubusercontent.com/your-org/cortex-linux/main/install.sh | bash -s -- --profile ai-dev
```

**Options:**
- `--profile <type>` - Select installation profile (ai-dev, security, finance, general)
- `--dry-run` - Show what would be installed without making changes
- `--containerize-security` - Install security tools in Distrobox containers only

### From ISO (Coming Soon)

1. Download the ISO image
2. Create bootable USB: `dd if=cortex-linux.iso of=/dev/sdX status=progress`
3. Boot from USB and follow the installer wizard
4. Configure AI keys, security settings, and user preferences

---

## Project Roadmap

| Version | Status | Release Date | Features |
|---------|--------|--------------|----------|
| **v0.1** | In Progress | Q2 2026 | Meta-installer, hardware detection, basic profiles |
| **v0.2** | Planned | Q3 2026 | Modular profile selection, containerized security |
| **v0.3** | Planned | Q4 2026 | First-boot wizard, agent safety rails, documentation |
| **v1.0** | Planned | Q1 2027 | Full installer ISO, reproducible builds, community support |

---

## Documentation

| Document | Description |
|----------|-------------|
| [SPEC.md](SPEC.md) | Technical specification and architecture |
| [META_INSTALLER.md](META_INSTALLER.md) | Meta-installer design and implementation |
| [ETHICS.md](ETHICS.md) | Responsible use policy and ethical guidelines |

---

## Contributing

We welcome contributions! Please see our [CONTRIBUTING.md](CONTRIBUTING.md) for details.

### Code of Conduct

All contributors must adhere to our [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

---

## Legal & Ethics

**Ethical Use Only:** Cortex Linux is designed for legitimate security research, development, and personal productivity. The included security tools must only be used on systems you own or have explicit permission to test.

**Security Tools:** Tools like Metasploit, Nmap, and Hashcat are installed via Distrobox containers to isolate them from the daily driver environment. They should only be used for authorized penetration testing and security audits.

**Liability:** Users are solely responsible for ensuring their use of Cortex Linux and its tools complies with all applicable laws and regulations.

See [ETHICS.md](ETHICS.md) for complete details.

---

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details.

---

## Acknowledgments

- Built with [Ubuntu](https://ubuntu.com/) as the base distribution
- AI infrastructure powered by [Ollama](https://ollama.com/)
- Security tools sourced from [Kali Linux](https://www.kali.org/)
- Desktop environment options from [GNOME](https://www.gnome.org/), [KDE](https://kde.org/), and [Xfce](https://xfce.org/)

---

**May it be wielded wisely.**
