# Cortex Linux: Meta-Installer Design

**Version:** 0.1  
**Last Updated:** March 2026  

---

## Overview

The meta-installer is a bash script that transforms an existing Ubuntu 24.04 system into Cortex Linux. This approach was chosen over building a full ISO because it:

- **Reduces initial development time** from 3-6 months to 1-2 months
- **Enables testing on existing hardware** without virtualization or rebooting
- **Simplifies迭代 development** with immediate feedback
- **Minimizes maintenance burden** by leveraging existing Ubuntu infrastructure

### What the Meta-Installer Does

1. Detects hardware capabilities (CPU, GPU, RAM)
2. Validates system requirements (Ubuntu 24.04, minimum RAM)
3. Prompts for installation profile selection
4. Adds required repositories (CUDA, Ollama, Docker)
5. Installs base system packages
6. Installs profile-specific packages
7. Configures system defaults
8. Sets up Distrobox containers for security tools
9. Configures AI agent integration
10. Displays first-boot instructions

### What It Does NOT Do

- ❌ Modify the kernel or bootloader
- ❌ Repartition disks or install a new OS
- ❌ Overwrite existing user data
- ❌ Create a bootable ISO (that's for Phase 5)

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   META-INSTALLER ARCHITECTURE               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  STEP 1: PRE-INSTALLATION CHECKS                            │
│  ├─ Check Ubuntu version                                    │
│  ├─ Detect CPU architecture                                 │
│  ├─ Detect GPU type                                         │
│  ├─ Detect available RAM                                    │
│  └─ Validate minimum requirements                           │
│                                                             │
│  STEP 2: USER INTERACTION                                   │
│  ├─ Show hardware detection results                         │
│  ├─ Present profile selection                               │
│  ├─ Offer dry-run mode                                      │
│  └─ Confirm installation                                    │
│                                                             │
│  STEP 3: REPOSITORY CONFIGURATION                           │
│  ├─ Add CUDA repository                                     │
│  ├─ Add Ollama repository                                   │
│  ├─ Add Docker repository                                   │
│  └─ Add Distrobox repository                                │
│                                                             │
│  STEP 4: BASE SYSTEM INSTALLATION                           │
│  ├─ Update package lists                                    │
│  ├─ Install core tools (git, curl, tmux, etc.)             │
│  ├─ Install AI/ML tools (Python, pip, pyenv)              │
│  └─ Install productivity tools (htop, glances, etc.)       │
│                                                             │
│  STEP 5: PROFILE-SPECIFIC INSTALLATION                      │
│  ├─ AI/Dev:Install ML frameworks, IDEs                     │
│  ├─ Security: Prepare Distrobox containers                  │
│  ├─ Finance: Install accounting software                   │
│  └─ General: Install productivity tools                    │
│                                                             │
│  STEP 6: POST-INSTALLATION CONFIGURATION                    │
│  ├─ Configure UFW firewall                                  │
│  ├─ Set up Distrobox containers                             │
│  ├─ Configure AI agent integration                          │
│  └─ Create user directories                                 │
│                                                             │
│  STEP 7: FINALIZATION                                       │
│  ├─ Display first-boot instructions                         │
│  ├─ Show configuration summary                              │
│  └─ Offer to run post-install wizard                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Script Structure

```bash
#!/bin/bash
set -euo pipefail

# ============================================================================
# CORTEX LINUX META-INSTALLER
# ============================================================================

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# ============================================================================
# HELPER FUNCTIONS
# ============================================================================

log_info() {
    echo -e "${BLUE}[INFO]${NC} $1"
}

log_success() {
    echo -e "${GREEN}[SUCCESS]${NC} $1"
}

log_warning() {
    echo -e "${YELLOW}[WARNING]${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

check_root() {
    if [[ $EUID -ne 0 ]]; then
        log_error "This script must be run as root or with sudo"
        exit 1
    fi
}

# ============================================================================
# HARDWARE DETECTION
# ============================================================================

detect_architecture() {
    local arch=$(uname -m)
    case $arch in
        x86_64)
            echo "x86_64"
            ;;
        aarch64)
            echo "ARM64"
            ;;
        riscv64)
            echo "RISC-V"
            ;;
        *)
            log_error "Unsupported architecture: $arch"
            exit 1
            ;;
    esac
}

detect_gpu() {
    if command -v nvidia-smi >/dev/null 2>&1; then
        echo "NVIDIA"
    elif lspci | grep -i nvidia >/dev/null 2>&1; then
        echo "NVIDIA-detected"
    elif lspci | grep -i amd >/dev/null 2>&1; then
        echo "AMD"
    else
        echo "Integrated"
    fi
}

detect_ram() {
    free -g | awk '/^Mem:/{print $2}'
}

detect_disk_space() {
    df -BG / | tail -1 | awk '{print $4}' | sed 's/G//'
}

# ============================================================================
# REQUIREMENT VALIDATION
# ============================================================================

validate_requirements() {
    local min_ram=8
    local min_disk=30
    local actual_ram=$(detect_ram)
    local actual_disk=$(detect_disk_space)
    
    log_info "Validating system requirements..."
    
    if [[ $actual_ram -lt $min_ram ]]; then
        log_warning "Less than $min_ram GB RAM available ($actual_ram GB)"
        read -p "Continue anyway? (y/N) " -n 1 -r
        echo
        if [[ ! $REPLY =~ ^[Yy]$ ]]; then
            exit 1
        fi
    fi
    
    if [[ $actual_disk -lt $min_disk ]]; then
        log_warning "Less than $min_disk GB free disk space ($actual_disk GB)"
        read -p "Continue anyway? (y/N) " -n 1 -r
        echo
        if [[ ! $REPLY =~ ^[Yy]$ ]]; then
            exit 1
        fi
    fi
}

# ============================================================================
# PROFILE SELECTION
# ============================================================================

show_profiles() {
    cat << EOF

SELECT A PROFILE FOR INSTALLATION:

  1. AI/Dev
     Description: AI/ML development, coding, data science
     Includes: Ollama, PyTorch, Jupyter, Docker, Git tools
     Size: ~5 GB
     Best for: Developers, data scientists, AI researchers

  2. Security
     Description: Penetration testing, network security
     Includes: Nmap, Metasploit, Wireshark, Hashcat (via Distrobox)
     Size: ~3 GB (containers)
     Best for: Security researchers, pen testers

  3. Finance
     Description: Financial management, accounting, tax
     Includes: GnuCash, Ledger, hledger, financial Python libraries
     Size: ~1 GB
     Best for: Financial professionals, accountants

  4. General
     Description: Productivity, career management, general use
     Includes: tmux, calcurse, taskwarrior, email tools
     Size: ~500 MB
     Best for: General productivity, workflow automation

EOF
}

select_profile() {
    local default_profile="general"
    
    if [[ -n "$PROFILE" ]]; then
        echo "$PROFILE"
        return
    fi
    
    show_profiles
    
    read -p "Select profile (1-4, default: $default_profile): " profile_choice
    profile_choice=${profile_choice:-4}
    
    case $profile_choice in
        1) echo "ai-dev" ;;
        2) echo "security" ;;
        3) echo "finance" ;;
        4) echo "general" ;;
        *) 
            log_error "Invalid selection"
            exit 1
            ;;
    esac
}

# ============================================================================
# DRY-RUN MODE
# ============================================================================

print_dry_run_summary() {
    local profile=$1
    local arch=$2
    local gpu=$3
    
    cat << EOF

DRY-RUN MODE:以下将被安装

SYSTEM DETECTION:
  Architecture: $arch
  GPU: $gpu

PROFILE: $profile

PACKAGES TO BE INSTALLED:
  - Core system tools (git, curl, tmux, etc.)
  - Python 3.11+ with ML libraries
  - Profile-specific tools

REPOSITORIES TO ADD:
  - CUDA (for NVIDIA GPU)
  - Ollama (for local LLMs)
  - Docker (for containerization)
  - Distrobox (for security tool containers)

CONFIGURATION CHANGES:
  - Configure UFW firewall
  - Set up Distrobox containers (if security profile)
  - Configure AI agent integration

Would you like to proceed? [y/N]

EOF
}

# ============================================================================
# REPOSITORY CONFIGURATION
# ============================================================================

add_cuda_repo() {
    local gpu=$1
    if [[ "$gpu" == *"NVIDIA"* ]]; then
        log_info "Adding CUDA repository..."
        curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/debian11/x86_64/cuda-debian11.key | gpg --dearmor -o /usr/share/keyrings/cuda-keyring.gpg
        echo "deb [signed-by=/usr/share/keyrings/cuda-keyring.gpg] https://developer.download.nvidia.com/compute/cuda/repos/debian11/x86_64/ /" > /etc/apt/sources.list.d/cuda.list
    else
        log_info "No NVIDIA GPU detected, skipping CUDA repository"
    fi
}

add_ollama_repo() {
    log_info "Adding Ollama repository..."
    curl -fsSL https://ollama.com/InstallOllama.sh | sh
}

add_docker_repo() {
    log_info "Adding Docker repository..."
    apt-get install -y ca-certificates curl gnupg
    install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    echo "deb [signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable" > /etc/apt/sources.list.d/docker.list
}

add_distrobox_repo() {
    log_info "Adding Distrobox repository..."
    curl -fsSL https://download.opensuse.org/repositories/home:/sterni:/distrobox/Debian_12/Release.key | gpg --dearmor -o /usr/share/keyrings/distrobox.gpg
    echo "deb [signed-by=/usr/share/keyrings/distrobox.gpg] https://download.opensuse.org/repositories/home:/sterni:/distrobox/Debian_12/ ./" > /etc/apt/sources.list.d/distrobox.list
}

# ============================================================================
# PACKAGE INSTALLATION
# ============================================================================

install_base_packages() {
    log_info "Installing base system packages..."
    
    apt-get update
    apt-get upgrade -y
    
    apt-get install -y \
        build-essential \
        git \
        curl \
        wget \
        tmux \
        htop \
        glances \
        zsh \
        fzf \
        ripgrep \
        bat \
        delta \
        jq \
        yq \
        python3 \
        python3-pip \
        python3-venv \
        pyenv \
        pandoc \
        pdftk \
        poppler-utils
}

install_ai_packages() {
    log_info "Installing AI/ML packages..."
    
    apt-get install -y \
        python3-numpy \
        python3-pandas \
        python3-matplotlib \
        python3-seaborn \
        python3-scikit-learn \
        python3-tensorflow \
        python3-transformers \
        python3-torch \
        python3-pydantic \
        python3-pdfreader \
        python3-pymupdf \
        python3-ofxclient \
        python3-beancount
}

install_profile_packages() {
    local profile=$1
    
    case $profile in
        ai-dev)
            log_info "Installing AI/Dev profile packages..."
            apt-get install -y \
                docker.io \
                docker-compose \
                github-cli \
                gitlab-cli \
                jupyter \
                jupyterlab \
                nodejs \
                npm
            ;;
        security)
            log_info "Preparing Distrobox containers for security tools..."
            apt-get install -y \
                distrobox \
                podman
            # Container creation happens after install
            ;;
        finance)
            log_info "Installing Finance profile packages..."
            apt-get install -y \
                gnucash \
                ledger \
                hledger
            ;;
        general)
            log_info "Installing General profile packages..."
            apt-get install -y \
                calcurse \
                taskwarrior \
                neomutt \
                offlineimap \
                isync \
                ranger \
                wezterm \
                zathura
            ;;
    esac
}

# ============================================================================
# Distrobox Container Setup
# ============================================================================

setup_security_containers() {
    local profile=$1
    
    if [[ "$profile" != "security" ]]; then
        log_info "Skipping security containers (not security profile)"
        return
    fi
    
    log_info "Setting up security containers..."
    
    # Create Kali container
    distrobox create kali-security --image kalilinux/kali-rolling:latest
    
    # Install security tools in container
    distrobox enter kali-security -- sh -c '
        apt-get update && apt-get install -y \
            nmap \
            wireshark \
            metasploit-framework \
            sqlmap \
            hydra \
            burpsuite \
            nikto \
            bettercap \
            aircrack-ng \
            hashcat \
            enum4linux \
            nishang \
            social-engineer-toolkit
    '
    
    # Create wrapper script for easy access
    cat > /usr/local/bin/security-tools << 'EOF'
#!/bin/bash
distrobox enter kali-security -- "$@"
EOF
    chmod +x /usr/local/bin/security-tools
    
    log_success "Security tools available via: security-tools <command>"
}

# ============================================================================
# SYSTEM CONFIGURATION
# ============================================================================

configure_firewall() {
    log_info "Configuring UFW firewall..."
    
    ufw default deny incoming
    ufw default allow outgoing
    ufw allow OpenSSH
    ufw --force enable
}

configure_gpu() {
    local gpu=$1
    
    if [[ "$gpu" == *"NVIDIA"* ]]; then
        log_info "Configuring NVIDIA GPU..."
        apt-get install -y \
            nvidia-driver-535 \
            nvidia-cuda-toolkit \
            nvidia-container-toolkit
        
        systemctl enable nvidia-persistenced
    else
        log_info "No NVIDIA GPU detected, skipping GPU configuration"
    fi
}

configure_ai_agents() {
    log_info "Configuring AI agent integration..."
    
    # Create configuration templates
    mkdir -p /etc/cortex/agent-config
    cp -r /path/to/templates/* /etc/cortex/agent-config/
    
    # Enable Ollama on boot
    systemctl enable ollama 2>/dev/null || true
}

# ============================================================================
# FIRST-BOOT WIZARD SETUP
# ============================================================================

setup_first_boot_wizard() {
    log_info "Setting up first-boot wizard..."
    
    cat > /etc/cortex/first-boot-wizard << 'EOF'
#!/bin/bash
# First-boot wizard for Cortex Linux
# Run this after installation to complete setup

echo "=== Cortex Linux First-Boot Wizard ==="
echo

# Configure AI API keys
read -p "Enter Ollama API key (optional): " OLLAMA_API_KEY

# Configure Git
read -p "Enter Git username: " GIT_USERNAME
read -p "Enter Git email: " GIT_EMAIL
git config --global user.name "$GIT_USERNAME"
git config --global user.email "$GIT_EMAIL"

# Configure firewall
ufw enable

# Pull AI models
echo "Pulling AI models..."
ollama pull llama3
ollama pull mistral

echo
echo "=== First-boot setup complete ==="
echo "Next steps:"
echo "  1. Configure additional AI API keys"
echo "  2. Set up your financial data directory"
echo "  3. Explore the installed tools"
EOF
    
    chmod +x /etc/cortex/first-boot-wizard
}

# ============================================================================
# FINALICATION
# ============================================================================

display_summary() {
    local profile=$1
    local arch=$2
    local gpu=$3
    
    cat << EOF

╔═══════════════════════════════════════════════════════════════╗
║           CORTEX LINUX META-INSTALLER COMPLETE                ║
╚═══════════════════════════════════════════════════════════════╝

INSTALLATION SUMMARY:
  Profile: $profile
  Architecture: $arch
  GPU: $gpu

NEXT STEPS:

  1. Reboot your system
     sudo reboot

  2. Run the first-boot wizard
     /etc/cortex/first-boot-wizard

  3. Configure your AI agents
     Edit /etc/cortex/agent-config/

  4. Start using Cortex Linux
     - Launch AI agents: opencode, claude-code
     - Use security tools: security-tools nmap -sV target
     - Access financial tools: gnucash

SUPPORT:
   Documentation: https://github.com/allend2092/cortex_linux_distro_creation
   Issues: https://github.com/allend2092/cortex_linux_distro_creation/issues

Thank you for choosing Cortex Linux!
EOF
}

# ============================================================================
# MAIN INSTALLER LOGIC
# ============================================================================

main() {
    echo "╔═══════════════════════════════════════════════════════════════╗"
    echo "║         CORTEX LINUX META-INSTALLER v0.1                      ║"
    echo "╚═══════════════════════════════════════════════════════════════╝"
    echo
    
    check_root
    
    # Detect hardware
    local arch=$(detect_architecture)
    local gpu=$(detect_gpu)
    local ram=$(detect_ram)
    
    log_info "Hardware detection:"
    log_success "  Architecture: $arch"
    log_success "  GPU: $gpu"
    log_success "  RAM: ${ram} GB"
    
    # Validate requirements
    validate_requirements
    
    # Select profile
    local profile=$(select_profile)
    
    log_info "Selected profile: $profile"
    
    # Dry-run mode
    if [[ "$DRY_RUN" == "true" ]]; then
        print_dry_run_summary "$profile" "$arch" "$gpu"
        exit 0
    fi
    
    # Confirm installation
    read -p "Proceed with installation? [y/N] " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        exit 1
    fi
    
    echo
    log_info "Starting installation..."
    
    # Add repositories
    add_cuda_repo "$gpu"
    add_ollama_repo
    add_docker_repo
    add_distrobox_repo
    
    # Install packages
    install_base_packages
    install_ai_packages
    install_profile_packages "$profile"
    
    # Setup security containers
    setup_security_containers "$profile"
    
    # Configure system
    configure_firewall
    configure_gpu "$gpu"
    configure_ai_agents
    setup_first_boot_wizard
    
    # Display summary
    display_summary "$profile" "$arch" "$gpu"
}

# Parse arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        --profile)
            PROFILE="$2"
            shift 2
            ;;
        --dry-run)
            DRY_RUN="true"
            shift
            ;;
        *)
            shift
            ;;
    esac
done

main
```

---

## Usage Examples

### Standard Installation

```bash
sudo ./install.sh --profile ai-dev
```

### Dry-Run Mode

```bash
./install.sh --dry-run --profile security
```

### Non-interactive Installation (for automation)

```bash
sudo PROFILE=finance ./install.sh
```

---

## Error Handling

### Common Errors and Solutions

1. **Insufficient Permission**
   ```
   ERROR: This script must be run as root or with sudo
   Solution: Run with sudo ./install.sh
   ```

2. **Unsupported Architecture**
   ```
   ERROR: Unsupported architecture: armv7l
   Solution: Cortex Linux only supports x86_64 and ARM64
   ```

3. **Insufficient RAM**
   ```
   WARNING: Less than 8 GB RAM available (4 GB)
   Solution: Close other applications or upgrade RAM
   ```

4. **GPU Driver Installation Failed**
   ```
   WARNING: NVIDIA driver installation failed
   Solution: Manually install drivers: apt-get install nvidia-driver-535
   ```

### Exit Codes

| Code | Meaning |
|------|-------|
| 0 | Success |
| 1 | Error (general) |
| 2 | Hardware requirements not met |
| 3 | User cancelled installation |

---

## Testing

### Test Scenarios

1. **Clean Ubuntu Install**
   - Install on fresh Ubuntu 24.04 VM
   - Verify all packages install correctly
   - Test profile-specific tools

2. **Hardware Variations**
   - Intel CPU, integrated GPU
   - Intel CPU, NVIDIA GPU
   - AMD CPU, AMD GPU
   - ARM64 (Raspberry Pi)

3. **Profile Combinations**
   - Test each profile individually
   - Test multiple profiles on same system (if supported)

4. **Rollback Testing**
   - Backup system before installation
   - Verify rollback is possible
   - Test restore procedure

---

## Future Improvements

### Phase 2 Enhancements

1. **Progress Indicators**
   - Show progress bars for each installation step
   - Estimate time remaining

2. **Configuration Backup**
   - Backup existing configurations before modification
   - Easy restore option

3. **Post-Install Validation**
   - Test installed tools after installation
   - Report any failures

4. **Update Mechanism**
   - Auto-update script
   - Profile-specific update options

### Phase 3 Enhancements

1. **Web-Based Installer**
   - Interactive web interface
   - Profile selection with GUI

2. **Containerized Installation**
   - Install in Podman container
   - Test without affecting host system

3. **Cloud Deployment**
   - Pre-built cloud images
   - One-click deployment

---

## Conclusion

The meta-installer is the first step toward making Cortex Linux accessible and testable. By starting with an installer that works on existing Ubuntu systems, we can:

- Gather user feedback quickly
- Iterate on design without ISO complexity
- Build a community before full ISO release

Once we have a working meta-installer and user feedback, we can proceed to Phase 5: full ISO build with live CD/DVD support.

---

**Ready to test?** Run `sudo ./install.sh --dry-run` to see what would be installed on your system.
