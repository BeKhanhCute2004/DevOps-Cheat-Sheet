[Hardware Information Commands](#hardware-information-commands)
<br>[Linux Boot Process](#linux-boot-process)

# Hardware Information Commands
| Command | Description | How to Read/Use |
|-------|-------|-------|
| lscpu | See CPU information |  |
| lsblk | See information about block devices | **RM (Removable)**: nếu là 1 thì đây là thiết bị có thể tháo rời (như USB), 0 là ổ cứng cố định.<br>**RO (Read-Only)**: nếu là 1, thiết bị chỉ có thể đọc (như CD-ROM).<br>**sdX (sda, sdb...)**: ổ cứng chuẩn SATA hoặc SSD đời cũ.<br>**nvmeXnY (nvme0n1...)**: ổ cứng SSD chuẩn NVMe tốc độ cao.<br>**sr0**: thường là ổ đĩa quang (CD/DVD).<br>**vda, vdb**: ổ đĩa ảo trong môi trường máy ảo (KVM/QEMU). |
| lspci -tv | Show PCI devices (graphics card, network card, etc.) in a tree-like diagram | một dòng điển hình: -[0000:00]-+-11.0-[02]----01.0 Device Name<br>**-[0000:00]- (Gốc)**: điểm xuất phát từ CPU/Chipset (Bus chính).<br>**+-11.0 (Cầu nối)**: thiết bị trung gian (Bridge) nằm trên Bus chính. nó giống như một cái "ổ cắm chuyền".<br>**-[02]- (Đường đi mới)**: sau khi đi qua cầu nối, bạn đang ở một con đường mới (Bus 02).<br>**----01.0 (Đích đến)**: thiết bị cuối cùng (Endpoint) cắm vào con đường đó.<br>**Device Name**: tên của thiết bị (Card mạng, Card màn hình, v.v.). |
| lsusb -tv | Display USB devices in a tree-like diagram | Đường đi dữ liệu: vậy sẽ là thiết bị -> port -> USB bus -> root hub (port 1) -> USB controller -> PCI bus -> CPU<br>**Lưu ý 1**: một USB bus có thể chưa nhiều port, tất cả thiết bị chung một bus sẽ chia sẻ chung băng thông và nguồn điện của bus đó.<br>**Class=...**: Mass Storage (ổ cứng), Human Interface Device (chuột/phím), Hub (bộ chia), ...<br>**5000M**: USB 3.0 (nhanh)<br>**480M**: USB 2.0 (trung bình)<br>**Lưu ý 2**: để xem số port của một bus thì nhìn vào phần Driver port 1, /[n]p là có [n] port<br>**Lưu ý 3**: một thiết bị có nhiều chức năng khi cắm vào port sẽ hiển thị nhiều nhánh và có If [ID] khác nhau |
| lshw -C network<br>lshw -C storage/lshw -C disk<br>lshw -C processor<br>lshw -C memory | List hardware configuration information | Tập trung xem các phần:<br>**description**: mô tả ngắn gọn thiết bị là gì.<br>**product**: tên model chính xác của linh kiện.<br>**vendor**: nhà sản xuất (intel, amd, realtek...).<br>**logical name**: tên định danh trong hệ thống (ví dụ: ổ cứng là /dev/sda, card mạng là eth0 hoặc enp0s3).<br>**configuration**:  chứa tên **driver** và các thông số vận hành (speed, latency, width).<br>**Lưu ý**: storage là list storage controller còn disk là list ổ đĩa |
| cat /proc/cpuinfo | Show detailed CPU information | Thông tin từng core trên CPU |
| cat /proc/meminfo | View detailed system memory information | Thông tin RAM |
| cat /proc/mounts | See mounted file systems | **Device**:tên thiết bị hoặc hệ thống tệp ảo được mount (ví dụ: /dev/sda1, proc, sysfs).<br>**Mount Point**: thư mục mà thiết bị đó được gắn vào (ví dụ: /, /home, /sys).<br>**FSType**: loại hệ thống tệp (ví dụ: ext4, xfs, tmpfs, sysfs).<br>**Options**: các tùy chọn khi mount (quyền đọc/ghi, bảo mật...).<br>**Dump**: dùng cho lệnh dump để sao lưu (thường là 0 trong /proc/mounts).<br>**Pass**: thứ tự kiểm tra ổ đĩa bằng fsck khi khởi động (thường là 0 ở đây).<br><br>**Lưu ý**: thiết bị thật bắt đầu bằng /dev, pseudo filesystems (proc, sysfs, ...) và tmpfs chạy trên RAM không tốn dung lượng ổ cứng. Đọc trên /proc/mount **tin cậy hơn** /etc/mtab |
| free -h | Display free and used memory |  |
| sudo dmidecode | Show hardware information from the BIOS | Dữ liệu 2 |
| hdparm -i /dev/[device_name] | Display disk data information | Dữ liệu 2 |
| hdparm -tT /dev/[device_name] | Conduct a read speed test on the device/disk | **t**: buffered, kiểm tra xem tốc độ đọc dữ liệu thô của ổ cứng là bao nhiêu.<br>**T**: cached, kiểm tra xem hệ thống quản lý bộ nhớ đệm có nhanh không, giống hệt nhau dù tham số ổ đĩa là khác nhau. |
| badblocks -s /dev/[device_name] | Test for unreadable blocks on the device/disk |  |
| fsck /dev/[device_name] | 	Run a disk check on an unmounted disk or partition |  |

# Linux Boot Process
## Giai đoạn 1: Đánh thức phần cứng (Firmware)
**BIOS/UEFI thức dậy**: Đây là phần mềm nằm trong chip trên Mainboard.
<br><br>**POST (Power-On Self-Test)**: Kiểm tra các linh kiện cơ bản (RAM, CPU, Keyboard).
<br><br>**Tìm kiếm Mục lục (Partition Table)**:
* BIOS: Tìm bảng MBR (512 byte đầu ổ cứng).
* UEFI: Tìm phân vùng EFI (ESP) định dạng FAT32.
**Chuyển giao**: BIOS/UEFI nạp đoạn mã mồi của Bootloader vào RAM và ra lệnh cho CPU thực thi.  

## Giai đoạn 2: Trình khởi động (Bootloader - GRUB)
Lúc này mã lệnh của GRUB đã nằm trên RAM:
<br><br>**GRUB Stage 1**: Đoạn mã cực nhỏ từ MBR/EFI sẽ tìm và nạp phần còn lại của GRUB từ thư mục /boot/grub trên ổ cứng.
<br><br>**Menu hiển thị**: Bạn thấy danh sách các Kernel Linux hoặc Windows.
<br><br>**Lựa chọn**:
* Nếu chọn Windows: GRUB chuyển giao (Chainload) cho bootmgr.efi.
* Nếu chọn Linux: GRUB nạp file Kernel và file initrd vào RAM.
**Kết thúc**: GRUB bàn giao toàn bộ quyền điều khiển RAM cho Kernel.

## Giai đoạn 3: Nhân hệ điều hành & Đội tiền trạm (Kernel & initrd)
Đây là giai đoạn "nhảy" từ RAM vào ổ cứng thật:
<br><br>**Kernel thực thi**: Tự giải nén và bắt đầu quản lý CPU, RAM.
<br><br>**initrd (Hệ điều hành mini)**: Vì Kernel chưa có driver ổ cứng, nó sẽ dùng "balo" initrd (đã nạp sẵn trên RAM) để lấy các driver thiết yếu.
<br><br>**Nhận diện phần cứng**: initrd giúp Kernel "thấy" được ổ cứng thật, card mạng, các phân vùng LVM/RAID.
<br><br>**Switch Root**: Kernel gắn (mount) phân vùng gốc / từ ổ cứng thật vào hệ thống và xóa initrd khỏi RAM để tiết kiệm chỗ.

## Giai đoạn 4: Tiến trình tổ tiên (Init Process)
Sau khi đã ở trong ổ cứng thật, Kernel gọi "người quản gia" trưởng:
<br><br>**Chạy /sbin/init**: Đây là tiến trình đầu tiên (PID 1).
<br><br>**Đọc cấu hình**: init đọc file /etc/inittab (đối với SysVinit) hoặc các file Unit (đối với Systemd hiện đại).
<br><br>**Xác định Runlevel**: Hệ thống xác định sẽ chạy ở chế độ nào (thường là Runlevel 3 - Server hoặc Runlevel 5 - Desktop).

## Giai đoạn 5: Bật dịch vụ & Đăng nhập (Services & Login)
"Quản gia" init bắt đầu gọi các nhân viên khác dậy theo thứ tự:
<br><br>**Kiểm tra ổ đĩa (fsck)**: Đảm bảo dữ liệu không bị lỗi.
<br><br>**Bật Mạng**: Kích hoạt các giao tiếp mạng (IP, Routing).
<br><br>**Bật Dịch vụ**: Chạy các script khởi động SSH, Database, Web Server...
<br><br>**Mở cổng TTY**: Kích hoạt các màn hình dòng lệnh (Teletype).
<br><br>**Login Prompt**: Hiện dòng chữ yêu cầu Username/Password.
