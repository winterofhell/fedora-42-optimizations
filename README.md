# fedora-41-optimizations
hi! this is my repo with some optimizations for Fedora that i use for better performance and latency in games/system(tested only on 41 beta, may work on other versions)

for the start:
my english is not so good, but i'll try my best

my fedora linux is on 41 beta version (14.10.2024). for best performance, i recommend using fedora-minimal iso. i also installed my fedora with sway wm spin, instead of gnome.
my system - ryzen 5 5500u, 8gb of ddr4 ram (i need an upgrade Dx) and rx550x discrete/rx vega 7 igpu.

**start optimizing!**

firtly, i enabled rpm fuson repos (very important, u can find info on internet). than, i disabled and deleted SElinux on my system (it can be dangerous in terms of security, but i rly don't need it)

after that, i recommend running "sudo dnf upgrade --refresh" command and than adding cachyos copr and installing cachyos-kernel. but you need to install this kernel only if ur cpu supports x86_64_v3. for cachyos kernel installation —https://copr.fedorainfracloud.org/coprs/bieszczaders/kernel-cachyos/builds/

after successful installation, it'll be good to install UKSMD for more responsive system with cachyos kernel as described here — https://copr.fedorainfracloud.org/coprs/bieszczaders/kernel-cachyos-addons/

okay, we've deleted selinux, installed more perfomant kernel and service. what's next? now, we need to install ananicy-cpp. it's easy - copy github page, cd into it and build. you can ask chatgpt to help you with that task or use official documentation for that — https://gitlab.com/ananicy-cpp/ananicy-cpp. ok, after installing and activating ananicycpp, we can disable some services.
for disabling them, you can use "sudo systemctl disable --now ..." command. services that you can disable: bluetooth.service (if u dont use bluetooth), ModemManager.service, cups.service, avahi-daemon.service, chronyd.service, NetworkManager-wait-online.service, systemd-readahead-collect.service systemd-readahead-replay.service, geoclue.service, smartd.service, upower.service, sshd.service. disabling them will help a little bit with system perfomance.

next, it'll be great to manage kernel params (using grub). sudo nano /etc/default/grub -> GRUB_CMDLINE_LINUX=="quiet lpj=XXXXXXX mitigations=off elevator=noop nowatchdog page_alloc.shuffle=1 pci=pcie_bus_perf intel_idle.max_cstate=1 libahci.ignore_sss=1 noautogroup" in lpj= section you need to replace XXXXXXX with your actual number by getting it from this command — sudo dmesg | grep -o "lpj=\([0-9]*\)". and after all that changes update grub -> sudo grub2-mkconfig -o /boot/grub2/grub.
