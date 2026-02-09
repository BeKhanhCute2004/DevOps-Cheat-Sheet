# Hardware Information Commands
| Command | Description | How to Read/Use |
|-------|-------|-------|
| lscpu | See CPU information |  |
| lsblk | See information about block devices | **RM (Removable)**: nếu là 1 thì đây là thiết bị có thể tháo rời (như USB), 0 là ổ cứng cố định.<br>**RO (Read-Only)**: nếu là 1, thiết bị chỉ có thể đọc (như CD-ROM).<br>**sdX (sda, sdb...)**: ổ cứng chuẩn SATA hoặc SSD đời cũ.<br>**nvmeXnY (nvme0n1...)**: ổ cứng SSD chuẩn NVMe tốc độ cao.<br>**sr0**: thường là ổ đĩa quang (CD/DVD).<br>**vda, vdb**: ổ đĩa ảo trong môi trường máy ảo (KVM/QEMU). |
| lspci -tv | Show PCI devices (graphics card, network card, etc.) in a tree-like diagram | một dòng điển hình: -[0000:00]-+-11.0-[02]----01.0 Device Name<br>**-[0000:00]- (Gốc)**: điểm xuất phát từ CPU/Chipset (Bus chính).<br>**+-11.0 (Cầu nối)**: thiết bị trung gian (Bridge) nằm trên Bus chính. nó giống như một cái "ổ cắm chuyền".<br>**-[02]- (Đường đi mới)**: sau khi đi qua cầu nối, bạn đang ở một con đường mới (Bus 02).<br>**----01.0 (Đích đến)**: thiết bị cuối cùng (Endpoint) cắm vào con đường đó.<br>**Device Name**: tên của thiết bị (Card mạng, Card màn hình, v.v.). |
| lsusb -tv | Display USB devices in a tree-like diagram | Đường đi dữ liệu: vậy sẽ là thiết bị -> port -> USB bus -> root hub (port 1) -> USB controller -> PCI bus -> CPU**Lưu ý 1**: một USB bus có thể chưa nhiều port, tất cả thiết bị chung một bus sẽ chia sẻ chung băng thông và nguồn điện của bus đó.<br>**Class=...**: Mass Storage (ổ cứng), Human Interface Device (chuột/phím), Hub (bộ chia), ...<br>**5000M**: USB 3.0 (nhanh)<br>**480M**: USB 2.0 (trung bình)<br>**Lưu ý 2**: để xem số port của một bus thì nhìn vào phần Driver port 1, /[n]p là có [n] port<br>**Lưu ý 3**: một thiết bị có nhiều chức năng khi cắm vào port sẽ hiển thị nhiều nhánh và có If [ID] khác nhau |
| lshw | List hardware configuration information | Dữ liệu 2 |
| cat /proc/cpuinfo | Show detailed CPU information | Dữ liệu 2 |
| cat /proc/meminfo | View detailed system memory information | Dữ liệu 2 |
| cat /proc/mounts | See mounted file systems | Dữ liệu 2 |
| free -h | Display free and used memory | Dữ liệu 2 |
| sudo dmidecode | Show hardware information from the BIOS | Dữ liệu 2 |
| hdparm -i /dev/[device_name] | Display disk data information | Dữ liệu 2 |
| hdparm -tT /dev/[device_name] | Conduct a read speed test on the device/disk | Dữ liệu 2 |
| badblocks -s /dev/[device_name] | Test for unreadable blocks on the device/disk | Dữ liệu 2 |
| fsck /dev/[device_name] | 	Run a disk check on an unmounted disk or partition | Dữ liệu 2 |

