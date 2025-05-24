---

Fedora 42 Optimizations for Gaming and Performance

> Tested on Fedora 42 (14.10.2024-24.05.2025) using the Fedora Minimal ISO and the Sway WM spin (instead of GNOME) AND on Gnome DE on Nvidia system.
System: Ryzen 5 5500U, 20 GB DDR4, RX550X discrete / RX Vega 7 iGPU, NVME disk AND second pc - Ryzen 5 5600, 16Gb DDR4, Gtx 1060, Sata SSD disk.




---

Preparation

1. Minimal Installation:
For the best performance, install Fedora using the Minimal ISO.


2. Enable RPM Fusion Repos:
Ensure you enable RPM Fusion repositories.
Follow the official guide at [RPM Fusion Configuration](https://rpmfusion.org/Configuration).



3. Disable SELinux:
If you do not require its security features, disable and remove SELinux (note: this reduces system security).
To temporarily set to permissive mode: sudo setenforce 0

To disable permanently (reboot required):
Edit /etc/selinux/config and change SELINUX=enforcing to SELINUX=disabled.

      
sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
# Reboot for changes to take effect

    


4. Update Your System:

sudo dnf upgrade --refresh



---

Kernel Replacement

Cachyos Kernel:
If your CPU supports x86_64_v3, add the Cachyos COPR repository and install the Cachyos‑kernel.
[Cachyos Kernel Installation](https://copr.fedorainfracloud.org/coprs/bieszczaders/kernel-cachyos/)

UKSMD Addons:
After installing the Cachyos kernel, install UKSMD for increased responsiveness.
[UKSMD Addons for Cachyos](https://copr.fedorainfracloud.org/coprs/bieszczaders/kernel-cachyos-addons/)



---

Service and Process Optimization

1. Ananicy‑cpp:
Clone, build, and install [Ananicy‑cpp](https://gitlab.com/ananicy-cpp/ananicy-cpp) to manage process priorities and reduce latency.


2. Disable Unnecessary Services:
Disable services you don’t use (adjust as needed):

sudo systemctl disable --now bluetooth.service ModemManager.service cups.service avahi-daemon.service chronyd.service NetworkManager-wait-online.service systemd-readahead-collect.service systemd-readahead-replay.service geoclue.service smartd.service upower.service sshd.service

Tip: Disabling unused services can free system resources and improve performance.




---

GRUB Kernel Parameters

1. Edit GRUB Configuration:
Open /etc/default/grub in your editor and modify the kernel parameters:

GRUB_CMDLINE_LINUX="quiet lpj=XXXXXXX mitigations=off elevator=noop nowatchdog page_alloc.shuffle=1 pci=pcie_bus_perf intel_idle.max_cstate=1 libahci.ignore_sss=1 noautogroup amd_pstate=active"

Replace XXXXXXX with the number obtained via:

sudo dmesg | grep -o "lpj=[0-9]*"


2. Update GRUB:

sudo grub2-mkconfig -o /boot/grub2/grub.cfg

Explanation: These parameters disable some mitigation and energy‑saving features (e.g. deep C‑states for Intel cpu or pstate fo Amd cpu !! ) to boost performance and reduce latency.




---

Additional System Tweaks

Enable systemd‑oomd:

sudo systemctl enable --now systemd-oomd

SSD TRIM:
Enable the fstrim timer and run TRIM manually:

sudo systemctl enable fstrim.timer
sudo fstrim -v /

(For AMD GPU Users) Environment Variables:
Open /etc/environment and add:

RADV_PERFTEST=gpl
mesa_glthread=true

CPU Scaling Driver Settings:
For AMD:

echo "active" | sudo tee /sys/devices/system/cpu/amd_pstate/status

For Intel:

echo "passive" | sudo tee /sys/devices/system/cpu/intel_pstate/status

Increase Open File Limit:
In /etc/security/limits.conf, add (replace yourusername with your user name):

yourusername hard nofile 1048576

(Optional for Intel Users) Disable iqrBalance:
If you’re experiencing performance issues with the iGPU, try disabling the iqrBalance service.

Install Gamemode:

sudo dnf install gamemode

Launch games using gamemoderun to benefit from runtime performance adjustments.

Regular Cache Cleaning:
Use tools like Stacer (or CLI commands) to clear DNF and package caches periodically.

For Windows Executables:
Consider using PortProton for launching .exe files with excellent compatibility.

Lightweight WM Recommendations:
If you’re using GNOME / KDE, lighter window managers like Sway / i3 / Hyprland and etc can reduce idle resource consumption.



---

Conclusion

Fedora is well optimized out of the box with up-to-date packages and a stable system. Applying the above tweaks—disabling unnecessary services, fine‑tuning kernel parameters, and using optimized components (Cachyos kernel, Ananicy‑cpp, Gamemode, etc.)—can further free resources and reduce latency in games and everyday use.


---

Russian Translation

---

Оптимизация Fedora 42 для игр и производительности

> Протестировано на Fedora 42 (14.10.2024-24.05.2025) с использованием минимального ISO-образа Fedora и Sway WM (вместо GNOME) И на системе с окружением GNOME и видеокартой Nvidia.
Система: Ryzen 5 5500U, 20 ГБ DDR4, дискретная RX550X / встроенная RX Vega 7, NVME-диск И второй ПК - Ryzen 5 5600, 16 ГБ DDR4, Gtx 1060, SATA SSD-диск.

---

Подготовка

1. Минимальная установка:
Для наилучшей производительности установите Fedora, используя минимальный ISO-образ.

2. Включите репозитории RPM Fusion:
Убедитесь, что вы включили репозитории RPM Fusion.
Следуйте официальному руководству по [настройке RPM Fusion](https://rpmfusion.org/Configuration).

3. Отключите SELinux:
Если вам не нужны его функции безопасности, отключите и удалите SELinux (примечание: это снижает безопасность системы).
Для временного переключения в режим permissive (разрешающий): `sudo setenforce 0`

Для постоянного отключения (требуется перезагрузка):
Отредактируйте `/etc/selinux/config` и измените `SELINUX=enforcing` на `SELINUX=disabled`.

      
sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
# Перезагрузитесь, чтобы изменения вступили в силу

    

4. Обновите вашу систему:

sudo dnf upgrade --refresh

---

Замена ядра

Ядро Cachyos:
Если ваш процессор поддерживает x86_64_v3, добавьте COPR-репозиторий Cachyos и установите ядро Cachyos.
[Установка ядра Cachyos](https://copr.fedorainfracloud.org/coprs/bieszczaders/kernel-cachyos/)

Дополнения UKSMD:
После установки ядра Cachyos установите UKSMD для повышения отзывчивости системы.
[Дополнения UKSMD для Cachyos](https://copr.fedorainfracloud.org/coprs/bieszczaders/kernel-cachyos-addons/)

---

Оптимизация служб и процессов

1. Ananicy-cpp:
Склонируйте, соберите и установите [Ananicy-cpp](https://gitlab.com/ananicy-cpp/ananicy-cpp) для управления приоритетами процессов и снижения задержек.

2. Отключите ненужные службы:
Отключите службы, которые вы не используете (настройте по необходимости):

sudo systemctl disable --now bluetooth.service ModemManager.service cups.service avahi-daemon.service chronyd.service NetworkManager-wait-online.service systemd-readahead-collect.service systemd-readahead-replay.service geoclue.service smartd.service upower.service sshd.service

Совет: Отключение неиспользуемых служб может освободить системные ресурсы и улучшить производительность.

---

Параметры ядра GRUB

1. Отредактируйте конфигурацию GRUB:
Откройте `/etc/default/grub` в вашем редакторе и измените параметры ядра:

GRUB_CMDLINE_LINUX="quiet lpj=XXXXXXX mitigations=off elevator=noop nowatchdog page_alloc.shuffle=1 pci=pcie_bus_perf intel_idle.max_cstate=1 libahci.ignore_sss=1 noautogroup amd_pstate=active"

Замените `XXXXXXX` на число, полученное с помощью команды:

sudo dmesg | grep -o "lpj=[0-9]*"

2. Обновите GRUB:

sudo grub2-mkconfig -o /boot/grub2/grub.cfg

Пояснение: Эти параметры отключают некоторые меры по снижению угроз (митигации) и функции энергосбережения (например, глубокие C-состояния для процессоров Intel или pstate для процессоров AMD!!) для повышения производительности и уменьшения задержек.

---

Дополнительные настройки системы

Включите systemd-oomd:

sudo systemctl enable --now systemd-oomd

TRIM для SSD:
Включите таймер fstrim и запустите TRIM вручную:

sudo systemctl enable fstrim.timer
sudo fstrim -v /

(Для пользователей GPU AMD) Переменные окружения:
Откройте `/etc/environment` и добавьте:

RADV_PERFTEST=gpl
mesa_glthread=true

Настройки драйвера масштабирования частоты CPU:
Для AMD:

echo "active" | sudo tee /sys/devices/system/cpu/amd_pstate/status

Для Intel:

echo "passive" | sudo tee /sys/devices/system/cpu/intel_pstate/status

Увеличьте лимит открытых файлов:
В `/etc/security/limits.conf` добавьте (замените `yourusername` на ваше имя пользователя):

yourusername hard nofile 1048576

(Опционально для пользователей Intel) Отключите iqrBalance:
Если вы испытываете проблемы с производительностью на встроенной графике (iGPU), попробуйте отключить службу iqrBalance.

Установите Gamemode:

sudo dnf install gamemode

Запускайте игры с помощью `gamemoderun`, чтобы воспользоваться преимуществами корректировок производительности во время выполнения.

Регулярная очистка кэша:
Используйте инструменты вроде Stacer (или команды CLI) для периодической очистки кэшей DNF и пакетов.

Для исполняемых файлов Windows:
Рассмотрите использование PortProton для запуска .exe файлов с отличной совместимостью.

Рекомендации по легковесным оконным менеджерам:
Если вы используете GNOME / KDE, более легковесные оконные менеджеры, такие как Sway / i3 / Hyprland и т.д., могут снизить потребление ресурсов в простое.

---

Заключение

Fedora хорошо оптимизирована "из коробки", предлагая актуальные пакеты и стабильную систему. Применение вышеописанных настроек — отключение ненужных служб, тонкая настройка параметров ядра и использование оптимизированных компонентов (ядро Cachyos, Ananicy-cpp, Gamemode и т.д.) — может дополнительно освободить ресурсы и снизить задержки в играх и при повседневном использовании.

---
