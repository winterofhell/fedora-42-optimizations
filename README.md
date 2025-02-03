---

Fedora 41 Optimizations for Gaming and Performance

> Tested on Fedora 41‑beta -> Fedora 41 (14.10.2024-01.02.2025) using the Fedora Minimal ISO and the Sway WM spin (instead of GNOME).
System: Ryzen 5 5500U, 8 GB DDR4 (upgrade recommended for me xD), RX550X discrete / RX Vega 7 iGPU, Nvme disk and second pc Ryzen 5 5600, 16Gb DDR4, Gtx 1060, Sata SSD disk.




---

Preparation

1. Minimal Installation:
For the best performance, install Fedora using the Minimal ISO.


2. Enable RPM Fusion Repos:
Ensure you enable RPM Fusion repositories (information available online).


3. Disable SELinux:
If you do not require its security features, disable and remove SELinux (note: this reduces system security).


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

GRUB_CMDLINE_LINUX="quiet lpj=XXXXXXX mitigations=off elevator=noop nowatchdog page_alloc.shuffle=1 pci=pcie_bus_perf intel_idle.max_cstate=1 libahci.ignore_sss=1 noautogroup"

Replace XXXXXXX with the number obtained via:

sudo dmesg | grep -o "lpj=[0-9]*"


2. Update GRUB:

sudo grub2-mkconfig -o /boot/grub2/grub.cfg

Explanation: These parameters disable some mitigation and energy‑saving features (e.g. deep C‑states) to boost performance and reduce latency.




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

RADV_PERFTEST=sam
__GL_THREADED_OPTIMIZATIONS=1
mesa_glthread=true

CPU Scaling Driver Settings:
For AMD:

echo "passive" | sudo tee /sys/devices/system/cpu/amd_pstate/status

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
If you’re not using GNOME, lighter window managers like Sway (as in my case) can reduce idle resource consumption.



---

Conclusion

Fedora is well optimized out of the box with up-to-date packages and a stable system. Applying the above tweaks—disabling unnecessary services, fine‑tuning kernel parameters, and using optimized components (Cachyos kernel, Ananicy‑cpp, Gamemode, etc.)—can further free resources and reduce latency in games and everyday use.


---

Russian Translation

Оптимизации Fedora 41 для игр и производительности

> Тестировалось на Fedora 41‑beta -> Fedora 41 (14.10.2024 - 01.02.2025) с минимальным ISO и Sway (вместо GNOME).
Система: Ryzen 5 5500U, 8 ГБ DDR4 (рекомендуется апгрейд), дискретная видеокарта RX550X / интегрированная RX Vega 7, Nvme диск + Ryzen 5 5600, 16Gb Ddr4, Gtx 1060 и Sata SSD диск.




---

Подготовка

1. Минимальная установка:
Для наилучшей производительности установите Fedora с помощью Minimal ISO.


2. Включите RPM Fusion:
Обязательно включите репозитории RPM Fusion (подробности можно найти в интернете).


3. Отключите SELinux:
Если SELinux вам не нужен, отключите и удалите его (учтите, что это снижает безопасность системы).


4. Обновите систему:

sudo dnf upgrade --refresh




---

Замена ядра

Кэшос‑ядро:
Если ваш CPU поддерживает x86_64_v3, добавьте репозиторий Cachyos COPR и установите Cachyos‑kernel.
[Инструкция по установке Cachyos‑kernel](https://copr.fedorainfracloud.org/coprs/bieszczaders/kernel-cachyos/)

UKSMD аддоны:
После установки Cachyos‑kernel установите UKSMD для повышения отзывчивости.
[UKSMD для Cachyos‑kernel](https://copr.fedorainfracloud.org/coprs/bieszczaders/kernel-cachyos-addons/)



---

Оптимизация служб и процессов

1. Ananicy‑cpp:
Склонируйте, соберите и установите [Ananicy‑cpp](https://gitlab.com/ananicy-cpp/ananicy-cpp) – утилиту для управления приоритетами задач и снижения задержек.


2. Отключение ненужных служб:
Отключите следующие службы, если они вам не нужны:

sudo systemctl disable --now bluetooth.service ModemManager.service cups.service avahi-daemon.service chronyd.service NetworkManager-wait-online.service systemd-readahead-collect.service systemd-readahead-replay.service geoclue.service smartd.service upower.service sshd.service




---

Параметры GRUB

1. Редактирование конфигурации GRUB:
Откройте файл /etc/default/grub и измените строку:

GRUB_CMDLINE_LINUX="quiet lpj=XXXXXXX mitigations=off elevator=noop nowatchdog page_alloc.shuffle=1 pci=pcie_bus_perf intel_idle.max_cstate=1 libahci.ignore_sss=1 noautogroup"

Замените XXXXXXX на число, полученное командой:

sudo dmesg | grep -o "lpj=[0-9]*"


2. Обновите GRUB:

sudo grub2-mkconfig -o /boot/grub2/grub.cfg

Пояснение: Эти параметры отключают некоторые функции энергосбережения и защиты (например, глубокие C‑state), чтобы увеличить производительность и снизить задержки.




---

Дополнительные настройки системы

Включите systemd‑oomd:

sudo systemctl enable --now systemd-oomd

TRIM для SSD:
Включите таймер fstrim и выполните TRIM:

sudo systemctl enable fstrim.timer
sudo fstrim -v /

(Для AMD GPU) Переменные окружения:
Откройте /etc/environment и добавьте:

RADV_PERFTEST=sam
__GL_THREADED_OPTIMIZATIONS=1
mesa_glthread=true

Настройка драйверов масштабирования для CPU:
Для AMD:

echo "passive" | sudo tee /sys/devices/system/cpu/amd_pstate/status

Для Intel:

echo "passive" | sudo tee /sys/devices/system/cpu/intel_pstate/status

Увеличение лимита открытых файлов:
В файле /etc/security/limits.conf добавьте (замените yourusername на ваше имя):

yourusername hard nofile 1048576

(Опционально для Intel) Отключите службу iqrBalance:
Если замечаете задержки с iGPU, попробуйте отключить службу iqrBalance.

Установка Gamemode:

sudo dnf install gamemode

Запускайте игры через gamemoderun для улучшения производительности.

Регулярная очистка кешей:
Используйте такие инструменты, как Stacer или встроенные команды, чтобы периодически очищать кеш DNF и системы.

Запуск Windows‑приложений:
Для игр в формате .exe рекомендую использовать PortProton – отличное решение для совместимости.

Легковесные WM:
Если вы не используете GNOME, оконные менеджеры (например, Sway) могут значительно снизить потребление ресурсов.



---

Заключение

Fedora уже оптимизирована благодаря актуальным пакетам и стабильной системе. Применив вышеописанные настройки – отключив ненужные службы, настроив параметры загрузчика и используя оптимизированные компоненты (Cachyos‑kernel, Ananicy‑cpp, Gamemode и т.д.) – вы сможете дополнительно освободить ресурсы и снизить задержки как в играх, так и в повседневной работе.


---