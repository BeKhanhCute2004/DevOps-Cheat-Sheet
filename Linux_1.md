# Xác định và cấu hình cài đặt phần cứng(Filesystems và Device Files)
## 1. Triết lý "Everything is a File" (Mọi thứ đều là file)
Trong Linux, bạn không tương tác trực tiếp với các dòng điện hay bảng mạch. Kernel (nhân hệ điều hành) đại diện mỗi thiết bị bằng một file.
* Lợi ích: Bạn có thể đọc dữ liệu từ chuột hoặc ghi dữ liệu vào ổ cứng bằng chính các công cụ đọc/ghi văn bản thông thường.
* Phân loại: Các file này thường nằm trong /dev (thiết bị thật) hoặc các "giả hệ thống file" (pseudofilesystems) như /proc và /sys để xem trạng thái.

## 2. Procfs (Process Filesystem) - /proc
Đây là một pseudofilesystem (hệ thống file ảo). Nó không chiếm không gian trên ổ cứng mà tồn tại trong RAM. Nó giống như một "cửa sổ" để bạn nhìn vào bên trong bộ não của Kernel.
Các thành phần chính:
* Thư mục số (PID): Ví dụ /proc/1. Mỗi con số là ID của một tiến trình đang chạy. Vào đây bạn sẽ thấy tiến trình đó đang dùng bao nhiêu RAM, chạy lệnh gì.
  * /proc/[PID]/cmdline: Chứa lệnh thực tế đã dùng để khởi chạy tiến trình này.
  * /proc/[PID]/status: Thông tin chi tiết về trạng thái (đang chạy hay ngủ, tiêu tốn bao nhiêu RAM).
  * /proc/[PID]/environ: Các biến môi trường mà tiến trình đó đang sử dụng.
  * /proc/[PID]/exe: Một liên kết (symlink) trỏ thẳng đến file thực thi gốc trên ổ cứng.
  * /proc/[PID](/task)/[TID]: Từng luồng bên trong một tiến trình.
* /proc/interrupts: Xem các thiết bị đang "ngắt" CPU như thế nào.
  * Cột 1: Số hiệu ngắt (IRQ).
  * Cột giữa: Số lần ngắt xảy ra trên từng nhân CPU (CPU0, CPU1...).
  * Cột cuối: Tên thiết bị gây ra ngắt (ví dụ: bàn phím, card mạng, timer).
  * Ứng dụng: Nếu một nhân CPU hoạt động 100% trong khi các nhân khác rảnh, bạn vào đây xem có thiết bị nào đang "spam" ngắt quá nhiều không.
* /proc/dma: Xem thiết bị nào đang dùng quyền "truy cập bộ nhớ đặc cách" mà không làm phiền CPU.
  * Hiển thị các kênh DMA đang được sử dụng. DMA cho phép các thiết bị (như ổ đĩa) chuyển dữ liệu thẳng vào RAM mà không cần chạy qua CPU.
  * Nếu danh sách trống, nghĩa là các driver hiện tại không đăng ký kênh DMA tĩnh hoặc hệ thống hiện đại quản lý theo cách khác.
* /proc/mounts: Liệt kê tất cả các hệ thống file hiện đang được "gắn" (mount) vào cây thư mục Linux.
  * Cấu trúc: [Thiết bị] [Điểm gắn] [Loại Filesystem] [Tùy chọn quyền] [Dump] [Pass]
  * Nó cho bạn biết: Tên thiết bị (ví dụ /dev/sda1), điểm gắn (ví dụ /), loại filesystem (ext4, xfs, tmpfs) và các tùy chọn quyền (đọc/ghi, chỉ đọc).
  * Mẹo: File này chính xác hơn file /etc/fstab vì nó phản ánh trạng thái thực tế hiện tại, còn fstab chỉ là cấu hình dự kiến.
* /proc/meminfo: "Bảng tổng sắp" chi tiết về RAM (còn trống bao nhiêu, đang dùng làm cache bao nhiêu).
* /proc/cpuinfo: Thông số kỹ thuật của chip (Model, xung nhịp, số nhân).
* /proc/uptime: Máy đã chạy liên tục được bao lâu (tính bằng giây).

## 3. Sysfs (System Filesystem) - /sys
sysfs là một loại hệ thống file ảo (pseudofilesystem) được tích hợp sẵn trong nhân Linux (Kernel). Nó là một cơ chế mà Kernel dùng để xuất các thông tin về cấu trúc phần cứng, các driver và các bus (USB, PCI...) ra ngoài để người dùng và các ứng dụng có thể thấy được.
Trong quá trình khởi động, Linux sẽ thực hiện một lệnh "gắn" (mount) hệ thống file sysfs vào thư mục /sys này.
Nếu /proc là một "mớ hỗn độn" (vừa chứa tiến trình, vừa chứa phần cứng), thì /sys ra đời để chuyên biệt hóa cho phần cứng. /sys cũng là pseudofilesystem.
* Cấu trúc: Nó tổ chức phần cứng theo cấu trúc cây (phân cấp) rất rõ ràng.
* Mục đích: Cung cấp thông tin về các driver (trình điều khiển) và cấu trúc bus (USB, PCI, SATA).
* Sự khác biệt: Bạn sẽ không thấy các tiến trình (PID) ở đây. Đây thuần túy là bản đồ thiết bị của máy tính.

## 4. Udev và thư mục /dev
Đây là nơi chứa các "file thiết bị" thực sự để người dùng/phần mềm tương tác.
* udev là gì? Nó là một trình quản lý thiết bị chạy ngầm (daemon). Khi bạn cắm một cái USB vào:
  * Kernel phát hiện ra phần cứng mới.
  * udev nhận tín hiệu và tạo ra một file tên là /dev/sdb1 (chẳng hạn).
  * Nó gán quyền truy cập để bạn có thể mở USB đó lên.
* Hotplug/Hotswap: Khả năng cắm/rút thiết bị khi máy đang chạy mà không cần khởi động lại, nhờ sự linh hoạt của udev.

## 5. D-Bus (Desktop Bus) - Cầu nối thông tin
Hãy tưởng tượng udev là người thợ thông báo "Có USB vừa cắm vào!", thì D-Bus là hệ thống loa phóng thanh truyền tin đó đến các ứng dụng trên màn hình (Desktop).
* Luồng hoạt động: Phần cứng (USB) -> Kernel -> udev -> D-Bus -> Ứng dụng (File Manager hiện cửa sổ USB).
* Ví dụ: khi gắn mount ổ CD-ROM vào Linux, hệ thống 'udev' sử dụng D-Bus để thông báo với trình quản lý cửa sổ đang chạy để hiển thị một biểu tượng lên trên màn hình desktop.

# Xác định và Cấu hình Cài đặt Phần cứng (Công cụ và Tiện ích để Tìm Hiểu Thiết bị Hệ thống Linux)


