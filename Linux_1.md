# Mục lục

[Xem thông tin phần cứng cơ bản](#hardware-information-commands)  

[Quá trình boot Linux](#linux-boot-process)

[So sánh Sysvinit Upstart và Systemd](#sysvinit-upstart-and-systemd)

[Cách dùng Systemd](#systemd)

[Thông tin ổ đĩa cơ bản](#disk-usage-commands)

[Phân vùng ổ đĩa](#partitioning)

[Định dạng file](#filesystem-types)

[LVM (Logical Volume Manager)](#lvm)

[SWAP](#swap)

[GRUB](#grub)

[Shared Library](#shared-library)


# Hardware Information Commands

| Command | Description | How to Read/Use |
|---------|-------------|-----------------|
| lscpu | See CPU information | |
| lsblk | See information about block devices | **RM (Removable)**: nếu là 1 thì đây là thiết bị có thể tháo rời (như USB), 0 là ổ cứng cố định.<br>**RO (Read-Only)**: nếu là 1, thiết bị chỉ có thể đọc (như CD-ROM).<br><br>**Quy ước đặt tên thiết bị**:<br>**sdX** (sda, sdb...): ổ cứng SATA/SCSI/SSD đời cũ — `sda` là ổ đầu tiên, `sda2` là phân vùng thứ hai của ổ đó.<br>**hdX** (hda, hdb...): ổ cứng IDE (cũ) — `hdc2` là ổ thứ ba, phân vùng thứ hai.<br>**nvmeXnY** (nvme0n1...): ổ SSD chuẩn NVMe tốc độ cao.<br>**srX** (sr0, sr1...): ổ đĩa quang CD/DVD — không có phân vùng.<br>**scdX** (scd0...): CD-ROM (cách gọi khác của srX) — không có phân vùng.<br>**vda, vdb**: ổ đĩa ảo trong môi trường máy ảo (KVM/QEMU).<br><br>**Linux Filesystem Layout**:<br>Mọi thứ trong Linux đều là file. Mọi thư mục đều nằm dưới `/` (root), nhưng mỗi thư mục có thể được mount trên một ổ/phân vùng riêng.<br>**Lợi ích**: Tách `/home` ra phân vùng riêng giúp người dùng lưu file thoải mái mà không làm đầy root partition, tránh gây mất ổn định hệ thống. || lspci -tv | Show PCI devices (graphics card, network card, etc.) in a tree-like diagram | Một dòng điển hình: `-[0000:00]-+-11.0-[02]----01.0 Device Name`<br>**-[0000:00]- (Gốc)**: điểm xuất phát từ CPU/Chipset (Bus chính).<br>**+-11.0 (Cầu nối)**: thiết bị trung gian (Bridge) nằm trên Bus chính. Nó giống như một cái "ổ cắm chuyền".<br>**-[02]- (Đường đi mới)**: sau khi đi qua cầu nối, bạn đang ở một con đường mới (Bus 02).<br>**----01.0 (Đích đến)**: thiết bị cuối cùng (Endpoint) cắm vào con đường đó.<br>**Device Name**: tên của thiết bị (Card mạng, Card màn hình, v.v.). |
| lsusb -tv | Display USB devices in a tree-like diagram | Đường đi dữ liệu: thiết bị → port → USB bus → root hub (port 1) → USB controller → PCI bus → CPU<br>**Lưu ý 1**: một USB bus có thể chứa nhiều port, tất cả thiết bị chung một bus sẽ chia sẻ chung băng thông và nguồn điện của bus đó.<br>**Class=...**: Mass Storage (ổ cứng), Human Interface Device (chuột/phím), Hub (bộ chia), ...<br>**5000M**: USB 3.0 (nhanh)<br>**480M**: USB 2.0 (trung bình)<br>**Lưu ý 2**: để xem số port của một bus thì nhìn vào phần Driver port 1, `/[n]p` là có [n] port<br>**Lưu ý 3**: một thiết bị có nhiều chức năng khi cắm vào port sẽ hiển thị nhiều nhánh và có If [ID] khác nhau |
| lshw -C network<br>lshw -C storage<br>lshw -C disk<br>lshw -C processor<br>lshw -C memory | List hardware configuration information | Tập trung xem các phần:<br>**description**: mô tả ngắn gọn thiết bị là gì.<br>**product**: tên model chính xác của linh kiện.<br>**vendor**: nhà sản xuất (Intel, AMD, Realtek...).<br>**logical name**: tên định danh trong hệ thống (ví dụ: ổ cứng là `/dev/sda`, card mạng là `eth0` hoặc `enp0s3`).<br>**configuration**: chứa tên **driver** và các thông số vận hành (speed, latency, width).<br>**Lưu ý**: storage là list storage controller còn disk là list ổ đĩa |
| cat /proc/cpuinfo | Show detailed CPU information | Thông tin từng core trên CPU |
| cat /proc/meminfo | View detailed system memory information | Thông tin RAM |
| cat /proc/mounts | See mounted file systems | **Device**: tên thiết bị hoặc hệ thống tệp ảo được mount (ví dụ: `/dev/sda1`, `proc`, `sysfs`).<br>**Mount Point**: thư mục mà thiết bị đó được gắn vào (ví dụ: `/`, `/home`, `/sys`).<br>**FSType**: loại hệ thống tệp (ví dụ: ext4, xfs, tmpfs, sysfs).<br>**Options**: các tùy chọn khi mount (quyền đọc/ghi, bảo mật...).<br>**Dump**: dùng cho lệnh dump để sao lưu (thường là 0 trong `/proc/mounts`).<br>**Pass**: thứ tự kiểm tra ổ đĩa bằng fsck khi khởi động (thường là 0 ở đây).<br><br>**Lưu ý**: thiết bị thật bắt đầu bằng `/dev`, pseudo filesystems (proc, sysfs, ...) và tmpfs chạy trên RAM không tốn dung lượng ổ cứng. Đọc trên `/proc/mount` **tin cậy hơn** `/etc/mtab` |
| free -h | Display free and used memory | |
| sudo dmidecode | Show hardware information from the BIOS | |
| hdparm -i /dev/[device_name] | Display disk data information | |
| hdparm -tT /dev/[device_name] | Conduct a read speed test on the device/disk | **t**: buffered, kiểm tra xem tốc độ đọc dữ liệu thô của ổ cứng là bao nhiêu.<br>**T**: cached, kiểm tra xem hệ thống quản lý bộ nhớ đệm có nhanh không, giống hệt nhau dù tham số ổ đĩa là khác nhau. |
| badblocks -s /dev/[device_name] | Test for unreadable blocks on the device/disk | |
| fsck /dev/[device_name] | Run a disk check on an unmounted disk or partition | |

# Linux Boot Process

## Giai đoạn 1: Đánh thức phần cứng (Firmware)

**BIOS/UEFI thức dậy**: Đây là phần mềm nằm trong chip trên Mainboard.

**POST (Power-On Self-Test)**: Kiểm tra các linh kiện cơ bản (RAM, CPU, Keyboard).

**Tìm kiếm Mục lục (Partition Table)**:
* BIOS: Tìm bảng MBR (512 byte đầu ổ cứng).
* UEFI: Tìm phân vùng EFI (ESP) định dạng FAT32.

**Chuyển giao**: BIOS/UEFI nạp đoạn mã mồi của Bootloader vào RAM và ra lệnh cho CPU thực thi.

## Giai đoạn 2: Trình khởi động (Bootloader - GRUB)

Lúc này mã lệnh của GRUB đã nằm trên RAM:

**GRUB Stage 1**: Đoạn mã cực nhỏ từ MBR/EFI sẽ tìm và nạp phần còn lại của GRUB từ thư mục /boot/grub trên ổ cứng.

**Menu hiển thị**: Bạn thấy danh sách các Kernel Linux hoặc Windows.

**Lựa chọn**:
* Nếu chọn Windows: GRUB chuyển giao (Chainload) cho bootmgr.efi.
* Nếu chọn Linux: GRUB nạp file Kernel và file initrd vào RAM.

**Kết thúc**: GRUB bàn giao toàn bộ quyền điều khiển RAM cho Kernel.

## Giai đoạn 3: Nhân hệ điều hành & Đội tiền trạm (Kernel & initrd)

Đây là giai đoạn "nhảy" từ RAM vào ổ cứng thật:

**Kernel thực thi**: Tự giải nén và bắt đầu quản lý CPU, RAM.

**initrd (Hệ điều hành mini)**: Vì Kernel chưa có driver ổ cứng, nó sẽ dùng "balo" initrd (đã nạp sẵn trên RAM) để lấy các driver thiết yếu.

**Nhận diện phần cứng**: initrd giúp Kernel "thấy" được ổ cứng thật, card mạng, các phân vùng LVM/RAID.

**Switch Root**: Kernel gắn (mount) phân vùng gốc / từ ổ cứng thật vào hệ thống và xóa initrd khỏi RAM để tiết kiệm chỗ.

## Giai đoạn 4: Tiến trình tổ tiên (Init Process)

Sau khi đã ở trong ổ cứng thật, Kernel gọi "người quản gia" trưởng:

**Chạy /sbin/init**: Đây là tiến trình đầu tiên (PID 1).

**Đọc cấu hình**: init đọc file /etc/inittab (đối với SysVinit) hoặc các file Unit (đối với Systemd hiện đại).

**Xác định Runlevel**: Hệ thống xác định sẽ chạy ở chế độ nào (thường là Runlevel 3 - Server hoặc Runlevel 5 - Desktop).

## Giai đoạn 5: Bật dịch vụ & Đăng nhập (Services & Login)

"Quản gia" init bắt đầu gọi các nhân viên khác dậy theo thứ tự:

**Kiểm tra ổ đĩa (fsck)**: Đảm bảo dữ liệu không bị lỗi.

**Bật Mạng**: Kích hoạt các giao tiếp mạng (IP, Routing).

**Bật Dịch vụ**: Chạy các script khởi động SSH, Database, Web Server...

**Mở cổng TTY**: Kích hoạt các màn hình dòng lệnh (Teletype).

**Login Prompt**: Hiện dòng chữ yêu cầu Username/Password.

# Sysvinit Upstart and Systemd

| Tiêu chí | SysVinit | Upstart | Systemd |
|----------|----------|---------|---------|
| **Tiến trình đầu tiên (PID 1)** | `/sbin/init` | `/sbin/init` (Upstart) | `/lib/systemd/systemd` |
| **Triết lý thiết kế** | Đơn giản, tuần tự, dựa trên script | Hướng sự kiện (event-driven) | Hướng sự kiện + song song + quản lý toàn diện |
| **Cơ chế khởi động** | Tuần tự (sequential) - chạy từng script một theo thứ tự | Bất đồng bộ (asynchronous) - phản ứng theo sự kiện | Song song (parallel) - khởi động nhiều service cùng lúc |
| **Tốc độ khởi động** | Chậm nhất (30-60 giây) | Trung bình (20-40 giây) | Nhanh nhất (5-20 giây) |
| **File cấu hình chính** | `/etc/inittab` | `/etc/init/*.conf` | `/etc/systemd/system/*.service`<br>`/lib/systemd/system/*.service` |
| **Định dạng cấu hình** | Shell script (.sh) | Cú pháp riêng (Upstart job) | Unit file (INI-like format) |
| **Quản lý dịch vụ** | `/etc/init.d/[service] start/stop/restart` | `start/stop/restart [service]`<br>`initctl` | `systemctl start/stop/restart [service]` |
| **Runlevel/Target** | **Runlevel 0**: Halt (tắt máy)<br>• Dừng tất cả services<br>• Unmount filesystems<br>• Tắt nguồn<br><br>**Runlevel 1**: Single-user mode<br>• Chế độ bảo trì<br>• Không có mạng<br>• Chỉ root login<br>• Dùng để sửa lỗi hệ thống<br><br>**Runlevel 2**: Multi-user (Debian/Ubuntu)<br>• Nhiều user đăng nhập được<br>• Có mạng<br>• Không có GUI<br>• Debian: bật đầy đủ services<br><br>**Runlevel 3**: Multi-user + Network<br>• Red Hat/CentOS: chế độ text đầy đủ<br>• Có mạng, có nhiều user<br>• Không có GUI<br>• Chạy server thường dùng level này<br><br>**Runlevel 4**: Không dùng<br>• Dành cho custom của admin<br>• Mặc định giống runlevel 3<br><br>**Runlevel 5**: Graphical mode<br>• Chế độ đầy đủ nhất<br>• Có GUI (X Window/Wayland)<br>• Có mạng, nhiều user<br>• Desktop Linux mặc định<br><br>**Runlevel 6**: Reboot<br>• Khởi động lại máy<br>• Dừng services<br>• Unmount filesystems<br>• Reboot kernel<br><br>**Kiểm tra**: `runlevel`<br>**Chuyển đổi**: `init [số]` hoặc `telinit [số]`<br>**Mặc định**: Thiết lập trong `/etc/inittab`<br>Ví dụ: `id:3:initdefault:` | **Tương thích Runlevel**:<br>Upstart giữ tương thích với SysVinit runlevel nhưng dùng **events**:<br><br>**Runlevel 0** → `runlevel PREVLEVEL=N RUNLEVEL=0`<br>• Event: shutdown<br><br>**Runlevel 1** → `runlevel PREVLEVEL=N RUNLEVEL=1`<br>• Event: single-user<br><br>**Runlevel 2-5** → `runlevel PREVLEVEL=N RUNLEVEL=[2-5]`<br>• Events: multi-user, graphical<br><br>**Runlevel 6** → `runlevel PREVLEVEL=N RUNLEVEL=6`<br>• Event: reboot<br><br>**Cú pháp trong job**:<br>`start on runlevel [2345]`<br>`stop on runlevel [!2345]`<br><br>**Kiểm tra**: `runlevel` hoặc `initctl list`<br>**Chuyển đổi**: `telinit [số]`<br>**Emit event**: `initctl emit [event-name]`<br><br>**Ưu điểm**: Linh hoạt hơn, không phụ thuộc hoàn toàn vào số runlevel | **Systemd Targets** (thay thế runlevel):<br><br>**poweroff.target** (≈ Runlevel 0)<br>• Tắt máy<br>• Alias: `systemctl poweroff`<br><br>**rescue.target** (≈ Runlevel 1)<br>• Single-user mode<br>• Chế độ cứu hộ<br>• Chỉ shell cơ bản<br>• Alias: `systemctl rescue`<br><br>**multi-user.target** (≈ Runlevel 2,3,4)<br>• Chế độ text đa người dùng<br>• Có mạng, đầy đủ services<br>• Không GUI<br>• Dùng cho server<br><br>**graphical.target** (≈ Runlevel 5)<br>• Chế độ đồ họa<br>• Kế thừa multi-user.target<br>• Thêm display manager (GDM, SDDM...)<br>• Desktop mặc định<br><br>**reboot.target** (≈ Runlevel 6)<br>• Khởi động lại<br>• Alias: `systemctl reboot`<br><br>**emergency.target** (mới)<br>• Khẩn cấp hơn rescue<br>• Shell tối thiểu nhất<br>• Filesystem chỉ đọc<br><br>**Kiểm tra**:<br>• `systemctl get-default` - xem target mặc định<br>• `systemctl list-units --type=target` - tất cả targets<br><br>**Chuyển đổi**:<br>• `systemctl isolate [target]` - chuyển ngay<br>• `systemctl set-default [target]` - đặt mặc định<br><br>**Ví dụ**:<br>• `systemctl isolate multi-user.target`<br>• `systemctl set-default graphical.target` |
| **Kiểm tra trạng thái service** | `/etc/init.d/[service] status`<br>`ps aux \| grep [service]` | `status [service]`<br>`initctl status [service]` | `systemctl status [service]` |
| **Quản lý phụ thuộc** | Thủ công qua thứ tự S[số] và K[số] trong script | Tự động dựa trên khai báo events<br>`start on`, `stop on` | Tự động + mạnh mẽ<br>`Requires=`, `After=`, `Before=`, `Wants=` |
| **Xử lý song song** | Không hỗ trợ - chạy tuần tự | Có hỗ trợ cơ bản | Hỗ trợ mạnh mẽ - tối ưu hóa tự động |
| **Theo dõi tiến trình** | Không tích hợp - phải dùng PID file | Tích hợp sẵn - theo dõi qua PID | Tích hợp mạnh mẽ - dùng cgroups để theo dõi chính xác |
| **Logging** | Dùng `/var/log/messages` hoặc syslog | Dùng syslog truyền thống | `journald` - binary log tích hợp<br>Xem bằng `journalctl` |
| **Khởi động lại service khi crash** | Không tự động - cần script riêng | Hỗ trợ qua `respawn` | Hỗ trợ mạnh qua `Restart=`<br>Nhiều tùy chọn: `always`, `on-failure`, `on-abnormal` |
| **Quản lý socket** | Không hỗ trợ | Không hỗ trợ | Socket activation - khởi động service khi có request |
| **Quản lý timer** | Dùng cron riêng biệt | Dùng cron riêng biệt | Tích hợp timer units - thay thế cron<br>`.timer` files |
| **Quản lý mount point** | Dùng `/etc/fstab` + script | Dùng `/etc/fstab` + events | Tích hợp `.mount` units + fstab |
| **Quản lý tài nguyên** | Không hỗ trợ | Không hỗ trợ | Tích hợp cgroups - giới hạn CPU, RAM, I/O |
| **Tương thích ngược** | N/A (đây là hệ thống gốc) | Tương thích với SysVinit scripts | Tương thích với cả SysVinit và Upstart<br>Chạy được legacy scripts |
| **Khả năng mở rộng** | Khó - cần viết shell script phức tạp | Khá tốt - dựa trên events | Rất tốt - nhiều loại units, plugins |
| **Debugging** | Khó - phải đọc log và script | Trung bình - `initctl log-priority` | Dễ - `journalctl`, `systemd-analyze` |
| **Phân tích thời gian boot** | Không tích hợp | Không tích hợp | `systemd-analyze blame`<br>`systemd-analyze critical-chain` |
| **User session management** | Không | Hỗ trợ hạn chế | `systemd --user` - quản lý session của từng user |
| **Network management** | Dùng scripts riêng | Dùng NetworkManager riêng | `systemd-networkd` tích hợp |
| **Ví dụ script/unit** | `#!/bin/bash`<br>`case "$1" in`<br>`  start)`<br>`    /usr/bin/daemon &`<br>`    ;;`<br>`esac` | `description "My Service"`<br>`start on runlevel [2345]`<br>`stop on runlevel [!2345]`<br>`respawn`<br>`exec /usr/bin/daemon` | `[Unit]`<br>`Description=My Service`<br>`After=network.target`<br>`[Service]`<br>`ExecStart=/usr/bin/daemon`<br>`Restart=on-failure`<br>`[Install]`<br>`WantedBy=multi-user.target` |

# Systemd

## Quản lý Service

| Lệnh | Mô tả |
|------|-------|
| `systemctl status <service>` | Xem trạng thái service |
| `systemctl start <service>` | Khởi động service |
| `systemctl stop <service>` | Dừng service |
| `systemctl restart <service>` | Khởi động lại service |
| `systemctl reload <service>` | Reload cấu hình (không restart) |
| `systemctl enable <service>` | Tự động chạy khi boot |
| `systemctl disable <service>` | Tắt tự động chạy |

## Xem thông tin

| Lệnh | Mô tả |
|------|-------|
| `systemctl list-units --type=service` | Liệt kê tất cả services |
| `systemctl list-units --type=service --state=running` | Chỉ xem services đang chạy |
| `systemctl --failed` | Xem services bị lỗi |
| `systemctl is-enabled <service>` | Kiểm tra có auto-start không |
| `systemctl is-active <service>` | Kiểm tra có đang chạy không |

## Xem Log (journalctl)

| Lệnh | Mô tả |
|------|-------|
| `journalctl -u <service>` | Xem log của service |
| `journalctl -u <service> -f` | Theo dõi log real-time |
| `journalctl -u <service> -n 100` | Xem 100 dòng gần nhất |
| `journalctl -u <service> --since today` | Log hôm nay |
| `journalctl -u <service> --since "1 hour ago"` | Log 1 giờ trước |
| `journalctl -p err` | Chỉ xem log lỗi |
| `journalctl -b` | Log từ lần boot hiện tại |

## Quản lý Unit Files

| Lệnh | Mô tả |
|------|-------|
| `systemctl cat <service>` | Xem nội dung unit file |
| `systemctl edit <service>` | Chỉnh sửa service (tạo override) |
| `systemctl daemon-reload` | **Reload sau khi sửa file** |
| `systemctl reset-failed` | Reset trạng thái failed |

## Phân tích Performance

| Lệnh | Mô tả |
|------|-------|
| `systemd-analyze` | Xem thời gian boot |
| `systemd-analyze blame` | Service nào boot lâu nhất |
| `systemd-cgtop` | Xem CPU/RAM của services real-time |

## Quản lý Hệ thống

| Lệnh | Mô tả |
|------|-------|
| `systemctl reboot` | Khởi động lại |
| `systemctl poweroff` | Tắt máy |
| `systemctl rescue` | Vào rescue mode |
| `systemctl get-default` | Xem target mặc định |
| `systemctl set-default multi-user.target` | Đặt chế độ text (no GUI) |
| `systemctl set-default graphical.target` | Đặt chế độ GUI |

## Lưu ý quan trọng

⚠️ **Sau khi sửa file .service**: PHẢI chạy `systemctl daemon-reload`  
⚠️ **Xem log real-time**: Thêm `-f` vào journalctl  
⚠️ **Service file location**: `/etc/systemd/system/` (custom, ưu tiên cao nhất) hoặc `/run/systemd/system/` (ưu tiên trung bình, bay hơi khi reboot) `/lib/systemd/system/` (system default, ưu tiên thấp nhất)


# Disk Usage Commands
Disk usage commands provide insight into disk space status. You can use the `df` and `du` commands to check disk space in Linux.

| Command | Description | How to Read/Use |
|---------|-------------|-----------------|
| `df -h` | Check free and used space on mounted systems | **Filesystem**: tên thiết bị hoặc hệ thống file ảo.<br>**Size**: tổng dung lượng phân vùng.<br>**Used**: dung lượng đã dùng.<br>**Avail**: dung lượng còn trống.<br>**Use%**: % đã dùng — cảnh báo nếu > 80%.<br>**Mounted on**: thư mục được mount vào.<br>**Lưu ý**: `-h` = human-readable, hiển thị KB/MB/GB thay vì bytes. |
| `df -i` | Show free inodes on mounted file systems | **Inode**: mỗi file/thư mục tốn 1 inode, không liên quan đến dung lượng.<br>**IUsed / IFree**: số inode đã dùng / còn trống.<br>**IUse%**: % inode đã dùng.<br>**Lưu ý**: ổ đĩa có thể đầy inode dù vẫn còn dung lượng trống — khi đó không tạo được file mới. |
| `fdisk -l` | Display disk partitions, sizes, and types | **Device**: tên phân vùng (vd: `/dev/sda1`).<br>**Start/End**: vị trí sector bắt đầu và kết thúc.<br>**Sectors / Size**: kích thước phân vùng.<br>**Type**: loại phân vùng (Linux, swap, EFI, NTFS...).<br>**Lưu ý**: cần `sudo` để chạy lệnh này. |
| `du -ah` | See disk usage for all files and directories | **Cột 1**: dung lượng của file/thư mục đó.<br>**Cột 2**: đường dẫn file/thư mục.<br>**Lưu ý**: `-a` = all (hiển thị cả file lẫn thư mục), `-h` = human-readable. Dùng kết hợp `du -ah | sort -rh | head -20` để tìm 20 file/thư mục lớn nhất. |
| `du -sh` | Show disk usage of the current directory | Hiển thị tổng dung lượng của thư mục hiện tại.<br>**Lưu ý**: `-s` = summary, chỉ hiện tổng, không liệt kê chi tiết bên trong. |
| `du -sh *` | Show disk usage of each item in current directory | Hiển thị dung lượng từng file/thư mục con trong thư mục hiện tại.<br>**Lưu ý**: dùng để nhanh chóng xác định thư mục nào đang chiếm nhiều dung lượng nhất. |
| `mount` | Show currently mounted file systems | Mỗi dòng có dạng: `[device] on [mount_point] type [fstype] ([options])`<br>**device**: thiết bị hoặc filesystem ảo.<br>**mount_point**: thư mục gắn vào.<br>**fstype**: loại filesystem (ext4, xfs, tmpfs, vfat...).<br>**options**: `ro` = read-only, `rw` = read-write, `noexec` = không cho chạy file thực thi. |
| `findmnt` | Display target mount point for all file systems | Hiển thị dạng cây, dễ đọc hơn `mount`.<br>**TARGET**: thư mục được mount vào.<br>**SOURCE**: thiết bị nguồn.<br>**FSTYPE**: loại filesystem.<br>**OPTIONS**: các tùy chọn mount.<br>**Lưu ý**: dùng `findmnt [device/mountpoint]` để tìm kiếm nhanh một thiết bị hoặc thư mục cụ thể. |
| `mount [device_path] [mount_point]` | Mount a device | Ví dụ: `mount /dev/sdb1 /mnt/usb`<br>**Lưu ý 1**: thư mục mount point phải tồn tại trước (`mkdir /mnt/usb`).<br>**Lưu ý 2**: thêm `-t [fstype]` nếu muốn chỉ định loại filesystem (vd: `-t ext4`, `-t vfat`).<br>**Lưu ý 3**: mount thủ công sẽ mất sau khi reboot — thêm vào `/etc/fstab` để mount tự động. |
| `umount [device_path]` | Unmount a device | Ví dụ: `umount /dev/sdb1` hoặc `umount /mnt/usb`<br>**Lưu ý 1**: không thể umount nếu có process đang dùng — dùng `lsof [mount_point]` để tìm process đó.<br>**Lưu ý 2**: luôn umount trước khi rút thiết bị để tránh mất dữ liệu. |

---

# Partitioning
## Quy trình chuẩn bị ổ đĩa trong Linux

1. **Lắp đặt vật lý** (hoặc cấp phát ổ đĩa cho VM)
2. **Phân vùng** (fdisk/gdisk/parted)
3. **Định dạng** filesystem (mkfs)
4. **Tạo mount point** (mkdir)
5. **Mount** thiết bị
6. **Thêm vào `/etc/fstab`** để tự động mount khi boot

---

## Quy ước đặt tên thiết bị

| Tên thiết bị | Mô tả | Ví dụ |
|--------------|-------|-------|
| `/dev/hda` | Ổ cứng IDE (cũ) | `/dev/hda1` = ổ IDE đầu tiên, phân vùng 1 |
| `/dev/sda` | Ổ cứng SATA/SCSI/SSD | `/dev/sda1` = ổ SATA đầu tiên, phân vùng 1 |
| `/dev/nvme0n1` | SSD NVMe | `/dev/nvme0n1p1` = SSD NVMe đầu tiên, phân vùng 1 |
| `/dev/vda` | Ổ đĩa ảo (VM) | `/dev/vda1` = ổ ảo đầu tiên, phân vùng 1 |

---

## Loại phân vùng

| Loại | Mô tả | Số thứ tự |
|------|-------|-----------|
| **Primary** (Chính) | Phân vùng độc lập, tối đa 4 phân vùng (chỉ giới hạn trên MBR) | 1-4 |
| **Extended** (Mở rộng) | Container chứa các phân vùng logic, chỉ có 1 trên mỗi ổ (chỉ có trên MBR, không có trên GPT) | Tính vào 1 trong 4 primary |
| **Logical** (Logic) | Phân vùng trong Extended, dùng khi cần >4 phân vùng | Từ 5 trở đi |
| **Swap** | Phân vùng bộ nhớ ảo hỗ trợ RAM | Bất kỳ |

**Khuyến nghị Swap size**:
- RAM < 2GB → Swap = 2 × RAM
- RAM 2-8GB → Swap = RAM
- RAM > 8GB → Swap = 4-8GB

---

## So sánh công cụ phân vùng

| Công cụ | Partition Table | Giới hạn | Tính năng |
|---------|-----------------|----------|-----------|
| **fdisk** | MBR | Phân vùng max 2TB, tối đa 4 primary | Công cụ cơ bản, phổ biến nhất |
| **gdisk** | GPT (GUID) | Phân vùng >2TB, tối đa 128 primary | Thay thế fdisk cho ổ đĩa lớn |
| **parted** | MBR & GPT | Không giới hạn | Resize, tạo/sửa phân vùng, có GUI (gparted) |

---

## fdisk - Công cụ phân vùng MBR

### Lệnh cơ bản

| Lệnh | Mô tả |
|------|-------|
| `fdisk /dev/sda` | Mở fdisk cho ổ đĩa `/dev/sda` |
| `fdisk -l` | Liệt kê tất cả ổ đĩa và phân vùng |
| `fdisk -l /dev/sda` | Xem chi tiết phân vùng của `/dev/sda` |

### Lệnh tương tác (sau khi chạy `fdisk /dev/sda`)

| Phím | Chức năng | Giải thích |
|------|-----------|------------|
| `p` | Print partition table | Xem danh sách phân vùng hiện tại |
| `n` | New partition | Tạo phân vùng mới |
| → `p` | Primary | Tạo primary partition (chọn số 1-4) |
| → `e` | Extended | Tạo extended partition (chọn số 2-4) |
| → `l` | Logical | Tạo logical partition (số từ 5 trở đi) |
| `t` | Change partition type | Đổi loại phân vùng |
| `d` | Delete partition | Xóa phân vùng |
| `w` | Write changes | **Lưu thay đổi** xuống ổ đĩa |
| `q` | Quit without saving | Thoát không lưu |
| `l` | List partition types | Liệt kê mã hex của các loại phân vùng |

### Partition Type Codes (MBR) - **Quan trọng cho thi**

| Mã Hex | Loại phân vùng |
|--------|----------------|
| **82** | Linux swap |
| **83** | Linux (mặc định) |
| **85** | Linux extended |
| **8e** | Linux LVM |
| **fd** | Linux RAID |

### Ví dụ thực tế
```bash
# 1. Xem danh sách phân vùng
fdisk -l /dev/sdb

# 2. Tạo phân vùng mới
fdisk /dev/sdb
# Trong fdisk:
n          # Tạo phân vùng mới
p          # Chọn primary
1          # Số phân vùng 1
[Enter]    # First sector (mặc định)
+10G       # Last sector (tạo phân vùng 10GB)
t          # Đổi type
1          # Chọn phân vùng 1
8e         # Đổi thành Linux LVM
p          # Xem lại
w          # Lưu và thoát

# 3. Thông báo kernel về thay đổi phân vùng
partprobe /dev/sdb
# hoặc
fdisk -l /dev/sdb
```

---

## gdisk - Công cụ phân vùng GPT

### Lệnh cơ bản

| Lệnh | Mô tả |
|------|-------|
| `gdisk /dev/sda` | Mở gdisk cho ổ đĩa `/dev/sda` |
| `gdisk -l /dev/sda` | Xem chi tiết GPT partition table |

### Lệnh tương tác (tương tự fdisk)

| Phím | Chức năng |
|------|-----------|
| `p` | Print partition table |
| `n` | New partition (tạo primary, số 1-128) |
| `t` | Change partition type |
| `d` | Delete partition |
| `w` | Write changes |
| `q` | Quit without saving |
| `?` | Help menu |

### Partition Type Codes (GPT) - **Quan trọng cho thi**

| Mã Hex | Loại phân vùng |
|--------|----------------|
| **8200** | Linux swap |
| **8300** | Linux filesystem (mặc định) |
| **8301** | Linux reserved |
| **8e00** | Linux LVM |
| **fd00** | Linux RAID |

---

## parted - Công cụ phân vùng nâng cao

### Lệnh cơ bản

| Lệnh | Mô tả |
|------|-------|
| `parted /dev/sda` | Mở parted cho `/dev/sda` |
| `parted -l` | Liệt kê tất cả ổ đĩa và phân vùng |
| `parted /dev/sda print` | Xem partition table của `/dev/sda` |

### Lệnh tương tác

| Lệnh | Mô tả | Ví dụ |
|------|-------|-------|
| `help` | Hiển thị trợ giúp | |
| `print` | Xem partition table | |
| `mklabel [type]` | Tạo partition table mới | `mklabel gpt` hoặc `mklabel msdos` |
| `mkpart [name] [start] [end]` | Tạo phân vùng | `mkpart primary 0% 10GB` |
| `rm [number]` | Xóa phân vùng | `rm 1` |
| `quit` | Thoát | |

### Ví dụ thực tế
```bash
# Tạo GPT partition table và phân vùng
parted /dev/sdc
(parted) mklabel gpt              # Tạo GPT table
(parted) mkpart primary 0% 20GB   # Phân vùng 20GB từ đầu ổ
(parted) mkpart primary 20GB 50GB # Phân vùng 30GB tiếp theo
(parted) print                    # Xem kết quả
(parted) quit
```

---

# Filesystem Types

| Filesystem | Mô tả | Khi nào dùng |
|------------|-------|--------------|
| **ext2** | Filesystem Linux cơ bản, không journaling | USB, /boot (hiếm dùng) |
| **ext3** | ext2 + journaling (khôi phục nhanh khi lỗi) | Hệ thống cũ |
| **ext4** | ext3 cải tiến, hiệu suất cao hơn | **Khuyên dùng** cho hầu hết trường hợp |
| **xfs** | Hiệu suất cao với nhiều file nhỏ | Database, server lưu trữ lớn |
| **btrfs** | Tính năng nâng cao (snapshot, compression) | Hệ thống cần snapshot, NAS |
| **ReiserFS** | Journaling filesystem đầu tiên | Ít dùng (developer đã qua đời) |
| **vfat** | FAT32 (DOS/Windows) | USB, chia sẻ với Windows |
| **iso9660** | Filesystem CD-ROM | Tạo file ISO |
| **udf** | Filesystem DVD | Ghi DVD |

---

## mkfs - Tạo Filesystem

### Lệnh cơ bản

| Lệnh | Mô tả |
|------|-------|
| `mkfs -t [type] [device]` | Tạo filesystem loại chỉ định |
| `mkfs.ext4 [device]` | Tạo ext4 (cách viết tắt) |
| `mkfs.xfs [device]` | Tạo xfs |
| `mkswap [device]` | Tạo swap partition |

### Tùy chọn quan trọng

| Tùy chọn | Mô tả | Ví dụ |
|----------|-------|-------|
| `-t [type]` | Loại filesystem | `-t ext4` |
| `-b [size]` | Block size (mặc định 4096 bytes) | `-b 8192` |
| `-m [%]` | % dung lượng dành cho root | `-m 5` (5%) |
| `-L [label]` | Đặt nhãn volume | `-L MyData` |
| `-O [option]` | Tùy chọn đặc biệt | `-O sparse_super` |

### Ví dụ thực tế
```bash
# 1. Tạo ext4 filesystem đơn giản
mkfs.ext4 /dev/sdb1

# 2. Tạo ext4 với tùy chọn nâng cao
mkfs -t ext4 -b 8192 -m 10 -L LargeData -O sparse_super /dev/sde4
# Block size 8KB (cho file lớn)
# 10% dành cho root
# Nhãn "LargeData"
# Ít superblock backup hơn

# 3. Tạo swap
mkswap /dev/sdb2
swapon /dev/sdb2

# 4. Tạo xfs
mkfs.xfs /dev/sdc1

# 5. Tạo vfat (FAT32)
mkfs.vfat /dev/sdd1
```

---

## Superblock & Inode - Khái niệm quan trọng

### Superblock

**Định nghĩa**: Phần đầu của filesystem chứa metadata (kích thước, inode stats, thời gian check cuối).

**Vị trí**: 
- Block đầu tiên + nhiều bản sao ở vị trí khác (để khôi phục)
- **Backup superblock đầu tiên của ext filesystem**: block **8193** ⚠️ **Quan trọng cho thi**

**Khôi phục superblock**:
```bash
# Xem vị trí superblock backup
dumpe2fs /dev/sda1 | grep -i superblock

# Khôi phục từ backup
e2fsck -b 8193 /dev/sda1
```

### Inode

**Định nghĩa**: Cấu trúc dữ liệu chứa metadata của file/thư mục.

**Chứa thông tin**:
- Quyền, owner, group
- Kích thước file
- Timestamps (atime, mtime, ctime)
- Con trỏ đến data blocks
- ⚠️ **KHÔNG chứa tên file** (tên file lưu trong directory entry)

**Lệnh liên quan**:

| Lệnh | Mô tả |
|------|-------|
| `ls -i [file]` | Xem inode number của file |
| `stat [file]` | Xem chi tiết inode |
| `df -i` | Xem số inode available/used trên filesystems |

**Lưu ý**: Số lượng inode **cố định** khi tạo filesystem, không thay đổi được sau đó.
```bash
# Xem inode number
ls -i /var/log/messages
# Output: 123456 /var/log/messages

# Xem chi tiết inode
stat /var/log/messages

# Kiểm tra inode usage
df -i
# Nếu IUse% = 100% → không tạo được file mới dù còn dung lượng trống
```

---

## File cấu hình

### `/etc/mke2fs.conf`

File cấu hình mặc định cho `mke2fs` (tạo ext2/ext3/ext4).

**Nội dung**: Định nghĩa block size, inode ratio, reserved blocks mặc định cho từng loại filesystem.

**Thay đổi hành vi**: Dùng `-O [option]` khi chạy `mkfs` để override cấu hình.

---

## Workflow hoàn chỉnh: Từ ổ đĩa mới đến sử dụng
```bash
# Bước 1: Xem ổ đĩa mới
lsblk
fdisk -l

# Bước 2: Phân vùng (ví dụ dùng fdisk cho MBR)
fdisk /dev/sdb
# n → p → 1 → [Enter] → +50G → t → 1 → 83 → w

# Bước 3: Thông báo kernel
partprobe /dev/sdb

# Bước 4: Tạo filesystem
mkfs.ext4 -L MyData /dev/sdb1

# Bước 5: Tạo mount point
mkdir -p /mnt/mydata

# Bước 6: Mount
mount /dev/sdb1 /mnt/mydata

# Bước 7: Kiểm tra
df -h /mnt/mydata
lsblk

# Bước 8: Tự động mount khi boot (thêm vào /etc/fstab)
echo "/dev/sdb1  /mnt/mydata  ext4  defaults  0  2" >> /etc/fstab

# Hoặc dùng UUID (khuyên dùng)
blkid /dev/sdb1  # Copy UUID
echo "UUID=xxxx-xxxx  /mnt/mydata  ext4  defaults  0  2" >> /etc/fstab

# Bước 9: Test fstab
mount -a  # Mount tất cả trong fstab
```

---

## Lệnh bổ sung quan trọng

| Lệnh | Mô tả |
|------|-------|
| `blkid` | Xem UUID và filesystem type của tất cả partitions |
| `blkid /dev/sdb1` | Xem UUID của partition cụ thể |
| `partprobe [device]` | Thông báo kernel đọc lại partition table |
| `dumpe2fs /dev/sda1` | Xem chi tiết superblock và filesystem info (ext) |
| `tune2fs -l /dev/sda1` | Xem/sửa filesystem parameters (ext) |
| `e2label /dev/sda1 [label]` | Xem/đặt label cho ext filesystem |

---

## Troubleshooting
```bash
# Partition mới tạo không hiển thị?
partprobe /dev/sdb
# hoặc
fdisk -l /dev/sdb

# Filesystem bị lỗi?
fsck /dev/sdb1          # Phải unmount trước
e2fsck -f /dev/sdb1     # Force check cho ext

# Superblock bị hỏng?
dumpe2fs /dev/sdb1 | grep -i superblock  # Tìm backup
e2fsck -b 8193 /dev/sdb1                 # Khôi phục từ backup

# Hết inode?
df -i                   # Kiểm tra IUse%
# → Xóa file không cần thiết hoặc tạo lại filesystem với nhiều inode hơn

# Mount failed?
mount -t ext4 /dev/sdb1 /mnt/test  # Chỉ định rõ filesystem type
dmesg | tail                       # Xem kernel message
journalctl -xe                     # Xem system log
```

---

# LVM

LVM cho phép quản lý ổ đĩa linh hoạt hơn phân vùng truyền thống.

**Cấu trúc 3 tầng**:
```
Physical Volumes (PV) → Volume Groups (VG) → Logical Volumes (LV)
      ổ cứng vật lý          nhóm ổ cứng           phân vùng logic
```

**Ví dụ thực tế**:
- **PV**: `/dev/sda1`, `/dev/sdb1` (các phân vùng vật lý)
- **VG**: `vg_data` (gộp sda1 + sdb1 thành 1 nhóm)
- **LV**: `lv_home`, `lv_var` (chia VG thành các phân vùng logic)

**Lợi ích**:
- ✅ Mở rộng/thu nhỏ phân vùng dễ dàng (không cần reboot)
- ✅ Gộp nhiều ổ cứng thành một
- ✅ Snapshot để backup
- ✅ Di chuyển dữ liệu giữa các ổ đĩa

---

## LVM Commands - Physical Volumes (PV)

| Command | Description | How to Read/Use |
|---------|-------------|-----------------|
| `pvdisplay` | Hiển thị chi tiết PV | **PV Name**: tên thiết bị (vd: `/dev/sda1`).<br>**VG Name**: PV này thuộc VG nào.<br>**PV Size**: dung lượng tổng.<br>**PE Size**: kích thước 1 đơn vị cơ bản (mặc định 4MB).<br>**Free PE**: số đơn vị còn trống. |
| `pvs` | Liệt kê PV ngắn gọn | **PV**: tên PV.<br>**VG**: thuộc VG nào.<br>**Fmt**: định dạng (lvm2).<br>**Attr**: thuộc tính (`a` = allocatable).<br>**PSize / PFree**: tổng dung lượng / còn trống. |
| `pvcreate /dev/sdX` | Tạo PV mới | Ví dụ: `pvcreate /dev/sdb1`<br>**Lưu ý**: phân vùng phải được tạo trước bằng `fdisk` hoặc `parted`. |
| `pvremove /dev/sdX` | Xóa PV | **Lưu ý**: PV phải không thuộc VG nào mới xóa được. Dùng `vgreduce` để gỡ PV ra khỏi VG trước. |
| `pvresize /dev/sdX` | Thay đổi kích thước PV | Dùng sau khi mở rộng phân vùng vật lý. |

---

## LVM Commands - Volume Groups (VG)

| Command | Description | How to Read/Use |
|---------|-------------|-----------------|
| `vgdisplay` | Hiển thị chi tiết VG | **VG Name**: tên VG.<br>**VG Size**: tổng dung lượng.<br>**PE Size**: kích thước 1 PE (Physical Extent).<br>**Total PE / Free PE**: tổng PE / PE còn trống.<br>**VG UUID**: ID duy nhất của VG. |
| `vgs` | Liệt kê VG ngắn gọn | **VG**: tên VG.<br>**#PV**: số PV trong VG này.<br>**#LV**: số LV đã tạo.<br>**VSize / VFree**: tổng dung lượng / còn trống. |
| `vgcreate [vg_name] /dev/sdX /dev/sdY` | Tạo VG mới | Ví dụ: `vgcreate vg_data /dev/sdb1 /dev/sdc1`<br>Gộp nhiều PV thành một VG. |
| `vgextend [vg_name] /dev/sdX` | Thêm PV vào VG | Ví dụ: `vgextend vg_data /dev/sdd1`<br>Mở rộng dung lượng VG. |
| `vgreduce [vg_name] /dev/sdX` | Gỡ PV ra khỏi VG | **Lưu ý**: PV phải không chứa dữ liệu (dùng `pvmove` để di chuyển dữ liệu trước). |
| `vgremove [vg_name]` | Xóa VG | **Lưu ý**: VG phải không chứa LV nào. Xóa hết LV trước bằng `lvremove`. |
| `vgrename [old_name] [new_name]` | Đổi tên VG | Ví dụ: `vgrename vg_data vg_storage` |

---

## LVM Commands - Logical Volumes (LV)

| Command | Description | How to Read/Use |
|---------|-------------|-----------------|
| `lvdisplay` | Hiển thị chi tiết LV | **LV Path**: đường dẫn đầy đủ (vd: `/dev/vg_data/lv_home`).<br>**LV Name**: tên LV.<br>**VG Name**: thuộc VG nào.<br>**LV Size**: dung lượng LV.<br>**LV Status**: `available` hoặc `NOT available`. |
| `lvs` | Liệt kê LV ngắn gọn | **LV**: tên LV.<br>**VG**: thuộc VG nào.<br>**Attr**: thuộc tính (`-wi-ao--` = writable, active, open).<br>**LSize**: dung lượng. |
| `lvcreate -L [size] -n [lv_name] [vg_name]` | Tạo LV mới | Ví dụ: `lvcreate -L 20G -n lv_home vg_data`<br>Tạo LV 20GB tên `lv_home` trong VG `vg_data`. |
| `lvcreate -l 100%FREE -n [lv_name] [vg_name]` | Tạo LV dùng hết dung lượng VG | Ví dụ: `lvcreate -l 100%FREE -n lv_storage vg_data`<br>Dùng toàn bộ dung lượng còn trống. |
| `lvextend -L +[size] /dev/[vg]/[lv]` | Mở rộng LV | Ví dụ: `lvextend -L +10G /dev/vg_data/lv_home`<br>Thêm 10GB vào LV.<br>**Lưu ý**: sau đó phải resize filesystem bằng `resize2fs` (ext4) hoặc `xfs_growfs` (xfs).<br> Có thể gộp extend và resize thành `lvextend -r -L +[size] /dev/[vg]/[lv]` |
| `lvreduce -L -[size] /dev/[vg]/[lv]` | Thu nhỏ LV | Ví dụ: `lvreduce -L -5G /dev/vg_data/lv_home`<br>**NGUY HIỂM**: phải unmount và resize filesystem trước, nếu không sẽ mất dữ liệu.<br> VD: `resize2fs /dev/vg_data/lv-home 15G` (để thu hẹp lại còn 15G)<br> Có thể gộp extend và resize thành `lvreduce -r -L +[size] /dev/[vg]/[lv]` |
| `lvremove /dev/[vg]/[lv]` | Xóa LV | Ví dụ: `lvremove /dev/vg_data/lv_home`<br>**Lưu ý**: phải unmount trước. Dữ liệu sẽ mất hoàn toàn. |
| `lvrename [vg_name] [old_lv] [new_lv]` | Đổi tên LV | Ví dụ: `lvrename vg_data lv_old lv_new` |

---

## LVM - Workflow thực tế

### Tạo LVM từ đầu
```bash
# 1. Tạo phân vùng trên ổ cứng (dùng fdisk hoặc parted)
fdisk /dev/sdb
# Tạo phân vùng mới, chọn type 8e (Linux LVM)

# 2. Tạo Physical Volume
pvcreate /dev/sdb1

# 3. Tạo Volume Group
vgcreate vg_data /dev/sdb1

# 4. Tạo Logical Volume
lvcreate -L 50G -n lv_home vg_data

# 5. Tạo filesystem
mkfs.ext4 /dev/vg_data/lv_home

# 6. Mount
mkdir /home_backup
mount /dev/vg_data/lv_home /home_backup

# 7. Tự động mount khi boot (thêm vào /etc/fstab)
echo "/dev/vg_data/lv_home /home_backup ext4 defaults 0 0" >> /etc/fstab
```

### Mở rộng LV khi hết chỗ
```bash
# 1. Kiểm tra dung lượng còn trống trong VG
vgs

# 2. Mở rộng LV
lvextend -L +20G /dev/vg_data/lv_home

# 3. Mở rộng filesystem (không cần unmount với ext4/xfs)
# Với ext4:
resize2fs /dev/vg_data/lv_home
# Với xfs:
xfs_growfs /home_backup

# 4. Kiểm tra
df -h
```

### Thêm ổ cứng mới vào VG
```bash
# 1. Tạo PV từ ổ mới
pvcreate /dev/sdc1

# 2. Thêm vào VG
vgextend vg_data /dev/sdc1

# 3. Kiểm tra
vgs
pvs
```

---

## LVM Snapshot (Bonus)

| Command | Description | How to Read/Use |
|---------|-------------|-----------------|
| `lvcreate -L [size] -s -n [snap_name] /dev/[vg]/[lv]` | Tạo snapshot | Ví dụ: `lvcreate -L 5G -s -n lv_home_snap /dev/vg_data/lv_home`<br>Tạo snapshot 5GB của `lv_home`.<br>**Dùng để**: backup trước khi update hệ thống. |
| `lvconvert --merge /dev/[vg]/[snap_name]` | Restore từ snapshot | Ví dụ: `lvconvert --merge /dev/vg_data/lv_home_snap`<br>**Lưu ý**: phải unmount LV gốc trước. Snapshot sẽ bị xóa sau khi merge. |
| `lvremove /dev/[vg]/[snap_name]` | Xóa snapshot | Snapshot cũng chiếm dung lượng, nên xóa khi không cần. |

---

## Troubleshooting
```bash

# LV không mount được?
lvchange -ay /dev/vg_data/lv_home  # Kích hoạt LV
mount /dev/vg_data/lv_home /mnt    # Mount thử

# VG không nhận diện được?
vgscan           # Quét lại VG
vgchange -ay     # Kích hoạt tất cả VG
# SWAP

```

---

# SWAP

**Swap** là vùng trên ổ cứng dùng làm RAM ảo khi RAM thật đầy.

**Khi nào dùng Swap?**
- RAM < 2GB → Swap = 2 × RAM
- RAM 2-8GB → Swap = RAM
- RAM > 8GB → Swap = 4-8GB (hoặc không cần)

**Loại Swap**:
- **Swap Partition**: phân vùng riêng (khuyên dùng)
- **Swap File**: file trong filesystem (linh hoạt hơn)

---

## Swap Commands - Swap Partition

| Command | Description | How to Read/Use |
|---------|-------------|-----------------|
| `swapon --show` | Hiển thị swap đang active | **NAME**: tên swap device.<br>**TYPE**: partition hoặc file.<br>**SIZE**: dung lượng.<br>**USED**: đã dùng bao nhiêu.<br>**PRIO**: độ ưu tiên (-2 đến 32767, cao hơn = ưu tiên hơn). |
| `free -h` | Xem tổng RAM và Swap | **Swap**: dòng swap cho biết total/used/free.<br>**Lưu ý**: nếu swap used cao (~80%) thì hệ thống đang thiếu RAM. |
| `mkswap /dev/sdX` | Tạo swap trên phân vùng | Ví dụ: `mkswap /dev/sdb2`<br>**Lưu ý**: phân vùng type phải là 82 (Linux swap) trong fdisk. |
| `swapon /dev/sdX` | Kích hoạt swap | Ví dụ: `swapon /dev/sdb2`<br>Swap sẽ mất sau khi reboot nếu không thêm vào `/etc/fstab`. |
| `swapoff /dev/sdX` | Tắt swap | Ví dụ: `swapoff /dev/sdb2`<br>**Lưu ý**: RAM phải đủ để chứa dữ liệu từ swap, nếu không hệ thống sẽ crash. |
| `swapon -a` | Kích hoạt tất cả swap trong `/etc/fstab` | Dùng sau khi sửa `/etc/fstab` để test. |

---

## Swap Commands - Swap File

| Command | Description | How to Read/Use |
|---------|-------------|-----------------|
| `dd if=/dev/zero of=/swapfile bs=1M count=[size_MB]` | Tạo swap file | Ví dụ: `dd if=/dev/zero of=/swapfile bs=1M count=2048`<br>Tạo file swap 2GB. |
| `fallocate -l [size]G /swapfile` | Tạo swap file nhanh hơn | Ví dụ: `fallocate -l 2G /swapfile`<br>Nhanh hơn `dd` nhưng không hỗ trợ trên một số filesystem. |
| `chmod 600 /swapfile` | Đặt quyền cho swap file | **BẮT BUỘC** vì lý do bảo mật (chỉ root đọc/ghi). |
| `mkswap /swapfile` | Format swap file | Biến file thường thành swap. |
| `swapon /swapfile` | Kích hoạt swap file | Sau đó thêm vào `/etc/fstab` để tự động mount:<br>`/swapfile none swap sw 0 0` |

---

## Swap - Workflow thực tế

### Tạo Swap Partition
```bash
# 1. Tạo phân vùng swap bằng fdisk
fdisk /dev/sdb
# n → tạo phân vùng mới
# t → đổi type thành 82 (Linux swap)
# w → lưu

# 2. Tạo swap
mkswap /dev/sdb2

# 3. Kích hoạt swap
swapon /dev/sdb2

# 4. Kiểm tra
swapon --show
free -h

# 5. Tự động mount khi boot
echo "/dev/sdb2 none swap sw 0 0" >> /etc/fstab
```

### Tạo Swap File (khuyên dùng vì linh hoạt)
```bash
# 1. Tạo file 2GB
fallocate -l 2G /swapfile

# 2. Đặt quyền
chmod 600 /swapfile

# 3. Format swap
mkswap /swapfile

# 4. Kích hoạt
swapon /swapfile

# 5. Kiểm tra
swapon --show
free -h

# 6. Tự động mount khi boot
echo "/swapfile none swap sw 0 0" >> /etc/fstab
```

### Tăng kích thước Swap File
```bash
# 1. Tắt swap
swapoff /swapfile

# 2. Xóa file cũ
rm /swapfile

# 3. Tạo file mới lớn hơn
fallocate -l 4G /swapfile

# 4. Đặt quyền và format
chmod 600 /swapfile
mkswap /swapfile

# 5. Kích hoạt lại
swapon /swapfile
```

### Xóa Swap
```bash
# 1. Tắt swap
swapoff /swapfile

# 2. Xóa dòng trong /etc/fstab
vim /etc/fstab
# Xóa dòng: /swapfile none swap sw 0 0

# 3. Xóa file
rm /swapfile
```

---

## Swappiness - Điều chỉnh mức độ dùng Swap

| Command | Description | How to Read/Use |
|---------|-------------|-----------------|
| `cat /proc/sys/vm/swappiness` | Xem giá trị swappiness hiện tại | **Giá trị**: 0-100<br>**0**: chỉ dùng swap khi RAM cạn kiệt.<br>**10**: ưu tiên RAM, ít dùng swap (khuyên dùng cho server).<br>**60**: mặc định, cân bằng.<br>**100**: ưu tiên dùng swap (chậm). |
| `sysctl vm.swappiness=[value]` | Đặt swappiness tạm thời | Ví dụ: `sysctl vm.swappiness=10`<br>**Lưu ý**: mất sau khi reboot. |
| `echo "vm.swappiness=10" >> /etc/sysctl.conf` | Đặt swappiness vĩnh viễn | Thêm vào file cấu hình để giữ sau reboot. |

---

## Troubleshooting
```bash
# Swap không hoạt động sau reboot?
cat /etc/fstab  # Kiểm tra có dòng swap không

# Hệ thống chậm, swap used cao?
free -h          # Kiểm tra RAM/Swap
top              # Xem process nào ăn RAM
htop             # Giao diện đẹp hơn top

# Xóa swap nhưng vẫn hiển thị?
swapoff -a       # Tắt tất cả swap
swapon --show    # Kiểm tra lại
```

# GRUB

## GRUB là gì?

**GRUB** (Grand Unified Boot Loader) = Trình quản lý khởi động Linux, hiển thị menu chọn hệ điều hành khi máy tính bật.

**Có 2 phiên bản**:
- **GRUB Legacy** (cũ) - CentOS 5, RHEL 5 trở về trước
- **GRUB2** (mới) - Hầu hết distro hiện đại (Ubuntu, CentOS 7+, Fedora...)

---

## So sánh GRUB Legacy vs GRUB2

| Tiêu chí | GRUB Legacy | GRUB2 |
|----------|-------------|-------|
| **File cấu hình chính** | `/boot/grub/menu.lst` hoặc<br>`/boot/grub/grub.conf` | `/boot/grub2/grub.cfg` |
| **Cách sửa cấu hình** | Sửa trực tiếp file menu.lst | **KHÔNG** sửa grub.cfg trực tiếp<br>→ Sửa `/etc/default/grub` + chạy lệnh tạo lại |
| **Tạo lại cấu hình** | Không cần (sửa trực tiếp) | `grub2-mkconfig -o /boot/grub2/grub.cfg` |
| **Cài đặt GRUB** | `grub-install /dev/sda` | `grub2-install /dev/sda` |
| **Độ phức tạp** | Đơn giản, dễ hiểu | Phức tạp hơn, tự động hơn |

---

## GRUB Legacy (Phiên bản cũ)

### Cấu trúc thư mục
```
/boot/
├── grub/
│   ├── menu.lst (hoặc grub.conf)  ← File cấu hình chính
│   ├── stage1                      ← Giai đoạn boot 1
│   └── stage2                      ← Giai đoạn boot 2
├── vmlinuz-X.X.X                   ← Kernel
├── initrd-X.X.X.img                ← Initial RAM disk
└── System.map-X.X.X                ← Symbol map
```

### File cấu hình: `/boot/grub/menu.lst`
```bash
# Cấu hình chung
default=0        # Chọn mục menu thứ 0 (đầu tiên) làm mặc định
timeout=10       # Chờ 10 giây trước khi tự động boot

# Mục menu thứ nhất (CentOS)
title CentOS (2.6.32-71.el6.x86_64)
    root (hd0,0)                                    # ổ đĩa 0, phân vùng 0
    kernel /boot/vmlinuz-2.6.32 root=LABEL=/ ro    # Đường dẫn kernel
    initrd /boot/initrd-2.6.32.img                 # Initial ramdisk

# Mục menu thứ hai (Windows)
title Windows 10
    rootnoverify (hd0,1)
    chainloader +1
```

**Giải thích các dòng**:

| Dòng | Ý nghĩa |
|------|---------|
| `default=0` | Boot mục menu đầu tiên (đếm từ 0) |
| `timeout=10` | Hiển thị menu 10 giây, sau đó tự động boot |
| `title [tên]` | Tên hiển thị trên menu |
| `root (hd0,0)` | **hd0** = ổ cứng đầu tiên, **0** = phân vùng đầu tiên<br>(⚠️ Đếm từ 0, khác với Linux `/dev/sda1`) |
| `kernel /boot/vmlinuz...` | Đường dẫn đến kernel |
| `root=LABEL=/` | Phân vùng root có label `/` |
| `ro` | Mount root ở chế độ read-only ban đầu |
| `initrd /boot/initrd...` | Đường dẫn đến initial ramdisk |

**Cách đánh số ổ đĩa trong GRUB Legacy**:

| GRUB Legacy | Linux | Giải thích |
|-------------|-------|------------|
| `(hd0,0)` | `/dev/sda1` | Ổ đầu tiên, phân vùng đầu tiên |
| `(hd0,1)` | `/dev/sda2` | Ổ đầu tiên, phân vùng thứ hai |
| `(hd1,0)` | `/dev/sdb1` | Ổ thứ hai, phân vùng đầu tiên |

### Lệnh GRUB Legacy

| Lệnh | Mô tả | Ví dụ |
|------|-------|-------|
| `grub-install [device]` | Cài đặt GRUB vào MBR | `grub-install /dev/sda` |
| `grub` | Vào GRUB shell để test | `grub`<br>`> root (hd0,0)`<br>`> setup (hd0)` |

### Sửa menu boot tạm thời (Khi đang boot)

1. Khi menu GRUB hiện ra, nhấn **`e`** trên mục muốn sửa
2. Chỉnh sửa dòng `kernel` hoặc `initrd`
3. Nhấn **`b`** để boot với thay đổi tạm thời
4. Hoặc nhấn **`c`** để vào GRUB command line

---

## GRUB2 (Phiên bản mới) - **Khuyên học phần này**

### Cấu trúc thư mục
```
/boot/
├── grub2/
│   ├── grub.cfg              ← File menu (TỰ ĐỘNG TẠO, KHÔNG SỬA TRỰC TIẾP)
│   ├── fonts/                ← Font chữ cho menu
│   ├── themes/               ← Giao diện menu
│   └── efi/                  ← UEFI boot files
├── vmlinuz-X.X.X             ← Kernel
├── initramfs-X.X.X.img       ← Initial RAM filesystem
└── System.map-X.X.X

/etc/
├── default/
│   └── grub                  ← **SỬA FILE NÀY** để thay đổi GRUB
└── grub.d/
    ├── 00_header
    ├── 10_linux              ← Tự động tìm kernel Linux
    ├── 30_os-prober          ← Tự động tìm Windows/OS khác
    └── 40_custom             ← Thêm mục menu tùy chỉnh
```

### File cấu hình: `/etc/default/grub` - **Quan trọng nhất**
```bash
GRUB_TIMEOUT=5                          # Chờ 5 giây
GRUB_DEFAULT=0                          # Boot mục đầu tiên
GRUB_DISABLE_SUBMENU=true               # Không dùng submenu
GRUB_TIMEOUT_STYLE=menu                 # Hiển thị menu (hoặc hidden)
GRUB_CMDLINE_LINUX="crashkernel=auto"   # Tùy chọn kernel
GRUB_DISABLE_RECOVERY="true"            # Ẩn chế độ recovery
```

**Các tham số quan trọng**:

| Tham số | Giá trị | Ý nghĩa |
|---------|---------|---------|
| `GRUB_TIMEOUT` | `5` | Hiển thị menu 5 giây |
| | `0` | Boot ngay, không hiển thị menu |
| | `-1` | Chờ mãi mãi cho đến khi chọn |
| `GRUB_DEFAULT` | `0` | Boot mục đầu tiên |
| | `saved` | Nhớ lựa chọn lần trước |
| | `"CentOS Linux"` | Boot mục có tên này |
| `GRUB_CMDLINE_LINUX` | `"quiet splash"` | Ẩn log, hiển thị splash screen |
| | `"text"` | Boot vào text mode (không GUI) |
| | `"single"` | Boot vào single-user mode |

### Quy trình thay đổi GRUB2 - **3 bước**
```bash
# Bước 1: Sửa file cấu hình
sudo vim /etc/default/grub
# Thay đổi GRUB_TIMEOUT=5 thành GRUB_TIMEOUT=10

# Bước 2: Tạo lại file grub.cfg
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
# Trên Ubuntu/Debian:
sudo update-grub

# Bước 3: Reboot để kiểm tra
sudo reboot
```

### Lệnh GRUB2

| Lệnh | Mô tả | Ví dụ |
|------|-------|-------|
| `grub2-mkconfig -o [file]` | Tạo file grub.cfg từ<br>/etc/default/grub và /etc/grub.d/ | `grub2-mkconfig -o /boot/grub2/grub.cfg` |
| `update-grub` | (Ubuntu/Debian)<br>Tương đương grub2-mkconfig | `update-grub` |
| `grub2-install [device]` | Cài đặt GRUB2 vào MBR | `grub2-install /dev/sda` |
| `grub2-set-default [number]` | Đặt mục menu mặc định | `grub2-set-default 0` |
| `grub2-editenv list` | Xem biến môi trường GRUB | `grub2-editenv list` |

### Thêm mục menu tùy chỉnh

**File**: `/etc/grub.d/40_custom`
```bash
#!/bin/sh
exec tail -n +3 $0

# Thêm mục boot vào Windows
menuentry "Windows 10" {
    insmod ntfs
    set root='(hd0,2)'
    chainloader +1
}

# Thêm mục boot vào kernel cũ hơn
menuentry "CentOS Linux (3.10.0-old)" {
    linux /boot/vmlinuz-3.10.0-old root=/dev/sda1 ro
    initrd /boot/initramfs-3.10.0-old.img
}
```

**Sau khi sửa**:
```bash
chmod +x /etc/grub.d/40_custom
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### Sửa menu boot tạm thời (Khi đang boot)

1. Khi menu GRUB2 hiện ra, nhấn **`e`** trên mục muốn sửa
2. Tìm dòng bắt đầu với `linux` hoặc `linux16`
3. Thêm/sửa tham số (ví dụ: thêm `single` để vào single-user mode)
4. Nhấn **`Ctrl+X`** hoặc **`F10`** để boot với thay đổi tạm thời
5. Hoặc nhấn **`c`** để vào GRUB2 command line

---

## Các tình huống thực tế

### 1. Quên mật khẩu root - Reset bằng GRUB2
```bash
# Trong menu GRUB, nhấn 'e' trên mục Linux
# Tìm dòng bắt đầu với 'linux16' hoặc 'linux'
# Thêm vào cuối dòng:
rd.break enforcing=0
#       rd.break: dừng quá trình boot ở giai đoạn Initramfs (Initial RAM Filesystem) = Hệ thống file tạm thời trong RAM
#       - Tại thời điểm này, hệ thống **chưa yêu cầu password**
#       - Bạn được vào **emergency shell** (shell cấp cứu) với quyền root
#       - Ổ cứng thật đã được mount vào `/sysroot` nhưng **chỉ ở chế độ read-only**
#       enforcing=0: Tắt SELinux ở chế độ enforcing, chỉ để warning
#       - Security-Enhanced Linux = Hệ thống bảo mật bổ sung
#       - Có 3 chế độ: enforcing (Chặn mọi hành động vi phạm policy), permissive (Chỉ cảnh báo, không chặn), disabled (Tắt hoàn toàn)
#       Khi bạn đổi password → File /etc/shadow thay đổi → SELinux context (nhãn bảo mật) của file bị sai → Lần boot sau, SELinux sẽ CHẶN đăng nhập → Bạn lại bị khóa ngoài!


# Nhấn Ctrl+X để boot
# Khi vào emergency shell:
mount -o remount,rw /sysroot
#       /sysroot: Thư mục tạm trong initramfs, chứa ổ cứng thật của bạn đã được mount.
#       Initramfs (filesystem tạm trong RAM)
#             ├── /bin
#             ├── /sbin
#             ├── /sysroot  ← Ổ cứng thật được mount VÀO ĐÂY
#             │    ├── /bin
#             │    ├── /etc
#             │    ├── /home
#             │    ├── /root
#             │    └── ...
#             └── ...
#       Nếu bạn chạy passwd root lúc này:
#       - Lệnh passwd sẽ tìm file /etc/shadow
#       - Nhưng /etc/shadow trong initramfs KHÔNG PHẢI file thật
#       - File thật là /sysroot/etc/shadow

# Phải chuyển thư mục root từ /sysroot về /
chroot /sysroot
passwd root
# Do SELinux không hoạt động lúc này, file /etc/shadow sau khi bị ghi đè sẽ bị mất nhãn bảo mật hoặc mang một cái nhãn sai (nhãn của môi trường cứu hộ).
touch /.autorelabel # ← tạo một file .autolabel để khi khởi động máy yêu cầu SELinux gán nhãn lại toàn bộ hệ thống
exit # ← Thoát khỏi chroot
exit # ← Thoát khỏi emergency shell
```

### 2. Thay đổi timeout menu
```bash
# Sửa file
sudo vim /etc/default/grub
# Đổi thành:
GRUB_TIMEOUT=10

# Tạo lại grub.cfg
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 3. Ẩn menu GRUB (boot thẳng)
```bash
# Sửa file
sudo vim /etc/default/grub
# Đổi thành:
GRUB_TIMEOUT=0
GRUB_TIMEOUT_STYLE=hidden

# Tạo lại grub.cfg
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 4. Cài lại GRUB khi bị lỗi
```bash
# Boot từ USB/LiveCD
# Mount phân vùng root
sudo mount /dev/sda1 /mnt
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys

# Chroot vào hệ thống
sudo chroot /mnt

# Cài lại GRUB
grub2-install /dev/sda
grub2-mkconfig -o /boot/grub2/grub.cfg

# Thoát và reboot
exit
sudo reboot
```

### 5. Dual boot với Windows

GRUB2 tự động phát hiện Windows nếu cài `os-prober`:
```bash
# Cài os-prober
sudo yum install os-prober      # CentOS/RHEL
sudo apt install os-prober      # Ubuntu/Debian

# Tạo lại grub.cfg
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# Windows sẽ tự động xuất hiện trong menu
```

---

## Thư mục `/boot` - Chứa gì?

| File/Thư mục | Mô tả |
|--------------|-------|
| `vmlinuz-X.X.X` | **Kernel** - Nhân Linux đã nén |
| `initrd-X.X.X.img` (Legacy)<br>`initramfs-X.X.X.img` (GRUB2) | **Initial RAM Disk** - Filesystem tạm trong RAM để boot |
| `System.map-X.X.X` | **Symbol Map** - Danh sách địa chỉ các hàm kernel |
| `config-X.X.X` | **Kernel Config** - Cấu hình kernel khi compile |
| `grub/` hoặc `grub2/` | Thư mục cấu hình GRUB |

---

## Troubleshooting

### GRUB không hiển thị menu?
```bash
# Kiểm tra timeout
grep GRUB_TIMEOUT /etc/default/grub

# Nếu =0 hoặc hidden, đổi thành:
GRUB_TIMEOUT=5
GRUB_TIMEOUT_STYLE=menu

# Tạo lại
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### GRUB bị lỗi "error: file not found"?
```bash
# Thường do đường dẫn sai trong grub.cfg
# Kiểm tra kernel có tồn tại không
ls /boot/vmlinuz*

# Tạo lại grub.cfg
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### Dual boot mất Windows sau khi cài Linux?
```bash
# Cài os-prober
sudo yum install os-prober

# Tạo lại menu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

### Lỗi "GRUB" hoặc "grub rescue>"?
```bash
# Boot từ LiveCD/USB
# Mount và chroot như hướng dẫn ở trên
# Sau đó:
grub2-install /dev/sda
grub2-mkconfig -o /boot/grub2/grub.cfg
```

---

# Shared Library

## 1. Khái niệm cơ bản

### Shared Library là gì?

**Định nghĩa**: Code dùng chung giữa nhiều chương trình, không nhúng vào từng chương trình.

**Đặc điểm**:
- Tên file: `libname.so.version` (vd: `libc.so.6`, `libssl.so.1.1`)
- `.so` = Shared Object
- Tiết kiệm RAM và ổ cứng

**So sánh**:
```
KHÔNG dùng Shared Library:
App A (10MB) = Code (1MB) + Libs (9MB)
App B (10MB) = Code (1MB) + Libs (9MB)
App C (10MB) = Code (1MB) + Libs (9MB)
Tổng: 30MB

DÙNG Shared Library:
App A (1MB) ─┐
App B (1MB) ─┼→ Libs (9MB chung)
App C (1MB) ─┘
Tổng: 12MB
```

---

## 2. Static vs Dynamic Linking

| | Static Linking | Dynamic Linking |
|---|----------------|-----------------|
| **Code thư viện** | Nhúng vào executable | Tách riêng, load khi chạy |
| **Kích thước** | Lớn (10-50MB) | Nhỏ (vài KB-MB) |
| **RAM** | Mỗi app 1 copy | Nhiều app dùng chung |
| **Phụ thuộc** | Độc lập | Cần thư viện hệ thống |
| **Cập nhật** | Phải compile lại | Tự động |
| **Khi dùng** | Embedded, container | 99% chương trình Linux |

---

## 3. Dynamic Linking - 2 loại

### 3.1. Load-time Linking (Phổ biến nhất)

**Load khi nào**: Ngay khi chương trình khởi động, **trước** khi `main()` chạy.

**Quy trình**:
```
./myapp
  ↓
Kernel load program
  ↓
Dynamic linker (ld.so) tìm + load TẤT CẢ thư viện
  ↓
Link các function
  ↓
Chạy main()
```

**Đặc điểm**:
- ✅ Đơn giản, tự động
- ✅ Lỗi thiếu lib → Phát hiện ngay khi khởi động
- ❌ Khởi động chậm hơn nếu nhiều lib
- ❌ Tốn RAM cho lib không dùng

**Xem load-time libraries**:
```bash
ldd /bin/ls
# Tất cả lib ở đây đều load ngay khi khởi động
```

---

### 3.2. Run-time Linking (Plugin, advanced)

**Load khi nào**: Trong lúc chương trình chạy, **khi cần**.

**Quy trình**:
```
main() chạy
  ↓
Gặp dlopen("libplugin.so")
  ↓
Load thư viện vào RAM
  ↓
dlsym() tìm function
  ↓
Gọi function
  ↓
dlclose() giải phóng (tuỳ chọn)
```

**Đặc điểm**:
- ✅ Khởi động nhanh (chỉ load khi cần)
- ✅ Tiết kiệm RAM
- ✅ Linh hoạt (plugin system)
- ❌ Phức tạp (phải code thủ công)
- ❌ Lỗi thiếu lib → Chỉ phát hiện khi chạy đến dòng đó

**Ứng dụng**: Firefox extensions, Apache modules, GIMP plugins

**Code ví dụ**:
```c
#include <dlfcn.h>

void* handle = dlopen("libplugin.so", RTLD_LAZY);
void (*func)() = dlsym(handle, "my_function");
func();
dlclose(handle);
```

**Lưu ý**: `ldd` KHÔNG nhìn thấy run-time libraries

---

## 4. Cơ chế tìm kiếm thư viện

### 4.1. Thứ tự ưu tiên (Cao → Thấp)
```
Program cần libfoo.so.1

1️⃣ LD_LIBRARY_PATH          ← Biến môi trường (CAO NHẤT)
   Tìm thấy? → DÙNG
   Không? ↓

2️⃣ /etc/ld.so.cache          ← Cache (NHANH NHẤT)
   Tìm thấy? → DÙNG
   Không? ↓

3️⃣ Thư mục mặc định           ← Fallback
   /lib, /lib64
   /usr/lib, /usr/lib64
   Tìm thấy? → DÙNG
   Không? ↓

❌ ERROR: cannot open shared object file
```

**⚠️ Lưu ý**: `LD_LIBRARY_PATH` ưu tiên cao nhất → Nguy hiểm nếu dùng sai!

---

### 4.2. Dynamic Linker (ld.so)

**Là gì**: Chương trình đặc biệt load và link thư viện khi program chạy.

**Vị trí**:
- 64-bit: `/lib64/ld-linux-x86-64.so.2`
- 32-bit: `/lib/ld-linux.so.2`

**Nhiệm vụ**:
1. Đọc executable → Lấy danh sách lib cần
2. Tìm lib theo thứ tự ưu tiên
3. Load lib vào RAM
4. Link các function
5. Chuyển quyền cho `main()`

**Debug**:
```bash
LD_DEBUG=libs ./myapp 2>&1 | grep libfoo
# Xem chi tiết quá trình tìm + load
```

---

## 5. Cấu hình đường dẫn thư viện

### 5.1. `/etc/ld.so.conf` + `ldconfig` (Khuyên dùng)

**Mục đích**: Cấu hình vĩnh viễn, toàn hệ thống.

**Cấu trúc**:
```
/etc/ld.so.conf              ← File chính (thường chỉ include)
└── /etc/ld.so.conf.d/       ← Thư mục config
    ├── mysql.conf → /usr/lib64/mysql
    ├── local.conf → /usr/local/lib
    └── myapp.conf → /opt/myapp/lib
```

**Quy trình**:
```bash
# 1. Thêm đường dẫn
echo "/opt/myapp/lib" | sudo tee /etc/ld.so.conf.d/myapp.conf

# 2. Cập nhật cache (BẮT BUỘC)
sudo ldconfig

# 3. Kiểm tra
ldconfig -p | grep myapp
```

**Ưu điểm**:
- ✅ Vĩnh viễn (tồn tại sau reboot)
- ✅ Nhanh (dùng cache)
- ✅ Áp dụng toàn hệ thống

**Nhược điểm**:
- ❌ Cần quyền root
- ❌ Phải chạy `ldconfig` sau mỗi lần sửa

---

### 5.2. `LD_LIBRARY_PATH` (Chỉ dùng test)

**Mục đích**: Cấu hình tạm thời, không cần root.

**Cú pháp**:
```bash
export LD_LIBRARY_PATH=/opt/lib1:/opt/lib2:$LD_LIBRARY_PATH
```

**Ưu điểm**:
- ✅ Không cần root
- ✅ Test nhanh
- ✅ Không ảnh hưởng hệ thống

**Nhược điểm**:
- ❌ Tạm thời (mất sau khi đóng terminal)
- ❌ Ưu tiên CAO NHẤT → Nguy hiểm
- ❌ Không dùng cache → Chậm

**⚠️ Cảnh báo**: Chỉ dùng để test, **KHÔNG** dùng trong production!

---

### 5.3. So sánh

| | `/etc/ld.so.conf` | `LD_LIBRARY_PATH` |
|---|-------------------|-------------------|
| **Quyền** | Cần root | Không cần |
| **Phạm vi** | Toàn hệ thống | Phiên terminal |
| **Tốc độ** | Nhanh (cache) | Chậm hơn |
| **Ưu tiên** | Thấp | **CAO NHẤT** |
| **Vĩnh viễn** | Có | Không |
| **Production** | ✅ Dùng | ❌ Không dùng |
| **Test/Debug** | ⚠️ Cần root | ✅ Dùng |

---

## 6. Lệnh quan trọng

### 6.1. `ldd` - Xem thư viện program cần
```bash
ldd [program]
```

**Ví dụ**:
```bash
ldd /bin/bash
# linux-vdso.so.1
# libtinfo.so.6 => /lib64/libtinfo.so.6
# libc.so.6 => /lib64/libc.so.6
# /lib64/ld-linux-x86-64.so.2  ← Dynamic linker
```

**Đọc kết quả**:
```
libc.so.6 => /lib64/libc.so.6 (0x00007f...)
   ↑            ↑                  ↑
  Tên       Đường dẫn        Địa chỉ RAM
```

**Lỗi thiếu lib**:
```bash
ldd /opt/myapp
# libfoo.so.1 => not found  ← THIẾU!
```

---

### 6.2. `ldconfig` - Cập nhật cache
```bash
sudo ldconfig
```

**Chức năng**:
1. Quét thư mục trong `/etc/ld.so.conf`
2. Tìm tất cả `.so` files
3. Tạo symlink (nếu cần)
4. Tạo `/etc/ld.so.cache`

**Khi nào chạy**:
- ✅ Sau khi cài lib mới
- ✅ Sau khi thêm đường dẫn vào `/etc/ld.so.conf.d/`
- ✅ Khi program báo "cannot open shared object file"

**Tùy chọn hữu ích**:

| Lệnh | Mô tả |
|------|-------|
| `ldconfig` | Cập nhật cache |
| `ldconfig -p` | Xem tất cả lib trong cache |
| `ldconfig -p | grep [lib]` | Tìm lib cụ thể |
| `ldconfig -v` | Verbose mode |

**Ví dụ**:
```bash
# Cài lib mới
sudo cp libfoo.so /usr/local/lib/
sudo ldconfig

# Kiểm tra
ldconfig -p | grep libfoo
# libfoo.so.1 => /usr/local/lib/libfoo.so.1
```

---

### 6.3. `LD_DEBUG` - Debug dynamic linker
```bash
LD_DEBUG=[option] ./program
```

**Options**:

| Option | Hiển thị |
|--------|----------|
| `libs` | Lib nào được load |
| `files` | File nào được mở |
| `bindings` | Function binding |
| `all` | Tất cả |

**Ví dụ**:
```bash
# Xem lib được load
LD_DEBUG=libs ./myapp 2>&1 | grep libfoo
# search cache=/etc/ld.so.cache
# trying file=/lib64/libfoo.so.1
```

---

## 7. GCC - Compile với Shared Library

### 7.1. Compile chương trình dùng lib

#### Dùng lib hệ thống (đã có sẵn)
```bash
# Compile và link với libc (tự động)
gcc app.c -o app

# Link với lib cụ thể (-l)
gcc app.c -o app -lm        # Link với libm.so (math)
gcc app.c -o app -lpthread  # Link với libpthread.so

# Chỉ định thư mục lib (-L)
gcc app.c -o app -L/opt/myapp/lib -lmylib
#                   ↑                 ↑
#              Thư mục lib      Tên lib (libmylib.so)
```

**Lưu ý**: `-lmylib` → Tìm `libmylib.so` hoặc `libmylib.a`

---

#### Dùng lib ở thư mục không chuẩn
```bash
# Cách 1: Dùng -L và -l
gcc app.c -o app -L/opt/lib -lfoo

# Cách 2: Chỉ định đường dẫn đầy đủ
gcc app.c -o app /opt/lib/libfoo.so

# Cách 3: Dùng -Wl,-rpath (nhúng đường dẫn vào executable)
gcc app.c -o app -L/opt/lib -lfoo -Wl,-rpath,/opt/lib
#                                      ↑
#                    Executable sẽ nhớ đường dẫn này
```

**So sánh**:

| Cách | Cần ldconfig? | Cần LD_LIBRARY_PATH? | Runtime |
|------|---------------|----------------------|---------|
| `-L -l` | ✅ Có | ✅ Có (hoặc ldconfig) | Tìm theo thứ tự ưu tiên |
| `-Wl,-rpath` | ❌ Không | ❌ Không | Tìm tại đường dẫn nhúng |

---

#### Compile static (nhúng lib vào executable)
```bash
# Link static với lib cụ thể
gcc app.c -o app -static -lm

# Link static với tất cả lib
gcc app.c -o app -static

# Kết quả: executable lớn, độc lập
ls -lh app
# -rwxr-xr-x app  (10MB thay vì 20KB)
```

---

### 7.2. Tạo Shared Library

#### Bước 1: Compile thành object file (.o) với -fPIC
```bash
gcc -c -fPIC mylib.c -o mylib.o
#       ↑
#   Position Independent Code (BẮT BUỘC cho shared lib)
```

**Tại sao cần `-fPIC`?**
- Shared lib được load vào **địa chỉ RAM bất kỳ**
- Code phải "độc lập vị trí" để chạy ở bất kỳ địa chỉ nào

---

#### Bước 2: Tạo shared library (.so)
```bash
gcc -shared mylib.o -o libmylib.so
#    ↑
#   Tạo shared library
```

**Hoặc gộp 2 bước**:
```bash
gcc -shared -fPIC mylib.c -o libmylib.so
```

---

#### Bước 3: Tạo symlink (nếu cần)
```bash
# libmylib.so.1.0.0 = File thật (phiên bản cụ thể)
ln -s libmylib.so.1.0.0 libmylib.so.1  # Symlink major version
ln -s libmylib.so.1 libmylib.so        # Symlink generic
```

**Kết quả**:
```
libmylib.so -> libmylib.so.1 -> libmylib.so.1.0.0
   (dev)         (runtime)         (file thật)
```

---

#### Bước 4: Cài đặt vào hệ thống
```bash
# Copy vào thư mục hệ thống
sudo cp libmylib.so* /usr/local/lib/

# Cập nhật cache
sudo ldconfig

# Kiểm tra
ldconfig -p | grep libmylib
```

---

### 7.3. GCC Cheatsheet

| Mục đích | Lệnh | Giải thích |
|----------|------|------------|
| **Compile thông thường** | `gcc app.c -o app` | Link dynamic với libc |
| **Link với lib cụ thể** | `gcc app.c -lm -lpthread -o app` | Link libm.so, libpthread.so |
| **Chỉ định thư mục lib** | `gcc app.c -L/opt/lib -lfoo -o app` | Tìm libfoo.so trong /opt/lib |
| **Nhúng đường dẫn lib** | `gcc app.c -L/opt/lib -lfoo -Wl,-rpath,/opt/lib -o app` | Executable nhớ đường dẫn |
| **Compile static** | `gcc -static app.c -o app` | Nhúng tất cả lib vào executable |
| **Xem lib program cần** | `ldd ./app` | List dynamic dependencies |
| | | |
| **Tạo shared lib (1 bước)** | `gcc -shared -fPIC mylib.c -o libmylib.so` | Tạo libmylib.so từ mylib.c |
| **Tạo shared lib (2 bước)** | `gcc -c -fPIC mylib.c -o mylib.o`<br>`gcc -shared mylib.o -o libmylib.so` | Compile object → Tạo .so |
| **Tạo static lib** | `gcc -c mylib.c -o mylib.o`<br>`ar rcs libmylib.a mylib.o` | Tạo libmylib.a (archive) |

---

### 7.4. Các flag GCC quan trọng

| Flag | Ý nghĩa | Khi nào dùng |
|------|---------|--------------|
| `-l[name]` | Link với lib`[name]`.so | `-lm` → libm.so, `-lpthread` → libpthread.so |
| `-L[dir]` | Thêm thư mục tìm lib | `-L/opt/lib` → Tìm trong /opt/lib |
| `-I[dir]` | Thêm thư mục tìm header (.h) | `-I/opt/include` |
| `-fPIC` | Position Independent Code | **BẮT BUỘC** khi tạo shared lib |
| `-shared` | Tạo shared library | Tạo .so file |
| `-static` | Link static | Nhúng lib vào executable |
| `-Wl,-rpath,[dir]` | Nhúng đường dẫn lib vào executable | Không cần ldconfig/LD_LIBRARY_PATH |
| `-Wl,-soname,[name]` | Đặt soname cho shared lib | `-Wl,-soname,libfoo.so.1` |

---

### 7.5. Ví dụ thực tế

#### Ví dụ 1: Tạo và dùng shared library
```bash
# File: mylib.c
#include <stdio.h>
void hello() { printf("Hello from mylib!\n"); }

# File: app.c
void hello();
int main() { hello(); return 0; }

# Bước 1: Tạo shared library
gcc -shared -fPIC mylib.c -o libmylib.so

# Bước 2: Compile app (link với mylib)
gcc app.c -L. -lmylib -o app
#           ↑    ↑
#       Thư mục  Lib name
#       hiện tại

# Bước 3: Chạy (lỗi - thiếu lib trong hệ thống)
./app
# Error: libmylib.so: cannot open shared object file

# Bước 4: Fix tạm thời
export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH
./app
# Hello from mylib!

# Bước 5: Fix vĩnh viễn (cách 1 - ldconfig)
sudo cp libmylib.so /usr/local/lib/
sudo ldconfig
unset LD_LIBRARY_PATH
./app  # OK!

# Hoặc (cách 2 - rpath)
gcc app.c -L. -lmylib -Wl,-rpath,$(pwd) -o app
./app  # OK! (không cần ldconfig)
```

---

#### Ví dụ 2: Compile với nhiều lib
```bash
# app.c cần: libm.so, libpthread.so, libssl.so
gcc app.c -o app -lm -lpthread -lssl -lcrypto

# Kiểm tra
ldd ./app
# libm.so.6 => /lib64/libm.so.6
# libpthread.so.0 => /lib64/libpthread.so.0
# libssl.so.1.1 => /lib64/libssl.so.1.1
# libcrypto.so.1.1 => /lib64/libcrypto.so.1.1
```

---

#### Ví dụ 3: Tạo lib với version
```bash
# Tạo shared lib với version
gcc -shared -fPIC -Wl,-soname,libfoo.so.1 foo.c -o libfoo.so.1.0.0

# Tạo symlink
ln -s libfoo.so.1.0.0 libfoo.so.1
ln -s libfoo.so.1 libfoo.so

# Cài đặt
sudo cp libfoo.so* /usr/local/lib/
sudo ldconfig

# Kết quả
ls -l /usr/local/lib/libfoo*
# libfoo.so -> libfoo.so.1
# libfoo.so.1 -> libfoo.so.1.0.0
# libfoo.so.1.0.0
```

---

## 8. Workflow thực tế

### Workflow 1: Xử lý lỗi "cannot open shared object file"
```bash
# Bước 1: Lỗi
./myapp
# Error: libfoo.so.1: cannot open shared object file

# Bước 2: Xác định thiếu gì
ldd ./myapp | grep "not found"
# libfoo.so.1 => not found

# Bước 3: Tìm thư viện
sudo find / -name "libfoo.so*" 2>/dev/null
# /opt/vendor/lib/libfoo.so.1

# Bước 4: Test tạm thời
export LD_LIBRARY_PATH=/opt/vendor/lib:$LD_LIBRARY_PATH
./myapp  # Chạy được!

# Bước 5: Fix vĩnh viễn
echo "/opt/vendor/lib" | sudo tee /etc/ld.so.conf.d/vendor.conf
sudo ldconfig
unset LD_LIBRARY_PATH

# Bước 6: Kiểm tra
ldd ./myapp | grep libfoo
# libfoo.so.1 => /opt/vendor/lib/libfoo.so.1 ✅
```

---

### Workflow 2: Compile từ source có nhiều lib
```bash
# Download
wget https://example.com/app-1.0.tar.gz
tar -xzf app-1.0.tar.gz
cd app-1.0

# Configure (tự động tìm lib)
./configure --prefix=/usr/local

# Nếu thiếu lib
# Error: libssl not found
sudo yum install openssl-devel  # RHEL/CentOS
# hoặc
sudo apt install libssl-dev     # Ubuntu/Debian

# Compile
make

# Install
sudo make install

# Cập nhật ldconfig
sudo ldconfig

# Kiểm tra
ldd /usr/local/bin/app
```

---

## 9. Troubleshooting

### Lỗi: "version 'GLIBC_X.XX' not found"
```bash
./myapp
# Error: version `GLIBC_2.28' not found
```

**Nguyên nhân**: Program compile trên hệ thống có glibc mới hơn.

**Kiểm tra**:
```bash
ldd --version
# ldd (GNU libc) 2.17  ← Hệ thống chỉ có 2.17

strings ./myapp | grep GLIBC_
# GLIBC_2.28  ← Cần 2.28
```

**Giải pháp**:
1. Compile lại trên hệ thống hiện tại
2. Upgrade OS
3. Dùng container

---

### Lỗi: LD_LIBRARY_PATH không hoạt động
```bash
export LD_LIBRARY_PATH=/opt/lib
./myapp
# Vẫn báo thiếu lib
```

**Nguyên nhân**:
1. Thư viện không tồn tại
2. Quyền file sai
3. Program là setuid (bỏ qua LD_LIBRARY_PATH vì bảo mật)

**Debug**:
```bash
# Kiểm tra file tồn tại
ls -l /opt/lib/libfoo.so*

# Kiểm tra quyền
ls -l /opt/lib/libfoo.so.1
# -rw-r--r-- ← SAI! Cần execute
sudo chmod 755 /opt/lib/libfoo.so.1

# Kiểm tra setuid
ls -l ./myapp
# -rwsr-xr-x ← Có 's' = setuid, bỏ qua LD_LIBRARY_PATH
```

---

### Lỗi: gcc không tìm thấy lib
```bash
gcc app.c -lmylib -o app
# /usr/bin/ld: cannot find -lmylib
```

**Nguyên nhân**: Thư mục lib không có trong search path.

**Giải pháp**:
```bash
# Thêm -L
gcc app.c -L/opt/mylib -lmylib -o app

# Hoặc chỉ định đường dẫn đầy đủ
gcc app.c /opt/mylib/libmylib.so -o app
```

---

## 10. Best Practices

### ✅ Nên làm

**1. Production: Dùng ldconfig**
```bash
echo "/opt/myapp/lib" | sudo tee /etc/ld.so.conf.d/myapp.conf
sudo ldconfig
```

**2. Tạo lib với version**
```bash
gcc -shared -fPIC -Wl,-soname,libfoo.so.1 foo.c -o libfoo.so.1.0.0
ln -s libfoo.so.1.0.0 libfoo.so.1
ln -s libfoo.so.1 libfoo.so
```

**3. Kiểm tra sau khi cài**
```bash
ldconfig -p | grep mylib
ldd /opt/myapp/bin/app
```

---

### ❌ Không nên làm

**1. KHÔNG dùng LD_LIBRARY_PATH trong production**
```bash
# SAI
export LD_LIBRARY_PATH=/opt/lib

# ĐÚNG
echo "/opt/lib" | sudo tee /etc/ld.so.conf.d/myapp.conf
sudo ldconfig
```

**2. KHÔNG quên -fPIC khi tạo shared lib**
```bash
# SAI
gcc -shared mylib.c -o libmylib.so

# ĐÚNG
gcc -shared -fPIC mylib.c -o libmylib.so
```

**3. KHÔNG quên ldconfig**
```bash
# SAI
sudo cp libfoo.so /usr/local/lib/

# ĐÚNG
sudo cp libfoo.so /usr/local/lib/
sudo ldconfig  # ← BẮT BUỘC
```

---

## 11. Tóm tắt nhanh

### Lệnh cần nhớ
```bash
# Xem lib program cần
ldd /path/to/program

# Cập nhật cache
sudo ldconfig

# Tìm lib trong cache
ldconfig -p | grep [lib]

# Debug linker
LD_DEBUG=libs ./program 2>&1 | less
```

### File quan trọng
```bash
/etc/ld.so.conf.d/*.conf  # Config đường dẫn lib
/etc/ld.so.cache          # Cache (binary)
/lib64/ld-linux-x86-64.so.2  # Dynamic linker
```

### GCC cơ bản
```bash
# Compile với lib
gcc app.c -lm -lpthread -o app

# Compile với lib ở thư mục khác
gcc app.c -L/opt/lib -lfoo -o app

# Tạo shared lib
gcc -shared -fPIC mylib.c -o libmylib.so

# Nhúng đường dẫn lib
gcc app.c -L/opt/lib -lfoo -Wl,-rpath,/opt/lib -o app
```

### Quy trình fix lỗi thiếu lib
```bash
1. ldd ./program | grep "not found"
2. find / -name "libfoo.so*"
3. export LD_LIBRARY_PATH=/path (test)
4. echo "/path" | sudo tee /etc/ld.so.conf.d/app.conf
5. sudo ldconfig
6. ldd ./program (kiểm tra lại)
```

### Load-time vs Run-time

| | Load-time | Run-time |
|---|-----------|----------|
| **Load khi** | Khởi động program | Gặp dlopen() |
| **Lỗi phát hiện** | Ngay khi chạy | Khi chạy đến dòng đó |
| **ldd thấy** | ✅ Có | ❌ Không |
| **Phổ biến** | 99% program | Plugin systems |
