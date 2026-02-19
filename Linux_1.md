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
| `lvextend -L +[size] /dev/[vg]/[lv]` | Mở rộng LV | Ví dụ: `lvextend -L +10G /dev/vg_data/lv_home`<br>Thêm 10GB vào LV.<br>**Lưu ý**: sau đó phải resize filesystem bằng `resize2fs` (ext4) hoặc `xfs_growfs` (xfs). |
| `lvreduce -L -[size] /dev/[vg]/[lv]` | Thu nhỏ LV | Ví dụ: `lvreduce -L -5G /dev/vg_data/lv_home`<br>**NGUY HIỂM**: phải unmount và resize filesystem trước, nếu không sẽ mất dữ liệu. |
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
| **Primary** (Chính) | Phân vùng độc lập, tối đa 4 phân vùng (giới hạn MBR) | 1-4 |
| **Extended** (Mở rộng) | Container chứa các phân vùng logic, chỉ có 1 trên mỗi ổ | Tính vào 1 trong 4 primary |
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
