# Hardware Information Commands
| Command | Description | How to Read/Use |
|-------|-------|-------|
| lscpu | See CPU information |  |
| lsblk | See information about block devices | **RM (Removable)**: nếu là 1 thì đây là thiết bị có thể tháo rời (như USB), 0 là ổ cứng cố định.<br>**RO (Read-Only)**: nếu là 1, thiết bị chỉ có thể đọc (như CD-ROM). |
| lspci -tv | Show PCI devices (graphics card, network card, etc.) in a tree-like diagram | Dữ liệu 2 |
| lsusb -tv | Display USB devices in a tree-like diagram | Dữ liệu 2 |
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

