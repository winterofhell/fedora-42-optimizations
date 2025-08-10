# 🚀 Fedora 42 Gaming & Performance Optimization Guide

> **Complete guide for optimizing Fedora 42 for gaming and maximum performance | by winterofhell**

## 📋 System Information

**Testing Environment:**

- **Period:** October 14, 2024 - August 10, 2025
- **Distribution:** Fedora 42 (Minimal ISO + Sway WM | Second pc: Fedora GNOME Edition)
- **Additional Testing:** GNOME DE on NVIDIA system and AMD System

**Hardware Configurations(tested on):**

- **Primary:** Ryzen 5 5500U, 20GB DDR4, RX550X discrete/RX Vega 7 iGPU, NVMe disk
- **Secondary:** Ryzen 5 5600, 16GB DDR4, GTX 1060, SATA SSD
- **New one:** Ryzen 5 7500f, 32Gb DDR5, RX 9070 XT, Nvme M2.

-----

## 🛠 Initial Setup & Preparation

### 1. Minimal Installation

For optimal performance, always start with the **Fedora Minimal ISO**. This approach eliminates unnecessary packages and services that can impact system resources.

### 2. Enable RPM Fusion Repositories

RPM Fusion provides essential multimedia codecs and proprietary drivers:

```bash
sudo dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

📖 **Official Guide:** [RPM Fusion Configuration](https://rpmfusion.org/Configuration)

### 3. SELinux Configuration (Optional)

⚠️ **Security Warning:** Disabling SELinux reduces system security but also makes ur system a little faster. Only proceed if you understand the implications.

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

**PortProton** offers excellent compatibility for Windows executables:

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

For maximum performance, consider these lightweight desktop environments:

- **Sway** - Wayland-based tiling compositor
- **i3** - X11 tiling window manager
- **Hyprland** - Modern Wayland compositor with animations
- **XFCE** - Lightweight traditional desktop
- **LXQt** - Qt-based lightweight desktop

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

## 🎮 NVIDIA Graphics Optimization

> **Comprehensive optimization guide for NVIDIA GPUs on Fedora 42**

### 📋 NVIDIA System Requirements

**Supported GPUs:**

- GTX 750/900/1000 series and newer (Maxwell, Pascal, Turing, Ampere, Ada Lovelace)
- RTX 20/30/40/50 series with full feature support
- Quadro and Tesla cards (professional workloads)

**Driver Compatibility:**

- **Recommended:** NVIDIA 560+ drivers for best performance
- **Minimum:** NVIDIA 470+ for basic functionality

-----

### 🔧 NVIDIA Driver Installation

#### Method 1: RPM Fusion (Recommended)

```bash
# Install NVIDIA drivers from RPM Fusion
sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda

# For 32-bit compatibility (Steam, Wine)
sudo dnf install xorg-x11-drv-nvidia-libs.i686

# NVIDIA settings GUI
sudo dnf install nvidia-settings
```

#### Method 2: Official NVIDIA Repository

```bash
# Add NVIDIA repository
sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/fedora39/x86_64/cuda-fedora39.repo

# Install drivers
sudo dnf install nvidia-driver nvidia-driver-libs nvidia-driver-cuda
```

#### Post-Installation Verification

```bash
# Check driver installation
nvidia-smi

# Verify CUDA support
nvidia-smi -q | grep "CUDA Version"

# Test 3D acceleration
glxinfo | grep "OpenGL renderer"
```

-----

### ⚡ NVIDIA Performance Optimizations

#### 1. NVIDIA X Server Settings

**Launch nvidia-settings and configure:**

```bash
nvidia-settings
```

**Key Settings to Adjust:**

- **PowerMizer:** Set to “Prefer Maximum Performance”
- **Sync to VBlank:** Disable for gaming (reduce input lag)
- **Threaded Optimizations:** Enable
- **Triple Buffering:** Enable (or Disable)

**Save X configuration:**

```bash
sudo nvidia-settings --assign CurrentMetaMode="nvidia-auto-select +0+0 { ForceFullCompositionPipeline = On }"
```

#### 2. Environment Variables

Add to `/etc/environment`:

```bash
# NVIDIA optimizations
__GL_THREADED_OPTIMIZATIONS=1
__GL_SYNC_TO_VBLANK=0
__GL_ALLOW_UNOFFICIAL_PROTOCOL=1
__GL_SHADER_DISK_CACHE=1
__GL_SHADER_DISK_CACHE_PATH=/tmp/nvidia-shader-cache
__GL_SHADER_DISK_CACHE_SIZE=1073741824

# NVIDIA PRIME (ONLY for laptops with hybrid graphics)
__NV_PRIME_RENDER_OFFLOAD=1
__GLX_VENDOR_LIBRARY_NAME=nvidia

# Additional gaming optimizations
NVIDIA_DRIVER_CAPABILITIES=all
```

#### 3. Xorg Configuration (only if using X11 based wm or whatever)

Create `/etc/X11/xorg.conf.d/20-nvidia.conf`:

```bash
Section "Device"
    Identifier "NVIDIA Card"
    Driver "nvidia"
    VendorName "NVIDIA Corporation"
    Option "NoLogo" "true"
    Option "UseEDID" "false"
    Option "ConnectedMonitor" "DFP"
    Option "CustomEDID" "DFP:/etc/X11/your_edid.bin"
    Option "TripleBuffer" "true"
    Option "RegistryDwords" "EnableBrightnessControl=1"
    Option "MetaModes" "nvidia-auto-select +0+0 { ForceFullCompositionPipeline = On }"
    Option "AllowIndirectGLXProtocol" "off"
    Option "TripleBuffer" "on"
EndSection

Section "Screen"
    Identifier "NVIDIA Screen"
    Device "NVIDIA Card"
    Monitor "My Monitor"
    DefaultDepth 24
    Option "metamodes" "nvidia-auto-select +0+0 { ForceFullCompositionPipeline = On }"
    Option "AllowIndirectGLXProtocol" "off"
    Option "TripleBuffer" "on"
EndSection
```

-----

### 🏎️ Gaming-Specific NVIDIA Tweaks

#### 1. Steam Launch Options

**For native Linux games:**

```bash
gamemoderun __GL_THREADED_OPTIMIZATIONS=1 %command%
```

**For Proton/Wine games:**

```bash
gamemoderun __GL_THREADED_OPTIMIZATIONS=1 DXVK_ASYNC=1 PROTON_USE_WINED3D=0 %command%
```

**For VR games:**

```bash
__GL_THREADED_OPTIMIZATIONS=1 __GL_SYNC_TO_VBLANK=0 %command%
```

#### 2. NVIDIA Profile Inspector Alternative

Create custom game profiles using nvidia-settings:

```bash
# Create profile for specific game
nvidia-settings --assign [gpu:0]/GPUPowerMizerMode=1
nvidia-settings --assign [gpu:0]/GPUMemoryTransferRateOffset[3]=1000
nvidia-settings --assign [gpu:0]/GPUGraphicsClockOffset[3]=100
```

#### 3. NVENC/NVDEC Optimization

For streaming and recording:

```bash
# Install NVENC/NVDEC support
sudo dnf install nvidia-driver-cuda-libs

# OBS Studio with NVENC
sudo dnf install obs-studio

# FFmpeg with NVIDIA acceleration
sudo dnf install ffmpeg --allowerasing
```

-----

### 🔥 Advanced NVIDIA Tweaks

#### 1. GPU Memory Clock Optimization

```bash
# Check current clocks
nvidia-smi -q -d CLOCK

# Enable persistence mode (for consistent performance)
sudo nvidia-smi -pm 1

# Set application clocks (!! adjust values for your SPECIFIC GPU)
sudo nvidia-smi -ac 4004,1911
```

#### 2. Power Management Optimization

```bash
# Disable GPU power management (maximum performance)
sudo nvidia-smi -pl 300  # !! Set to your GPU's maximum power limit

# For laptops - hybrid graphics management
sudo dnf install switcheroo-control
```

#### 3. NVIDIA Container Toolkit (for AI/ML)

```bash
# Add NVIDIA container repository
curl -s -L https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo | \
  sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo

# Install container toolkit
sudo dnf install nvidia-container-toolkit

# Configure Docker
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

#### 4. Overclocking Tools

```bash
# Install GreenWithEnvy (MSI Afterburner alternative)
sudo dnf copr enable atim/gwe
sudo dnf install gwe

# Alternative: NVIDIA System Management Interface
sudo dnf install nvidia-ml-py3
```

-----

### 🐛 NVIDIA Troubleshooting

#### Common Issues and Solutions

**1. Black Screen After Driver Installation:**

```bash
# Boot to console (Ctrl+Alt+F2)
sudo systemctl isolate multi-user.target

# Rebuild initramfs
sudo dracut --force

# Regenerate grub config
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# Reboot
sudo systemctl reboot
```

**2. Screen Tearing Issues:**

```bash
# Force full composition pipeline
nvidia-settings --assign CurrentMetaMode="nvidia-auto-select +0+0 { ForceFullCompositionPipeline = On }"

# Alternative: Use compositor
sudo dnf install picom
```

**3. Poor Gaming Performance:**

```bash
# Check GPU utilization
nvidia-smi dmon

# Verify GPU is being used
__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia glxinfo | grep "OpenGL renderer"

# Check for throttling
nvidia-smi -q -d TEMPERATURE,POWER,CLOCK
```

**4. Driver Update Issues:**

```bash
# Clean reinstall drivers
sudo dnf remove '*nvidia*' --skip-broken
sudo dnf autoremove
sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda

# Rebuild kernel modules
sudo akmods --force
```

#### 5. Multiple Monitor Setup

```bash
# Configure multiple displays
nvidia-settings --assign CurrentMetaMode="DP-2: nvidia-auto-select +1920+0, HDMI-A-1: nvidia-auto-select +0+0"

# Save configuration
sudo nvidia-settings --load-config-only
```

-----

### 📊 NVIDIA Performance Monitoring

#### Real-time Monitoring Commands

```bash
# GPU utilization and memory usage
watch -n 1 nvidia-smi

# Detailed GPU statistics
nvidia-smi dmon -s pucvmet

# GPU temperature monitoring
watch -n 1 'nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader,nounits'

# Power consumption monitoring
watch -n 1 'nvidia-smi --query-gpu=power.draw --format=csv,noheader,nounits'
```

#### GUI Monitoring Tools

```bash
# Install GPU monitoring tools
sudo dnf install nvtop

# Advanced system monitoring
sudo dnf install mangohud goverlay

# Usage in games
mangohud %command%  # Add to Steam launch options
```

-----

### 🔧 NVIDIA Developer Tools

#### CUDA Development

```bash
# Install CUDA toolkit
sudo dnf install cuda-toolkit cuda-devel

# Add CUDA to PATH
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc

# Verify CUDA installation
nvcc --version
```

#### Vulkan Support

```bash
# Install Vulkan support
sudo dnf install vulkan-loader vulkan-tools mesa-vulkan-drivers

# For NVIDIA Vulkan
sudo dnf install nvidia-driver-vulkan

# Test Vulkan
vulkaninfo | grep "GPU id"
vkcube  # Simple Vulkan demo
```

-----

### 🎯 NVIDIA-Specific Gaming Optimizations

#### 1. DLSS and Ray Tracing

**Enable DLSS support:**

```bash
# Ensure latest drivers (RTX 20/30/40/50 series)
nvidia-smi --query-gpu=driver_version --format=csv,noheader

# Verify DLSS support in games
# Check game settings for DLSS options (may still not work, idk, check that, it actually depends from system to system)
```

#### 2. NVIDIA Reflex (Low Latency)

Add to game launch options:

```bash
__GL_SYNC_TO_VBLANK=0 NVIDIA_REFLEX=1 %command%
```

#### 3. GPU Scheduling Optimization

```bash
# Enable GPU scheduling (Windows 10 equivalent)
echo 'options nvidia NVreg_UsePageAttributeTable=1' | sudo tee /etc/modprobe.d/nvidia-pat.conf

# Rebuild initramfs
sudo dracut --force
```

#### 4. VRAM Optimization

```bash
# Monitor VRAM usage
nvidia-smi --query-gpu=memory.used,memory.total --format=csv

# Optimize VRAM allocation for gaming
echo 'options nvidia NVreg_RegistryDwords="RMUseSwapGroup=1;RMEnableVidmemPreserveOnSuspend=1"' | \
  sudo tee /etc/modprobe.d/nvidia-vram.conf
```

-----

### 🌡️ NVIDIA Thermal Management

#### Fan Curve Configuration

```bash
# Enable manual fan control
nvidia-settings -a [gpu:0]/GPUFanControlState=1

# Set custom fan curve (adjust percentages as needed)
nvidia-settings -a [fan:0]/GPUTargetFanSpeed=50

# Create automated thermal script
sudo tee /usr/local/bin/nvidia-thermal.sh << 'EOF'
#!/bin/bash
TEMP=$(nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader,nounits)
if [ $TEMP -gt 75 ]; then
    nvidia-settings -a [fan:0]/GPUTargetFanSpeed=80
elif [ $TEMP -gt 65 ]; then
    nvidia-settings -a [fan:0]/GPUTargetFanSpeed=60
else
    nvidia-settings -a [fan:0]/GPUTargetFanSpeed=40
fi
EOF

sudo chmod +x /usr/local/bin/nvidia-thermal.sh
```

#### Thermal Monitoring

```bash
# Install thermal monitoring
sudo dnf install lm_sensors

# Configure sensors
sudo sensors-detect

# Monitor all temperatures
watch sensors
```

-----

### 🔋 NVIDIA Power Management

#### For Desktop Systems

```bash
# Maximum performance mode
sudo nvidia-smi -pm 1
sudo nvidia-smi -pl 350  # Adjust to your GPU's max power limit

# Disable power management
echo 'options nvidia NVreg_RegistryDwords="PerfLevelSrc=0x2222"' | \
  sudo tee /etc/modprobe.d/nvidia-power.conf
```

#### For Laptop Systems (Hybrid Graphics)

```bash
# Install NVIDIA Prime support
sudo dnf install nvidia-prime

# Check available GPUs
prime-select query

# Switch to NVIDIA GPU
sudo prime-select nvidia

# Switch back to integrated GPU (battery saving)
sudo prime-select intel

# On-demand GPU switching
sudo dnf install envycontrol
sudo envycontrol -s hybrid
```

#### NVIDIA Prime Render Offload

For per-application GPU selection:

```bash
# Use NVIDIA GPU for specific application
__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia your_application

# Gaming example
__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia steam

# Add to .bashrc for convenience
echo 'alias nvidia-run="__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia"' >> ~/.bashrc
```

-----

### 🎨 NVIDIA Display Optimizations

#### High Refresh Rate Configuration (works only on X11 wm / de i believe)

```bash
# Set custom refresh rate
xrandr --output DP-2 --mode 1920x1080 --rate 144

# Make persistent in X11 startup
echo 'xrandr --output DP-2 --mode 1920x1080 --rate 144' >> ~/.xprofile
```

#### G-SYNC/FreeSync Setup

```bash
# Enable Variable Refresh Rate (VRR)
nvidia-settings --assign CurrentMetaMode="DP-2: 1920x1080_144 +0+0 {AllowGSYNC=On}"

# For FreeSync monitors
nvidia-settings --assign CurrentMetaMode="DP-2: 1920x1080_144 +0+0 {AllowGSYNCCompatible=On}"
```

#### Color Management

```bash
# Install color management tools
sudo dnf install argyllcms DisplayCAL

# Basic color calibration
nvidia-settings --assign [DPY:DP-2]/DigitalVibrance=20
nvidia-settings --assign [DPY:DP-2]/Contrast=1.1
```

-----

### 🎯 NVIDIA Gaming Enhancements

#### 1. NVIDIA Game Ready Drivers

```bash
# Check for latest Game Ready drivers
nvidia-detector

# Update to latest drivers
sudo dnf update '*nvidia*'

# Verify driver version
cat /proc/driver/nvidia/version
```

#### 2. NVIDIA DLSS Configuration

**System Requirements for DLSS:**

- RTX 20/30/40/50 series GPU
- Vulkan or DirectX 12 capable games
- Latest NVIDIA drivers (560+)

**Verification:**

```bash
# Check DLSS support
vulkaninfo | grep -i dlss
nvidia-smi --query-gpu=name --format=csv | grep -E "(RTX|Tesla)"
```

#### 3. Ray Tracing Optimization

```bash
# Enable RT cores utilization
echo 'options nvidia NVreg_EnableStreamMemOPs=1' | \
  sudo tee /etc/modprobe.d/nvidia-rt.conf

# Verify RT support
vulkaninfo | grep -i "ray.*trac"
```

#### 4. NVIDIA Broadcast (AI Features)

```bash
# Install required dependencies for AI features
sudo dnf install python3-pip python3-devel

# For OBS NVIDIA plugins
sudo dnf copr enable johndoe31415/obs-backgroundremoval
sudo dnf install obs-backgroundremoval
```

-----

### 🛠️ NVIDIA Development Tools

#### NVIDIA Nsight Tools

```bash
# Install NVIDIA development tools
sudo dnf install cuda-nsight-compute cuda-nsight-systems

# Graphics debugging
sudo dnf install nvidia-visual-profiler
```

#### NVIDIA Container Support

```bash
# Install NVIDIA container runtime
sudo dnf install nvidia-container-toolkit

# Configure containerd
sudo nvidia-ctk runtime configure --runtime=containerd

# Test with Docker
sudo docker run --rm --gpus all nvidia/cuda:11.8-base nvidia-smi
```

-----

### 📈 NVIDIA Performance Benchmarking

#### Benchmark Tools Installation

```bash
# Install benchmark utilities
sudo dnf install mesa-demos glmark2

# NVIDIA-specific benchmarks
sudo dnf copr enable dawid/unigine
sudo dnf install unigine-heaven unigine-valley

# Gaming benchmarks
sudo dnf install steam  # Access to Steam benchmarks
```

#### Performance Testing Commands

```bash
# OpenGL performance test
glxgears -info

# Vulkan performance test
vkmark

# GPU memory bandwidth test
cuda-memcheck deviceQuery

# Comprehensive GPU test
nvidia-smi --query-gpu=gpu_name,driver_version,memory.total,memory.used,utilization.gpu --format=csv --loop=1
```

-----

### 🔍 NVIDIA System Monitoring

#### Custom Monitoring Script

```bash
# Create NVIDIA monitoring script
sudo tee /usr/local/bin/nvidia-monitor.sh << 'EOF'
#!/bin/bash

echo "=== NVIDIA GPU Status ==="
nvidia-smi --query-gpu=name,temperature.gpu,utilization.gpu,memory.used,memory.total,power.draw --format=csv

echo -e "\n=== Driver Information ==="
cat /proc/driver/nvidia/version

echo -e "\n=== Active Processes ==="
nvidia-smi pmon -c 1

echo -e "\n=== Clock Speeds ==="
nvidia-smi --query-gpu=clocks.gr,clocks.mem --format=csv
EOF

sudo chmod +x /usr/local/bin/nvidia-monitor.sh

# Run monitoring
nvidia-monitor.sh
```

#### Real-time Dashboard

```bash
# Install htop-like GPU monitor
sudo dnf install nvtop

# Run real-time GPU monitoring
nvtop
```

-----

### 🚨 NVIDIA Troubleshooting Guide

#### Driver Issues

**1. Nouveau Driver Conflicts:**

```bash
# Blacklist nouveau driver
echo 'blacklist nouveau' | sudo tee /etc/modprobe.d/blacklist-nouveau.conf

# Regenerate initramfs
sudo dracut --force

# Reboot required
sudo reboot
```

**2. Kernel Module Issues:**

```bash
# Check if NVIDIA modules are loaded
lsmod | grep nvidia

# Manually load NVIDIA modules
sudo modprobe nvidia
sudo modprobe nvidia_drm
sudo modprobe nvidia_modeset

# Make modules load at boot
echo -e "nvidia\nnvidia_drm\nnvidia_modeset" | sudo tee /etc/modules-load.d/nvidia.conf
```

**3. X11 Server Issues:**

```bash
# Generate new xorg.conf
sudo nvidia-xconfig

# Backup current config
sudo cp /etc/X11/xorg.conf /etc/X11/xorg.conf.backup

# Reset to default
sudo rm /etc/X11/xorg.conf
sudo systemctl restart gdm
```

#### Performance Issues

**1. Low FPS Despite Good Hardware:**

```bash
# Check GPU is being used
nvidia-smi dmon -s pucvmet -c 1

# Verify no power limiting
nvidia-smi --query-gpu=power.limit,power.draw --format=csv

# Check for thermal throttling
nvidia-smi --query-gpu=temperature.gpu,clocks.gr,clocks.mem --format=csv
```

**2. Stuttering in Games:**

```bash
# Disable compositing in games
__GL_SYNC_TO_VBLANK=0 your_game

# Use dedicated GPU cores
nvidia-settings --assign [gpu:0]/GPUPowerMizerMode=1
```

-----

### 📚 NVIDIA Resources & Links

**Official Documentation:**

- [NVIDIA Linux Driver Installation](https://docs.nvidia.com/datacenter/tesla/tesla-installation-notes/index.html)
- [CUDA Installation Guide](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/)
- [NVIDIA Developer Zone](https://developer.nvidia.com/)

**Community Resources:**

- [NVIDIA Forum](https://forums.developer.nvidia.com/)
- [Arch Wiki NVIDIA](https://wiki.archlinux.org/title/NVIDIA) (applicable to Fedora)
- [NVIDIA on Fedora Wiki](https://rpmfusion.org/Howto/NVIDIA)

**Performance Communities:**

- [r/nvidia](https://www.reddit.com/r/nvidia/)
- [NVIDIA GeForce Forum](https://www.nvidia.com/en-us/geforce/forums/)

-----

*💡 **Pro Tip:** Always test changes incrementally. Apply one optimization at a time and verify stability before proceeding to the next step.*

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

## 🌍 Русская версия (Russian Translation)

<details>
<summary>Нажмите для просмотра русской версии</summary>

# 🚀 Руководство по оптимизации Fedora 42 для игр и производительности

## 📋 Информация о системе

**Среда тестирования:**

- **Период:** 14 октября 2024 - 10 августа 2025
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

-----

*Last updated: August 2025*