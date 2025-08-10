# 🚀 Fedora 42 Gaming & Performance Optimization Guide

> **Complete guide for optimizing Fedora 42 for gaming and maximum performance**

## 📋 System Information

**Testing Environment:**

- **Period:** October 14, 2024 - May 24, 2025
- **Distribution:** Fedora 42 (Minimal ISO + Sway WM spin instead of GNOME)
- **Additional Testing:** GNOME DE on NVIDIA system

**Hardware Configurations:**

- **Primary:** Ryzen 5 5500U, 20GB DDR4, RX550X discrete/RX Vega 7 iGPU, NVMe disk
- **Secondary:** Ryzen 5 5600, 16GB DDR4, GTX 1060, SATA SSD

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

⚠️ **Security Warning:** Disabling SELinux reduces system security. Only proceed if you understand the implications.

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

**Prerequisites:** CPU must support x86_64_v3 instruction set

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
- `elevator=mq-deadline` - Uses deadline I/O scheduler (better for SSDs than noop)
- `nowatchdog` - Disables hardware watchdog
- `intel_idle.max_cstate=1` - Limits CPU idle states for lower latency
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
- **Storage Performance:** Improved SSD performance with TRIM

-----

## 🌍 Русская версия (Russian Translation)

<details>
<summary>Нажмите для просмотра русской версии</summary>

# 🚀 Руководство по оптимизации Fedora 42 для игр и производительности

## 📋 Информация о системе

**Среда тестирования:**

- **Период:** 14 октября 2024 - 24 мая 2025
- **Дистрибутив:** Fedora 42 (Минимальный ISO + Sway WM вместо GNOME)
- **Дополнительное тестирование:** GNOME DE на системе с NVIDIA

**Конфигурации оборудования:**

- **Основная:** Ryzen 5 5500U, 20ГБ DDR4, дискретная RX550X/встроенная RX Vega 7, NVMe диск
- **Вторичная:** Ryzen 5 5600, 16ГБ DDR4, GTX 1060, SATA SSD

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