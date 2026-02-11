# Mục lục

[Hardware Information Commands](#hardware-information-commands)  
[Linux Boot Process](#linux-boot-process)

# Hardware Information Commands

| Command | Description | How to Read/Use |
|---------|-------------|-----------------|
| lscpu | See CPU information | |
| lsblk | See information about block devices | **RM (Removable)**: nếu là 1 thì đây là thiết bị có thể tháo rời (như USB), 0 là ổ cứng cố định.<br>**RO (Read-Only)**: nếu là 1, thiết bị chỉ có thể đọc (như CD-ROM).<br>**sdX (sda, sdb...)**: ổ cứng chuẩn SATA hoặc SSD đời cũ.<br>**nvmeXnY (nvme0n1...)**: ổ cứng SSD chuẩn NVMe tốc độ cao.<br>**sr0**: thường là ổ đĩa quang (CD/DVD).<br>**vda, vdb**: ổ đĩa ảo trong môi trường máy ảo (KVM/QEMU). |
| lspci -tv | Show PCI devices (graphics card, network card, etc.) in a tree-like diagram | Một dòng điển hình: `-[0000:00]-+-11.0-[02]----01.0 Device Name`<br>**-[0000:00]- (Gốc)**: điểm xuất phát từ CPU/Chipset (Bus chính).<br>**+-11.0 (Cầu nối)**: thiết bị trung gian (Bridge) nằm trên Bus chính. Nó giống như một cái "ổ cắm chuyền".<br>**-[02]- (Đường đi mới)**: sau khi đi qua cầu nối, bạn đang ở một con đường mới (Bus 02).<br>**----01.0 (Đích đến)**: thiết bị cuối cùng (Endpoint) cắm vào con đường đó.<br>**Device Name**: tên của thiết bị (Card mạng, Card màn hình, v.v.). |
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

# Sysvinit, Upstart và Systemd

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
| **Hỗ trợ container** | Không | Không | Có - `systemd-nspawn` |
| **Tương thích ngược** | N/A (đây là hệ thống gốc) | Tương thích với SysVinit scripts | Tương thích với cả SysVinit và Upstart<br>Chạy được legacy scripts |
| **Độ phức tạp** | Đơn giản - dễ học | Trung bình | Phức tạp - nhiều tính năng, khó học ban đầu |
| **Kích thước** | Nhỏ gọn (~100KB) | Trung bình (~500KB) | Lớn (~1-2MB core + các module) |
| **Khả năng mở rộng** | Khó - cần viết shell script phức tạp | Khá tốt - dựa trên events | Rất tốt - nhiều loại units, plugins |
| **Debugging** | Khó - phải đọc log và script | Trung bình - `initctl log-priority` | Dễ - `journalctl`, `systemd-analyze` |
| **Phân tích thời gian boot** | Không tích hợp | Không tích hợp | `systemd-analyze blame`<br>`systemd-analyze critical-chain` |
| **User session management** | Không | Hỗ trợ hạn chế | `systemd --user` - quản lý session của từng user |
| **Network management** | Dùng scripts riêng | Dùng NetworkManager riêng | `systemd-networkd` tích hợp |
| **Hostname management** | File `/etc/hostname` + scripts | File `/etc/hostname` | `hostnamectl` |
| **Time/Date management** | `date`, `hwclock` commands | `date`, `hwclock` commands | `timedatectl` |
| **Locale management** | File `/etc/locale.conf` | File `/etc/locale.conf` | `localectl` |
| **Ví dụ script/unit** | `#!/bin/bash`<br>`case "$1" in`<br>`  start)`<br>`    /usr/bin/daemon &`<br>`    ;;`<br>`esac` | `description "My Service"`<br>`start on runlevel [2345]`<br>`stop on runlevel [!2345]`<br>`respawn`<br>`exec /usr/bin/daemon` | `[Unit]`<br>`Description=My Service`<br>`After=network.target`<br>`[Service]`<br>`ExecStart=/usr/bin/daemon`<br>`Restart=on-failure`<br>`[Install]`<br>`WantedBy=multi-user.target` |
| **Distro sử dụng (hiện tại)** | Hầu như không còn<br>Slackware (vẫn dùng) | Ubuntu <15.04 (đã chuyển sang Systemd)<br>Chrome OS (vẫn dùng) | Hầu hết distro hiện đại:<br>RHEL/CentOS 7+<br>Debian 8+<br>Ubuntu 15.04+<br>Fedora 15+<br>Arch Linux<br>openSUSE |
| **Ưu điểm chính** | • Đơn giản, dễ hiểu<br>• Ổn định, đã test qua thời gian<br>• Ít phụ thuộc<br>• Dễ debug với shell script | • Nhanh hơn SysVinit<br>• Event-driven linh hoạt<br>• Tương thích ngược tốt<br>• Khởi động lại service tự động | • Rất nhanh (khởi động song song)<br>• Quản lý toàn diện hệ thống<br>• Logging mạnh mẽ<br>• Quản lý phụ thuộc tự động<br>• Nhiều tính năng tích hợp |
| **Nhược điểm chính** | • Quá chậm<br>• Không tự động hóa<br>• Khó quản lý phụ thuộc<br>• Thiếu tính năng hiện đại | • Ít được phát triển<br>• Cộng đồng nhỏ<br>• Tài liệu hạn chế<br>• Bị thay thế bởi Systemd | • Phức tạp, khó học<br>• "Làm quá nhiều thứ" (vi phạm Unix philosophy)<br>• Binary log khó đọc trực tiếp<br>• Gây tranh cãi trong cộng đồng |
| **Trạng thái hiện tại** | Legacy - không còn phát triển | Deprecated - Canonical đã bỏ | Tiêu chuẩn hiện tại - đang phát triển mạnh |
