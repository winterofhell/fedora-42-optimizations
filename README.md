# 🚀 Fedora 42 Gaming & Performance Optimization Guide

> **Complete guide for optimizing Fedora 42 for gaming and maximum performance | by winterofhell**

## 📋 System Information

**Testing Environment:**

- **Period:** October 14, 2024 - August 11, 2025
- **Distribution:** Fedora 42 (Minimal ISO + Sway WM | Second pc: Fedora GNOME Edition)
- **Additional Testing:** GNOME DE on NVIDIA system and AMD System
- **This may also work on any other distro, but i cannot guarantee that all these tweaks will work on any distro / or your system. It is always necessary to test everything. :)**

**Hardware Configurations(tested on):**

- **Primary:** Ryzen 5 5500U, 20GB DDR4, RX550X discrete/RX Vega 7 iGPU, NVMe disk
- **Secondary:** Ryzen 5 5600, 16GB DDR4, GTX 1060, SATA SSD
- **New one:** Ryzen 5 7500f, 32Gb DDR5, RX 9070 XT, Nvme M2.

-----

## 🛠 Initial Setup & Preparation

### 1. Minimal Installation

For optimal performance, always start with the **Fedora Minimal ISO**. This approach eliminates unnecessary packages and services that can impact system resources. Btw it is not necessary to do this.

### 2. Enable RPM Fusion Repositories

RPM Fusion provides essential multimedia codecs and proprietary drivers:

```bash
sudo dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

📖 **Official Guide:** [RPM Fusion Configuration](https://rpmfusion.org/Configuration)

### 3. SELinux Configuration (Optional)

⚠️ **Security Warning:** Disabling SELinux reduces system security but also makes ur system a little faster. Only proceed if you understand the implications. (Personally, I don't care about SELinux and i always disable it.)

**Temporary disable (until reboot):**

```bash
sudo setenforce 0
```

**Permanent disable (requires reboot):**

```bash
sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
# Reboot required for changes to take effect
```

### 4. System Update

Always start with a fully updated system:

```bash
sudo dnf upgrade --refresh
```

-----

## ⚡ Kernel Optimization

### CachyOS Kernel Installation

The CachyOS kernel provides significant performance improvements for gaming and general system responsiveness.

**Prerequisites:** CPU must support x86_64_v3 instruction set !!

```bash
# Checking for the cpu support
Check support by the following the command

/lib64/ld-linux-x86-64.so.2 --help | grep "(supported, searched)"

# If it does not detect x86_64_v3 support do NOT install this kernel. If it detects only x86_64_v2, you can use the LTS kernel.
```

```bash
# Add CachyOS COPR repository
sudo dnf copr enable bieszczaders/kernel-cachyos

# Install CachyOS kernel
sudo dnf install kernel-cachyos kernel-cachyos-devel
```

📖 **More Info:** [CachyOS Kernel Installation](https://copr.fedorainfracloud.org/coprs/bieszczaders/kernel-cachyos/)

### UKSMD Installation

UKSMD (Userspace Kernel Same-page Merging Daemon) reduces memory usage and improves system responsiveness:

```bash
# Add UKSMD addon repository
sudo dnf copr enable bieszczaders/kernel-cachyos-addons

# Install UKSMD
sudo dnf install uksmd

# Enable and start UKSMD service
sudo systemctl enable --now uksmd
```

📖 **More Info:** [UKSMD Addons](https://copr.fedorainfracloud.org/coprs/bieszczaders/kernel-cachyos-addons/)

-----

## 🔧 System Services Optimization

### Ananicy-cpp Installation

Ananicy-cpp automatically manages process priorities and reduces system latency:

```bash
# Install build dependencies
sudo dnf groupinstall "Development Tools"
sudo dnf install cmake systemd-devel spdlog-devel fmt-devel nlohmann-json-devel

# Clone and build
git clone https://gitlab.com/ananicy-cpp/ananicy-cpp.git
cd ananicy-cpp
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j$(nproc)
sudo make install

# Enable service
sudo systemctl enable --now ananicy-cpp
```

### Service Management

Disable unnecessary services to free system resources:

```bash
sudo systemctl disable --now \
    bluetooth.service \
    ModemManager.service \
    cups.service \
    avahi-daemon.service \
    chronyd.service \
    NetworkManager-wait-online.service \
    geoclue.service \
    smartd.service \
    upower.service
```

💡 **Tip:** Only disable services you don’t need. Review each service before disabling to avoid breaking functionality you rely on.

-----

## ⚙️ GRUB Kernel Parameters

### Configuration

Edit `/etc/default/grub` and modify the kernel command line:

```bash
sudo nano /etc/default/grub
```

Add these parameters to `GRUB_CMDLINE_LINUX`:

```bash
GRUB_CMDLINE_LINUX="quiet lpj=XXXXXXX mitigations=off elevator=mq-deadline nowatchdog page_alloc.shuffle=1 pci=pcie_bus_perf intel_idle.max_cstate=1 processor.max_cstate=1 libahci.ignore_sss=1 noautogroup amd_pstate=active"
```

**Get the LPJ value:**

```bash
sudo dmesg | grep -o "lpj=[0-9]*"
```

### Update GRUB Configuration

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

**Parameter Explanations:**

- `mitigations=off` - Disables CPU vulnerability mitigations for better performance
- `elevator=mq-deadline` - Uses deadline I/O scheduler (better for ssds than noop)
- `nowatchdog` - Disables hardware watchdog
- `intel_idle.max_cstate=1` - Limits CPU idle states for lower latency (intel only)
- `amd_pstate=active` - Enables AMD P-State driver for better power management

-----

## 🎯 Advanced System Tweaks

### Memory Management

**Enable systemd-oomd (Out-of-Memory Daemon):**

```bash
sudo systemctl enable --now systemd-oomd
```

### Storage Optimization

**Enable SSD TRIM:**

```bash
# Enable automatic TRIM
sudo systemctl enable --now fstrim.timer

# Run manual TRIM
sudo fstrim -v /
```

### Graphics Optimization (AMD Users)

Add to `/etc/environment`:

```bash
# AMD GPU optimizations
RADV_PERFTEST=gpl
mesa_glthread=true
AMD_VULKAN_ICD=RADV
```

### CPU Scaling Configuration

**For AMD systems:**

```bash
echo "active" | sudo tee /sys/devices/system/cpu/amd_pstate/status
```

**For Intel systems:**

```bash
echo "passive" | sudo tee /sys/devices/system/cpu/intel_pstate/status
```

### System Limits

**Increase file descriptor limit** in `/etc/security/limits.conf`:

```bash
# Replace 'yourusername' with your actual username
yourusername hard nofile 1048576
yourusername soft nofile 1048576
```

### IRQ Balance (Intel iGPU Users)

If experiencing performance issues with Intel integrated graphics:

```bash
# Check status
sudo systemctl status irqbalance

# Disable if needed
sudo systemctl disable --now irqbalance
```

-----

## 🎮 Gaming Optimizations

### GameMode Installation

GameMode applies system optimizations when gaming:

```bash
sudo dnf install gamemode gamemode-devel

# Verify installation
gamemoded -t
```

**Usage:** Launch games with `gamemoderun` prefix or configure in Steam launch options.

### Windows Games Compatibility

**PortProton** offers excellent compatibility for Windows executables (i use portproton instead of Lutris / Bottles and this is my fav proton executor app!):

```bash
sudo dnf copr enable boria138/portproton
sudo dnf install portproton
```

### Steam Optimizations

Add to Steam launch options for games:

```bash
gamemoderun %command%
```

Or for Proton games:

```bash
gamemoderun DXVK_ASYNC=1 %command%
```

-----

## 🧹 Maintenance & Cleanup

### Package Cache Management

**Clean DNF cache:**

```bash
sudo dnf clean all
```

**Clean system journals:**

```bash
# Keep only last 7 days of logs
sudo journalctl --vacuum-time=7d

# Or limit by size (keep only 100MB)
sudo journalctl --vacuum-size=100M
```

### Automated Maintenance

Create a simple maintenance script:

```bash
#!/bin/bash
# Save as ~/maintenance.sh and make executable

echo "🧹 Running system maintenance..."

# Update system
sudo dnf upgrade --refresh

# Clean caches
sudo dnf clean all

# Clean old journal entries
sudo journalctl --vacuum-time=7d

# Run TRIM on SSD
sudo fstrim -v /

echo "✅ Maintenance complete!"
```

-----

## 🖥️ Desktop Environment Recommendations

### Lightweight Alternatives

For maximum performance, consider these lightweight desktop environments (if installing using Minimal iso):

- **Sway** - Wayland-based tiling compositor (i have 700mb on idle with it)
- **i3** - X11 tiling window manager (600mb on idle)
- **Hyprland** - Modern Wayland compositor with animations (900mb on idle)
- **XFCE** - Lightweight traditional desktop
- **LXQt** - Qt-based lightweight desktop

**KDE Plasma Edition (NEW in Fedora 42):**
KDE Plasma is now an official Fedora edition alongside Workstation (GNOME). 
This means better integration, support, and optimization out of the box.

### Fedora 42 Specific Enhancements

**What's New for Performance:**
- Updated Mesa drivers for better AMD/Intel graphics
- Improved Wayland compositor performance
- Better power management defaults
- Enhanced container support

**Fedora 42 Gaming Improvements:**
- Better Steam Deck compatibility mode
- Enhanced Proton integration
- Improved HDR support preparation

### Laptop Power Management

**Install Power Optimization Tools:**
```bash
sudo dnf install powertop tlp tlp-rdw

sudo systemctl enable --now tlp

# Configure TLP for gaming/performance mode

sudo nano /etc/tlp.conf

# Set: TLP_DEFAULT_MODE=performance (when plugged in)
```

### Advanced Memory Management

**Configure Swap Behavior:**
```bash
# Reduce swappiness for better gaming performance
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf

# Improve memory allocation for gaming
echo 'vm.vfs_cache_pressure=50' | sudo tee -a /etc/sysctl.conf
```

**Container/Flatpak Gaming Section**


### Container Gaming Optimization

**Steam Flatpak Optimization:**
```bash
# Install Steam as Flatpak for better sandboxing
flatpak install com.valvesoftware.Steam

# Grant necessary permissions for gaming
flatpak override --user --filesystem=~/.local/share/Steam com.valvesoftware.Steam
```


### GNOME Optimizations

If staying with GNOME:

```bash
# Install GNOME tweaks
sudo dnf install gnome-tweaks gnome-extensions-app

# Disable animations for better performance
gsettings set org.gnome.desktop.interface enable-animations false

# Reduce resource usage
gsettings set org.gnome.shell.overrides workspaces-only-on-primary false
```

-----

## 🎮 NVIDIA Graphics Optimization for Fedora 42 (Wayland)

<details>
<summary>NVIDIA optimization, drivers installation, tools and more</summary>

> **Comprehensive optimization guide for NVIDIA GPUs on Fedora 42 with Wayland display server**

### 📋 NVIDIA System Requirements

**Supported GPUs:**

- GTX 900/1000 series and newer (Maxwell, Pascal, Turing, Ampere, Ada Lovelace, Blackwell)
- RTX 20/30/40/50 series with full feature support
- Quadro and Tesla cards (professional workloads)

**Driver Compatibility:**

- **Recommended:** NVIDIA 575+ drivers for optimal Wayland support
- **Minimum:** NVIDIA 570+ for stable Wayland functionality
- **Note:** NVIDIA driver stack seeing much better Wayland support with its latest drivers

-----

### 🔧 NVIDIA Driver Installation (Fedora 42 Wayland)

#### Method 1: RPM Fusion (Strongly Recommended)

RPM Fusion remains the most reliable method for NVIDIA drivers on Fedora 42. This approach ensures proper integration with the Wayland display server and system updates.

```bash
# Enable RPM Fusion repositories (if not already enabled)
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

# Update system packages
sudo dnf update

# Install NVIDIA drivers with Wayland support
sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda

# Install 32-bit compatibility libraries (essential for Steam, Wine, gaming)
sudo dnf install xorg-x11-drv-nvidia-libs.i686

# Install NVIDIA settings utility
sudo dnf install nvidia-settings

# Install additional tools for monitoring
sudo dnf install nvidia-ml-py3
```

#### Post-Installation Verification

Understanding what each command tells us helps ensure your system is properly configured for optimal performance.

```bash
# Verify driver installation and check version
nvidia-smi
# This should show your GPU, driver version (570+), and current utilization

# Confirm CUDA support is available
nvidia-smi -q | grep "CUDA Version"
# Essential for AI workloads and some games using GPU compute

# Verify Wayland is using NVIDIA GPU
echo $XDG_SESSION_TYPE
# Should output "wayland" on Fedora 42

# Check that GBM backend is working
nvidia-smi --query-gpu=name,driver_version --format=csv
# Confirms proper driver loading
```

#### Enable Wayland for NVIDIA (Essential Step)

Wayland requires specific configuration to work properly with NVIDIA drivers. This step ensures your desktop environment can utilize hardware acceleration.

```bash
# Enable DRM kernel mode setting (required for Wayland)
echo 'options nvidia_drm modeset=1 fbdev=1' | sudo tee /etc/modprobe.d/nvidia-drm-modeset.conf

# Enable early loading of NVIDIA modules
echo -e 'nvidia\nnvidia_modeset\nnvidia_uvm\nnvidia_drm' | sudo tee /etc/modules-load.d/nvidia.conf

# Rebuild initramfs to include changes
sudo dracut --force

# Reboot to apply kernel module changes
sudo reboot
```

-----

### ⚡ NVIDIA Wayland Performance Optimizations

#### 1. Environment Variables for Wayland

These environment variables optimize NVIDIA GPU behavior specifically for Wayland compositors. Unlike X11, Wayland handles many optimizations automatically, but these variables fine-tune performance.

Add to `/etc/environment`:

```bash
# Core NVIDIA Wayland optimizations
GBM_BACKEND=nvidia-drm
__GLX_VENDOR_LIBRARY_NAME=nvidia

# Enable threaded optimizations (improves CPU-GPU parallelism)
__GL_THREADED_OPTIMIZATIONS=1

# Shader compilation caching (reduces loading times)
__GL_SHADER_DISK_CACHE=1
__GL_SHADER_DISK_CACHE_PATH=/tmp/nvidia-shader-cache
__GL_SHADER_DISK_CACHE_SIZE=1073741824

# Disable VSync for gaming (reduces input lag)
__GL_SYNC_TO_VBLANK=0

# Enable unofficial protocol extensions (compatibility)
__GL_ALLOW_UNOFFICIAL_PROTOCOL=1

# Wayland-specific optimizations
WLR_DRM_NO_ATOMIC=1
WLR_NO_HARDWARE_CURSORS=1

# Gaming optimizations
NVIDIA_DRIVER_CAPABILITIES=all
PROTON_ENABLE_NVAPI=1
```

#### 2. Kernel Module Parameters

Modern NVIDIA drivers benefit from specific kernel parameters that improve Wayland compatibility and performance.

Create `/etc/modprobe.d/nvidia-power-management.conf`:

```bash
# Enable modern power management features
options nvidia NVreg_DynamicPowerManagement=0x02

# Enable Page Attribute Table (improves memory performance)
options nvidia NVreg_UsePageAttributeTable=1

# Enable ResizableBAR support (RTX 30/40/50 series)
options nvidia NVreg_EnableResizableBar=1

# Preserve video memory during suspend
options nvidia NVreg_PreserveVideoMemoryAllocations=1

# Enable stream memory operations (required for some workloads)
options nvidia NVreg_EnableStreamMemOPs=1
```

#### 3. GNOME Wayland Specific Settings

GNOME on Wayland requires particular attention to achieve optimal NVIDIA performance. These settings address common issues with the GNOME compositor.

```bash
# Enable NVIDIA acceleration for GNOME Wayland session
gsettings set org.gnome.mutter experimental-features "['variable-refresh-rate']"

# Configure GNOME for gaming performance
gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'cycle-windows'
gsettings set org.gnome.desktop.interface enable-animations false

# Set scaling factor for high-DPI displays (adjust as needed)
gsettings set org.gnome.desktop.interface scaling-factor 1
```

#### 4. KDE Plasma Wayland Configuration

KDE Plasma has excellent Wayland support and works particularly well with NVIDIA drivers when properly configured.

```bash
# Enable variable refresh rate support
kwriteconfig5 --file kwinrc --group Compositing --key VariableRefreshRate true

# Optimize compositor settings for gaming
kwriteconfig5 --file kwinrc --group Compositing --key LatencyPolicy Low
kwriteconfig5 --file kwinrc --group Compositing --key RenderTimeEstimator 1

# Restart KWin to apply changes
qdbus org.kde.KWin /KWin reconfigure
```

-----

### 🏎️ Gaming-Specific NVIDIA Optimizations

#### 1. Steam Launch Options for Wayland

Steam gaming on Wayland requires specific launch parameters to ensure games use the NVIDIA GPU properly and achieve optimal performance.

**For native Linux games:**

```bash
# Basic optimization with GameMode
gamemoderun __GL_THREADED_OPTIMIZATIONS=1 %command%

# Enhanced performance for competitive gaming
gamemoderun __GL_SYNC_TO_VBLANK=0 __GL_THREADED_OPTIMIZATIONS=1 %command%
```

**For Proton/Wine games:**

```bash
# Standard Proton optimization
gamemoderun __GL_THREADED_OPTIMIZATIONS=1 PROTON_ENABLE_NVAPI=1 %command%

# Advanced optimization with DXVK async compilation
gamemoderun __GL_THREADED_OPTIMIZATIONS=1 DXVK_ASYNC=1 PROTON_ENABLE_NVAPI=1 %command%

# For games requiring maximum performance
gamemoderun __GL_SYNC_TO_VBLANK=0 __GL_THREADED_OPTIMIZATIONS=1 DXVK_ASYNC=1 PROTON_ENABLE_NVAPI=1 %command%
```

#### 2. Lutris Gaming Optimization

Lutris provides excellent integration with NVIDIA drivers on Wayland. Configure these settings for optimal gaming performance.

```bash
# Install Lutris with NVIDIA support
sudo dnf install lutris wine

# Configure Lutris environment variables (in Lutris preferences)
# Add these to System Options → Environment variables:
__GL_THREADED_OPTIMIZATIONS=1
__GL_SHADER_DISK_CACHE=1
PROTON_ENABLE_NVAPI=1
DXVK_HUD=fps,memory,gpuload
```

#### 3. GameMode Integration

GameMode automatically optimizes system performance during gaming sessions, providing better resource allocation and reduced latency.

```bash
# Install GameMode
sudo dnf install gamemode

# Configure GameMode for NVIDIA optimization
sudo tee /etc/gamemode.ini << 'EOF'
[general]
renice=10
ioprio=1

[gpu]
apply_gpu_optimisations=accept-responsibility
gpu_device=0
amd_performance_level=high

[custom]
start=nvidia-smi -pm 1 && nvidia-smi -pl 300
end=nvidia-smi -pm 0 && nvidia-smi -ac -r
EOF
```

-----

### 🔥 Advanced NVIDIA Wayland Tweaks

#### 1. Variable Refresh Rate (VRR) Support

Modern NVIDIA drivers support variable refresh rate on Wayland, providing smoother gaming experiences with compatible monitors.

```bash
# Enable VRR in GNOME (requires GNOME 45+)
gsettings set org.gnome.mutter experimental-features "['variable-refresh-rate']"

# For KDE Plasma, enable in system settings or via command:
kwriteconfig5 --file kwinrc --group Compositing --key VariableRefreshRate true

# Verify VRR is working
sudo dnf install drm_info
drm_info | grep -i vrr
```

#### 2. HDR Support (Experimental)

High Dynamic Range support is gradually improving on Wayland with NVIDIA drivers. These settings enable experimental HDR functionality.

```bash
# Enable HDR support (requires compatible display and recent drivers)
echo 'options nvidia NVreg_EnableHDR=1' | sudo tee /etc/modprobe.d/nvidia-hdr.conf

# GNOME HDR support (experimental, GNOME 46+)
gsettings set org.gnome.mutter experimental-features "['variable-refresh-rate','hdr']"
```

#### 3. Performance Monitoring and Tuning

Effective performance monitoring helps identify bottlenecks and verify that optimizations are working correctly.

```bash
# Install monitoring tools
sudo dnf install nvtop mangohud goverlay

# Create monitoring script for gaming sessions
sudo tee /usr/local/bin/nvidia-gaming-monitor.sh << 'EOF'
#!/bin/bash
echo "=== NVIDIA Gaming Performance Monitor ==="
echo "GPU: $(nvidia-smi --query-gpu=name --format=csv,noheader)"
echo "Driver: $(nvidia-smi --query-gpu=driver_version --format=csv,noheader)"
echo "=== Real-time Stats ==="
nvidia-smi dmon -s pucvmet
EOF

sudo chmod +x /usr/local/bin/nvidia-gaming-monitor.sh
```

-----

### 🛡️ Power Management and Thermal Optimization

#### 1. Advanced Power Management

Proper power management ensures consistent performance while preventing unnecessary power consumption during idle periods.

```bash
# Configure advanced power management
echo 'options nvidia NVreg_DynamicPowerManagement=0x02' | sudo tee -a /etc/modprobe.d/nvidia-power.conf

# Enable runtime power management for laptops
sudo tee /etc/udev/rules.d/80-nvidia-pm.rules << 'EOF'
# Enable runtime PM for NVIDIA VGA/3D controller devices
SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x030000", TEST=="power/control", ATTR{power/control}="auto"
SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x030200", TEST=="power/control", ATTR{power/control}="auto"
EOF
```

#### 2. Thermal Management

Effective thermal management prevents throttling and maintains optimal performance during extended gaming sessions.

```bash
# Install thermal monitoring tools
sudo dnf install lm_sensors

# Configure sensor detection
sudo sensors-detect --auto

# Create thermal monitoring script
sudo tee /usr/local/bin/nvidia-thermal.sh << 'EOF'
#!/bin/bash
TEMP=$(nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader,nounits)
POWER=$(nvidia-smi --query-gpu=power.draw --format=csv,noheader,nounits)

echo "GPU Temperature: ${TEMP}°C"
echo "Power Draw: ${POWER}W"

# Alert if temperature is high
if [ $TEMP -gt 83 ]; then
    echo "WARNING: GPU temperature is high!"
    notify-send "GPU Temperature Warning" "GPU is running at ${TEMP}°C"
fi
EOF

sudo chmod +x /usr/local/bin/nvidia-thermal.sh
```

-----

### 🐛 NVIDIA Wayland Troubleshooting

#### Common Issues and Modern Solutions

Understanding how to diagnose and resolve issues ensures optimal performance and system stability.

**1. Wayland Session Not Starting with NVIDIA:**

This is the most common issue when transitioning from X11 to Wayland with NVIDIA drivers.

```bash
# Verify kernel module parameters are correct
cat /etc/modprobe.d/nvidia-drm-modeset.conf
# Should contain: options nvidia_drm modeset=1 fbdev=1

# Check if DRM modeset is enabled
cat /sys/module/nvidia_drm/parameters/modeset
# Should output: Y

# Rebuild initramfs and reboot if necessary
sudo dracut --force
sudo reboot

# Verify Wayland session after reboot
echo $XDG_SESSION_TYPE
# Should output: wayland
```

**2. Poor Gaming Performance Despite Good Hardware:**

Performance issues often stem from incorrect GPU usage or power management settings.

```bash
# Verify GPU is being utilized
nvidia-smi dmon -s pucvmet -c 10

# Check for power limiting
nvidia-smi --query-gpu=power.limit,power.draw --format=csv
# Power draw should approach power limit during gaming

# Monitor GPU clocks during gaming
watch -n 1 'nvidia-smi --query-gpu=clocks.gr,clocks.mem --format=csv'
# Clocks should reach maximum values during gaming
```

**3. Screen Tearing or Stuttering:**

Modern Wayland compositors handle tearing better than X11, but some configuration may be needed.

```bash
# For GNOME, ensure VRR is enabled
gsettings get org.gnome.mutter experimental-features
# Should include 'variable-refresh-rate'

# For games, disable VSync in-game and use compositor VSync
# Add to Steam launch options:
__GL_SYNC_TO_VBLANK=0 %command%
```

**4. High Idle Power Consumption:**

Preventing unnecessary power draw during idle periods improves battery life and reduces heat.

```bash
# Enable runtime power management
echo 'auto' | sudo tee /sys/bus/pci/devices/0000:*/power/control

# Verify power management is working
cat /sys/bus/pci/devices/0000:*/power/runtime_status
# Should show 'suspended' for idle GPU

# Monitor idle power consumption
nvidia-smi --query-gpu=power.draw --format=csv --loop=1
```

-----

### 🔧 NVIDIA Developer and AI Tools

#### CUDA Development Environment

Setting up CUDA properly ensures compatibility with AI frameworks and development tools.

```bash
# Install CUDA toolkit
sudo dnf install cuda-toolkit cuda-devel cuda-runtime

# Configure environment variables
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export CUDA_HOME=/usr/local/cuda' >> ~/.bashrc

# Reload environment
source ~/.bashrc

# Verify CUDA installation
nvcc --version
nvidia-smi --query-gpu=compute_cap --format=csv
```

#### Container Support for AI/ML Workloads

Container support enables easy deployment of AI and machine learning applications.

```bash
# Install NVIDIA Container Toolkit
curl -fsSL https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo | \
  sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo

sudo dnf install nvidia-container-toolkit

# Configure Docker/Podman
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# Test container support
sudo docker run --rm --gpus all nvidia/cuda:12.3-runtime-ubuntu20.04 nvidia-smi
```

-----

### 📊 Performance Monitoring and Benchmarking

#### Comprehensive Monitoring Setup

Effective monitoring helps optimize performance and identify potential issues before they impact gaming or work performance.

```bash
# Install comprehensive monitoring suite
sudo dnf install nvtop btop mangohud goverlay

# Create performance monitoring script
sudo tee /usr/local/bin/nvidia-perf-monitor.sh << 'EOF'
#!/bin/bash
clear
echo "=== NVIDIA Performance Monitor ==="
echo "System: $(hostnamectl --static) | $(date)"
echo "Driver: $(nvidia-smi --query-gpu=driver_version --format=csv,noheader)"
echo "GPU: $(nvidia-smi --query-gpu=name --format=csv,noheader)"
echo ""

# GPU utilization and memory
nvidia-smi --query-gpu=utilization.gpu,memory.used,memory.total,temperature.gpu,power.draw,clocks.gr,clocks.mem --format=csv

echo ""
echo "=== Active GPU Processes ==="
nvidia-smi pmon -c 1

echo ""
echo "Press Ctrl+C to exit continuous monitoring..."
watch -n 2 nvidia-smi
EOF

sudo chmod +x /usr/local/bin/nvidia-perf-monitor.sh
```

#### Gaming Performance Overlay

MangoHud provides real-time performance metrics during gaming sessions.

```bash
# Configure MangoHud for optimal display
mkdir -p ~/.config/MangoHud

cat > ~/.config/MangoHud/MangoHud.conf << 'EOF'
# GPU and CPU information
gpu_stats
cpu_stats
gpu_temp
cpu_temp

# Frame rate and timing
fps
frametime
frame_timing

# Memory usage
vram
ram

# Position and appearance
position=top-left
font_size=22
alpha=0.8

# Limit logging to prevent performance impact
log_duration=60
EOF
```

-----

### 📚 Additional Resources

**Official NVIDIA Documentation:**
- [NVIDIA Linux Driver Installation Guide](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/index.html)
- [CUDA Installation Guide for Linux](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/)

**Fedora-Specific Resources:**
- [RPM Fusion NVIDIA Guide](https://rpmfusion.org/Howto/NVIDIA)
- [Fedora 42 Release Notes](https://fedoraproject.org/wiki/Releases/42/ChangeSet)

**Wayland and Gaming:**
- [Gaming on Linux with NVIDIA](https://www.gamingonlinux.com/)
- [MangoHud Documentation](https://github.com/flightlessmango/MangoHud)

-----

*💡 **Pro Tip:** Wayland offers hardware acceleration and generally better performance than traditional X11, making it the optimal choice for modern NVIDIA gaming setups on Fedora 42. Always verify changes incrementally and monitor performance to ensure optimal configuration.*

</details>

-----

## 🔍 Monitoring & Verification

### Performance Monitoring Tools

```bash
# Install useful monitoring tools
sudo dnf install htop iotop powertop neofetch

# For detailed system information
sudo dnf install hardinfo
```

### Benchmark Tools

```bash
# Gaming benchmarks
sudo dnf install glmark2 unigine-superposition

# System benchmarks
sudo dnf install sysbench stress-ng
```

-----

## 🚨 Troubleshooting

### Common Issues

**1. Boot Issues After Kernel Parameters:**

- Boot with previous kernel from GRUB menu
- Remove problematic parameters from `/etc/default/grub`
- Regenerate GRUB config

**2. Graphics Issues:**

- Check driver installation: `lspci -k | grep -A 2 -E "(VGA|3D)"`
- Verify correct driver loading: `lsmod | grep -E "(amdgpu|nvidia|i915)"`

**3. Performance Regression:**

- Monitor system resources: `htop`, `iotop`
- Check for thermal throttling: `watch sensors`
- Verify services status: `systemctl list-units --failed`

### Recovery Commands

```bash
# Reset GRUB to defaults
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# Reset SELinux context (if re-enabling SELinux)
sudo restorecon -R /

# Check system integrity
sudo dnf check
sudo rpm -Va
```

-----

## 📊 Expected Performance Gains

Based on testing, users can expect:

- **Boot Time:** 15-30% improvement
- **Gaming Performance:** 5-15% FPS increase
- **System Responsiveness:** Significantly reduced input lag
- **Memory Usage:** 10-20% reduction in idle RAM usage
- **Storage Performance:** Improved SSD performance with trim

-----

## 🌍 Русская версия | НЕ все переведено (Russian Translation)

<details>
<summary>Нажмите для просмотра русской версии</summary>

# 🚀 Руководство по оптимизации Fedora 42 для игр и производительности

## 📋 Информация о системе

**Среда тестирования:**

- **Период:** 14 октября 2024 - 11 августа 2025
- **Дистрибутив:** Fedora 42 (Минимальный ISO + Sway WM / 2 система на Гноме)
- **Дополнительное тестирование:** GNOME DE на системе с NVIDIA

**Конфигурации оборудования:**

- **Основная:** Ryzen 5 5500U, 20ГБ DDR4, дискретная RX550X/встроенная RX Vega 7, NVMe диск
- **Вторичная:** Ryzen 5 5600, 16ГБ DDR4, GTX 1060, SATA SSD
- **Новая система:** Ryzen 5 7500f, 32Гб Ддр5, RX 9070 XT, nvme m2

## 🛠 Первоначальная настройка и подготовка

### 1. Минимальная установка

Для оптимальной производительности всегда начинайте с **Fedora Minimal ISO**. Этот подход исключает ненужные пакеты и службы.

### 2. Включение репозиториев RPM Fusion

```bash
sudo dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

### 3. Настройка SELinux (опционально)

⚠️ **Предупреждение о безопасности:** Отключение SELinux снижает безопасность системы.

**Временное отключение:**

```bash
sudo setenforce 0
```

**Постоянное отключение:**

```bash
sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
# Требуется перезагрузка
```

### 4. Обновление системы

```bash
sudo dnf upgrade --refresh
```

## ⚡ Оптимизация ядра

### Установка ядра CachyOS

```bash
sudo dnf copr enable bieszczaders/kernel-cachyos
sudo dnf install kernel-cachyos kernel-cachyos-devel
```

### Установка UKSMD

```bash
sudo dnf copr enable bieszczaders/kernel-cachyos-addons
sudo dnf install uksmd
sudo systemctl enable --now uksmd
```

## 🔧 Оптимизация системных служб

### Установка Ananicy-cpp

```bash
sudo dnf groupinstall "Development Tools"
sudo dnf install cmake systemd-devel spdlog-devel fmt-devel nlohmann-json-devel

git clone https://gitlab.com/ananicy-cpp/ananicy-cpp.git
cd ananicy-cpp
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j$(nproc)
sudo make install
sudo systemctl enable --now ananicy-cpp
```

### Отключение ненужных служб

```bash
sudo systemctl disable --now \
    bluetooth.service \
    ModemManager.service \
    cups.service \
    avahi-daemon.service \
    chronyd.service \
    NetworkManager-wait-online.service \
    geoclue.service \
    smartd.service \
    upower.service
```

## ⚙️ Параметры ядра GRUB

Отредактируйте `/etc/default/grub`:

```bash
GRUB_CMDLINE_LINUX="quiet lpj=XXXXXXX mitigations=off elevator=mq-deadline nowatchdog page_alloc.shuffle=1 pci=pcie_bus_perf intel_idle.max_cstate=1 processor.max_cstate=1 libahci.ignore_sss=1 noautogroup amd_pstate=active"
```

Получите значение LPJ:

```bash
sudo dmesg | grep -o "lpj=[0-9]*"
```

Обновите GRUB:

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

## 🎯 Продвинутые настройки системы

### Управление памятью

```bash
sudo systemctl enable --now systemd-oomd
```

### Оптимизация хранилища

```bash
sudo systemctl enable --now fstrim.timer
sudo fstrim -v /
```

### Оптимизация графики (пользователи AMD)

Добавьте в `/etc/environment`:

```bash
RADV_PERFTEST=gpl
mesa_glthread=true
AMD_VULKAN_ICD=RADV
```

### Настройка масштабирования CPU

**Для систем AMD:**

```bash
echo "active" | sudo tee /sys/devices/system/cpu/amd_pstate/status
```

**Для систем Intel:**

```bash
echo "passive" | sudo tee /sys/devices/system/cpu/intel_pstate/status
```

### Системные лимиты

В `/etc/security/limits.conf`:

```bash
yourusername hard nofile 1048576
yourusername soft nofile 1048576
```

## 🎮 Игровые оптимизации

### Установка GameMode

```bash
sudo dnf install gamemode gamemode-devel
```

### PortProton для Windows игр

```bash
sudo dnf copr enable boria138/portproton
sudo dnf install portproton
```

## 🧹 Обслуживание и очистка

### Очистка кэша пакетов

```bash
sudo dnf clean all
sudo journalctl --vacuum-time=7d
```

## 🖥️ Рекомендации по окружению рабочего стола

Для максимальной производительности рассмотрите:

- **Sway** - Wayland композитор
- **i3** - X11 тайловый менеджер окон
- **Hyprland** - Современный Wayland композитор
- **XFCE** - Легковесный традиционный рабочий стол

</details>

-----

## 🤝 Contributing

Found improvements or have suggestions? Feel free to:

- Open an issue on GitHub
- Submit a pull request
- Share your optimization results

-----

## 📚 Additional Resources

- [Fedora Documentation](https://docs.fedoraproject.org/)
- [RPM Fusion](https://rpmfusion.org/)
- [CachyOS Kernel](https://github.com/CachyOS/linux-cachyos)
- [Ananicy-cpp](https://gitlab.com/ananicy-cpp/ananicy-cpp)
- [Gaming on Linux](https://www.gamingonlinux.com/)

-----

## ⚖️ Disclaimer

This guide modifies system settings that may affect stability and security. Always:

- Create system backups before applying changes
- Test changes on non-critical systems first
- Understand the implications of each modification
- Keep recovery media accessible

**Performance results may vary** based on hardware configuration and specific use cases.

-----

## 📝 Changelog

- **v1.0** - Initial comprehensive guide
- **v1.1** - Added troubleshooting section and Russian translation
- **v1.2** - Enhanced with monitoring tools and maintenance scripts
- **v1.3** - Added NVIDIA drivers and performance guide and more

-----

*Last updated: August 2025*
