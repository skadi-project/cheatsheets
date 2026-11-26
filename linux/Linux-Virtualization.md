# Виртуализация Linux: QEMU/KVM и libvirt

Практический стек для запуска Linux и Windows в виртуальных машинах: QEMU/KVM — гипервизор и эмулятор, `libvirt` — API и сервис управления, `virt-manager` — GUI, `virt-viewer` — отдельный SPICE/VNC-клиент. Используйте KVM для нативной производительности на Linux-хосте. Для контейнеров вместо ВМ смотрите Docker/Podman; для desktop-ВМ также подходят GNOME Boxes и VirtualBox.

> Команды ниже рассчитаны на Arch Linux и Debian/Ubuntu. Пути firmware, имена systemd-сервисов и состав пакетов могут отличаться между версиями. Проверяйте локальный список файлов OVMF.

## 1. Что нужно на хосте

| Пакет | Назначение |
|---|---|
| `qemu-full` / `qemu-system-*` | QEMU в пользовательском пространстве и эмуляция устройств; драйвер KVM входит в ядро Linux |
| `libvirt` | API, демоны и управление жизненным циклом ВМ |
| `virt-manager` | GTK-GUI для libvirt |
| `virt-viewer` | SPICE/VNC-клиент консоли ВМ |
| `dnsmasq` | DHCP/DNS для NAT-сетей libvirt |
| `swtpm` | Эмулятор TPM 1.2/2.0 |
| `edk2-ovmf` / `ovmf` | UEFI firmware для x86_64-гостей |
| `dmidecode` | Чтение DMI/SMBIOS хоста при диагностике |
| `iptables-nft` / `iptables` | Совместимость сетевых правил libvirt с nftables |
| `openbsd-netcat` / `netcat-openbsd` | Проверка TCP-сокетов и SPICE/VNC |
| `spice-vdagent` / `spice-vdagent` | Clipboard, resize и интеграция SPICE внутри Linux-гостя |
| `bridge-utils` / `bridge-utils` | Утилиты Linux bridge; для libvirt NAT обычно не нужны |

## 2. Установка

### Arch Linux

```bash
sudo pacman -Syu
sudo pacman -S --needed qemu-full libvirt virt-manager virt-viewer dnsmasq swtpm edk2-ovmf dmidecode iptables-nft openbsd-netcat spice-vdagent bridge-utils virt-install
```

В Arch `virt-install` может входить в пакет `virt-manager` или поставляться отдельно. Если отдельный пакет не найден, проверьте:

```bash
pacman -F /usr/bin/virt-install
```

### Debian

```bash
sudo apt update
sudo apt install qemu-system-x86 qemu-utils libvirt-daemon-system libvirt-clients virt-manager virt-viewer virtinst dnsmasq-base swtpm swtpm-tools ovmf dmidecode iptables netcat-openbsd spice-vdagent bridge-utils
```

Для полноценного `dnsmasq` на хосте он обычно не нужен: libvirt использует `dnsmasq-base` для своей изолированной NAT-сети. Если нужен отдельный DHCP/DNS-сервис, установите `dnsmasq` вместо или вдобавок к `dnsmasq-base`.

### Ubuntu

```bash
sudo apt update
sudo apt install qemu-kvm qemu-utils libvirt-daemon-system libvirt-clients virt-manager virt-viewer virtinst dnsmasq-base swtpm swtpm-tools ovmf dmidecode iptables netcat-openbsd spice-vdagent bridge-utils
```

На Ubuntu пакет `qemu-kvm` — предпочтительный метапакет. Если нужны дополнительные архитектуры и устройства, добавьте:

```bash
sudo apt install qemu-system-x86
```

Проверьте имена пакетов в своей версии:

```bash
apt-cache policy qemu-kvm libvirt-daemon-system ovmf swtpm
```

## 3. Проверка аппаратной виртуализации

Проверьте CPU-флаги:

```bash
lscpu | grep -E 'Virtualization|VT-x|AMD-V'
grep -E -c 'vmx|svm' /proc/cpuinfo
```

`vmx` означает Intel VT-x, `svm` — AMD-V. Ноль совпадений обычно означает выключенную виртуализацию в UEFI/BIOS или недоступность nested virtualization.

Проверьте KVM:

```bash
ls -l /dev/kvm
lsmod | grep -E '^kvm(_intel|_amd)?'
```

Минимальная проверка QEMU:

```bash
qemu-system-x86_64 --version
virsh --version
virt-install --version
```

## 4. Сервис libvirt и права

### Arch Linux

В новых версиях Arch основной вариант — модульный `virtqemud`:

```bash
sudo systemctl enable --now virtqemud.socket
sudo systemctl enable --now virtnetworkd.socket
sudo systemctl enable --now virtstoraged.socket
```

Если в системе используется монолитный демон:

```bash
sudo systemctl enable --now libvirtd.service
```

Не запускайте монолитный и модульные демоны одновременно без необходимости.

### Debian/Ubuntu

```bash
sudo systemctl enable --now libvirtd
```

Проверка:

```bash
systemctl --no-pager --full status libvirtd virtqemud.socket 2>/dev/null
virsh -c qemu:///system list --all
```

Добавьте текущего пользователя в группы:

```bash
sudo usermod -aG libvirt,kvm "$USER"
```

После этого выйдите из графического сеанса и войдите снова либо выполните:

```bash
newgrp libvirt
```

Подключайтесь к системному экземпляру:

```bash
virsh -c qemu:///system uri
```

Ожидаемый URI: `qemu:///system`. Пользовательский `qemu:///session` проще по правам, но хуже подходит для общих сетей, автозапуска и системного управления.

## 5. Сеть

### NAT-сеть libvirt

Проверьте сеть `default`:

```bash
sudo virsh net-list --all
sudo virsh net-start default
sudo virsh net-autostart default
ip addr show virbr0
```

Если сети нет:

```bash
sudo virsh net-define /etc/libvirt/qemu/networks/default.xml
sudo virsh net-start default
sudo virsh net-autostart default
```

Обычно `virbr0` получает адрес `192.168.122.1/24`. `dnsmasq` выдаёт гостю DHCP-адрес, а libvirt делает NAT наружу.

Проверка DHCP/DNS и сокета:

```bash
sudo ss -lntup | grep -E 'dnsmasq|libvirt'
# Проверка ответа DNS (dig: bind в Arch, dnsutils в Debian/Ubuntu)
dig @192.168.122.1 example.org A
# Проверка только TCP-порта, не ответа DNS
nc -vz 192.168.122.1 53
```

### Bridge networking

NAT — безопасный и простой режим. Bridge нужен, если гость должен быть полноценным узлом физической LAN. Не настраивайте bridge поверх Wi-Fi без проверки: многие Wi-Fi-драйверы не поддерживают обычный L2-bridge.

```bash
ip link show
bridge link
sudo brctl show
```

`bridge-utils` — старый набор утилит. Для новых систем предпочтительнее NetworkManager, systemd-networkd или netplan. Создавайте bridge на физическом интерфейсе средствами менеджера сети, затем подключайте интерфейс ВМ к нему через libvirt.

## 6. Диски и ISO

Создайте каталог для ISO и дисков. Первый пример `virt-install` создаст qcow2 сам; не создавайте диск вручную от root с открытыми правами.

```bash
sudo install -d -m 0755 /var/lib/libvirt/images /var/lib/libvirt/boot
```

Положите ISO в каталог образов и проверьте checksum:

```bash
sha256sum /var/lib/libvirt/boot/linux.iso
```

Для Windows заранее скачайте ISO с сайта Microsoft и ISO virtio-драйверов `virtio-win`. Не скачивайте установочные образы из непроверенных источников.

## 7. Linux-гость через virt-install

Пример BIOS/SeaBIOS, NAT и SPICE:

```bash
sudo virt-install \
  --connect qemu:///system \
  --name linux-test \
  --memory 4096 \
  --vcpus 2 \
  --cpu host-passthrough \
  --disk path=/var/lib/libvirt/images/linux.qcow2,size=40,format=qcow2,bus=virtio \
  --cdrom /var/lib/libvirt/boot/linux.iso \
  --network network=default,model=virtio \
  --graphics spice \
  --video virtio \
  --sound ich9 \
  --channel spicevmc \
  --os-variant generic
```

UEFI-вариант:

```bash
sudo virt-install \
  --connect qemu:///system \
  --name linux-uefi \
  --memory 4096 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/linux-uefi.qcow2,size=40,format=qcow2,bus=virtio \
  --cdrom /var/lib/libvirt/boot/linux.iso \
  --network network=default,model=virtio \
  --boot uefi \
  --graphics spice \
  --os-variant generic
```

Если `--boot uefi` не находит firmware, перечислите доступные варианты:

```bash
virsh domcapabilities --virttype kvm | less
find /usr/share/edk2 -type f 2>/dev/null
find /usr/share/OVMF -type f 2>/dev/null
find /usr/share/qemu -iname '*OVMF*' -o -iname '*edk2*' 2>/dev/null
```

Управление:

```bash
virsh -c qemu:///system list --all
virsh -c qemu:///system start linux-test
virsh -c qemu:///system shutdown linux-test
virsh -c qemu:///system destroy linux-test   # аварийное выключение
virsh -c qemu:///system autostart linux-test
```

## 8. Windows-гость

### Windows 10: BIOS/SeaBIOS (устаревший режим)

Используйте ISO Windows 10 (`Win10.iso`). Для Windows 11 переходите к сценарию UEFI + TPM 2.0 ниже.

```bash
sudo virt-install \
  --connect qemu:///system \
  --name windows-bios \
  --memory 8192 \
  --vcpus 4 \
  --cpu host-passthrough \
  --disk path=/var/lib/libvirt/images/windows-bios.qcow2,size=80,format=qcow2,bus=virtio \
  --cdrom /var/lib/libvirt/boot/Win10.iso \
  --disk path=/var/lib/libvirt/boot/virtio-win.iso,device=cdrom \
  --network network=default,model=virtio \
  --graphics spice \
  --video qxl \
  --channel spicevmc \
  --os-variant win10
```

Windows Setup может не видеть VirtIO-диск. Нажмите `Load driver`, откройте ISO virtio-win и выберите `viostor` для `bus=virtio` (для VirtIO SCSI нужен `vioscsi`), затем каталог своей версии Windows и архитектуры `amd64`. Альтернатива для первой установки — `bus=sata`, затем установить VirtIO-драйвер и переключить диск.

### UEFI через OVMF

Создайте ВМ в `virt-manager`: `New VM` → ISO → память/CPU → диск → `Customize configuration before install` → `Overview` → `Firmware` → `UEFI`. Выберите OVMF/edk2 firmware без Secure Boot для Windows 10.

CLI-вариант, если firmware зарегистрирована в libvirt:

```bash
sudo virt-install \
  --connect qemu:///system \
  --name windows-uefi \
  --memory 8192 \
  --vcpus 4 \
  --cpu host-passthrough \
  --disk path=/var/lib/libvirt/images/windows-uefi.qcow2,size=80,format=qcow2,bus=virtio \
  --cdrom /var/lib/libvirt/boot/Win10.iso \
  --disk path=/var/lib/libvirt/boot/virtio-win.iso,device=cdrom \
  --network network=default,model=virtio \
  --boot uefi \
  --graphics spice \
  --video virtio \
  --channel spicevmc \
  --os-variant win10
```

UEFI variables хранятся отдельно от firmware code. Не удаляйте NVRAM-файл после установки: там лежат записи загрузчика.

### TPM 2.0 и Secure Boot

Подключите TPM 2.0 через `virt-manager`: `Add Hardware` → `TPM` → `Type: Emulator` → `Version: 2.0`. В CLI добавьте:

```bash
--tpm backend.type=emulator,backend.version=2.0,model=tpm-crb
```

Для Windows 11 создайте новую ВМ с ISO Windows 11, чипсетом Q35, UEFI и TPM 2.0. Требуются не менее 4 GiB RAM, диск не менее 64 GB и поддерживаемый CPU. В Windows проверьте версию TPM через `tpm.msc`, а режим BIOS и состояние безопасной загрузки — через `msinfo32`.

Secure Boot включается только с firmware OVMF/edk2, которая поддерживает Secure Boot и содержит подходящие ключи. В `virt-manager` это обычно отдельный вариант firmware с пометкой `secure` или переключатель `Secure Boot` в настройках firmware. TPM не включает Secure Boot.

Проверка доступных firmware:

```bash
virsh domcapabilities --virttype kvm | grep -i -A8 -B4 firmware
find /usr/share -type f \( -iname '*OVMF*' -o -iname '*edk2*' \) 2>/dev/null
```

Windows 11 требует UEFI с поддержкой Secure Boot; поддержка прошивкой и фактически включённая безопасная загрузка — разные вещи. Для сценария с включённой проверкой подписи выбирайте прошивку с предустановленными ключами, подходящими для Windows. Если такого варианта нет, проверьте доступные пакеты OVMF для своей версии дистрибутива. Не подменяйте firmware-файлы вручную без резервной копии NVRAM.

## 9. SPICE, virt-viewer и агент

Откройте графическую консоль:

```bash
virt-viewer --connect qemu:///system linux-test
virt-manager --connect qemu:///system
```

В Linux-госте установите агент:

```bash
# Debian/Ubuntu guest
sudo apt update
sudo apt install spice-vdagent

# Arch guest
sudo pacman -S spice-vdagent
```

После входа в графический сеанс проверьте процесс:

```bash
pgrep -a spice-vdagent
```

SPICE channel должен быть добавлен в конфигурацию ВМ. Clipboard и динамический resize зависят от guest agent и desktop environment.

## 10. virt-manager: быстрый сценарий

1. Запустите `virt-manager --connect qemu:///system`.
2. `File` → `New Virtual Machine`.
3. Выберите локальный ISO.
4. Назначьте RAM и CPU.
5. Создайте qcow2-диск.
6. Включите `Customize configuration before install`.
7. Для Linux выберите `VirtIO` disk/network и SPICE.
8. Для Windows добавьте ISO `virtio-win` как второй CD-ROM.
9. Для Windows 11 выберите UEFI, добавьте TPM 2.0 и включите Secure Boot только при наличии secure OVMF.
10. Установите гостевую ОС и проверьте драйверы.

## 11. Диагностика

```bash
virsh -c qemu:///system capabilities >/dev/null
virsh -c qemu:///system dominfo linux-test
virsh -c qemu:///system dumpxml linux-test | less
sudo journalctl -u libvirtd -u virtqemud --since '-15 min'
sudo journalctl -u virtnetworkd --since '-15 min'
sudo ls -l /var/lib/libvirt/dnsmasq/
```

Частые ошибки:

| Ошибка | Причина | Решение |
|---|---|---|
| `Permission denied: /dev/kvm` | Нет прав группы `kvm` | Добавьте пользователя в `kvm`, перелогиньтесь |
| `failed to connect to the hypervisor` | Демон libvirt не запущен или неверный URI | Запустите сервис, используйте `qemu:///system` |
| Гость без сети | Не запущена `default` или сломан `dnsmasq` | `virsh net-start default`, смотрите journal |
| Linux/Windows не видит диск | Нет VirtIO-драйвера | Подключите virtio-win или временно используйте SATA |
| `UEFI` отсутствует | Не установлен OVMF/edk2 или firmware не зарегистрирована | Установите пакет, проверьте `domcapabilities` |
| Windows 11 не проходит проверку | Нет TPM 2.0, UEFI не поддерживает Secure Boot или CPU не подходит | Проверьте все требования в настройках ВМ |
| Clipboard SPICE не работает | Нет `spice-vdagent` или channel | Установите агент, добавьте `spicevmc` channel |
| Bridge не выдаёт IP | Wi-Fi не поддерживает обычный bridge или DHCP не настроен | Используйте NAT или настройте bridge через NetworkManager/netplan |

## 12. Удаление ВМ

Операция необратимо удаляет конфигурацию и, с `--remove-all-storage`, диски. Проверьте имя ВМ и сделайте резервную копию перед удалением.

```bash
virsh -c qemu:///system shutdown windows-uefi
virsh -c qemu:///system domblklist windows-uefi --details
virsh -c qemu:///system domstate windows-uefi
# Дождитесь состояния shut off, затем удалите конфигурацию и NVRAM:
virsh -c qemu:///system undefine windows-uefi --nvram
# Удаляйте qcow2 отдельно только после проверки пути.
```

## Ссылки

- [QEMU Documentation](https://www.qemu.org/docs/master/)
- [libvirt Documentation](https://libvirt.org/docs.html)
- [virt-install man page](https://manpages.debian.org/unstable/virtinst/virt-install.1.en.html)
- [ArchWiki: QEMU](https://wiki.archlinux.org/title/QEMU)
- [ArchWiki: Libvirt](https://wiki.archlinux.org/title/Libvirt)
