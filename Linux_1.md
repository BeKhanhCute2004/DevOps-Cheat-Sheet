# Xác định và cấu hình cài đặt phần cứng(Filesystems và Device Files)
## 1. Triết lý "Everything is a File" (Mọi thứ đều là file)
Trong Linux, bạn không tương tác trực tiếp với các dòng điện hay bảng mạch. Kernel (nhân hệ điều hành) đại diện mỗi thiết bị bằng một file.
* Lợi ích: Bạn có thể đọc dữ liệu từ chuột hoặc ghi dữ liệu vào ổ cứng bằng chính các công cụ đọc/ghi văn bản thông thường.
* Phân loại: Các file này thường nằm trong /dev (thiết bị thật) hoặc các "giả hệ thống file" (pseudofilesystems) như /proc và /sys để xem trạng thái.

## 2. Procfs (Process Filesystem) - /proc
Đây là một pseudofilesystem (hệ thống file ảo). Nó không chiếm không gian trên ổ cứng mà tồn tại trong RAM. Nó giống như một "cửa sổ" để bạn nhìn vào bên trong bộ não của Kernel.
Các thành phần chính:
* Thư mục số (PID): Ví dụ /proc/1. Mỗi con số là ID của một tiến trình đang chạy. Vào đây bạn sẽ thấy tiến trình đó đang dùng bao nhiêu RAM, chạy lệnh gì.
* Các file thông tin phần cứng:
  * /proc/interrupts: Xem các thiết bị đang "ngắt" CPU như thế nào (giống như việc các thiết bị giơ tay xin phát biểu).
  * /proc/dma: Xem thiết bị nào đang dùng quyền "truy cập bộ nhớ đặc cách" mà không làm phiền CPU.
  * /proc/mounts: Danh sách "những gì đang được gắn vào hệ thống" (ổ đĩa, phân vùng).
