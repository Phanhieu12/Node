# ✅ CÁCH CÀI WSL TRỰC TIẾP LÊN Ổ KHÁC (VD: D:\WSL\Ubuntu)

---

## 🧩 Bước 1: Kích hoạt WSL và các thành phần cần thiết
1. Mở **PowerShell** với quyền Administrator.
2. Chạy lần lượt các lệnh sau để kích hoạt WSL và Virtual Machine Platform:

   **Kích hoạt tính năng WSL**:
   ```powershell
   dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
   ```

   **Kích hoạt tính năng Virtual Machine Platform (cần cho WSL 2)**:
   ```powershell
   dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
   ```

3. **Khởi động lại máy tính** để áp dụng các thay đổi.

---

## 🧩 Bước 2: Cài đặt WSL
1. Sau khi khởi động lại, chạy lệnh sau để cài đặt WSL:
   ```powershell
   wsl --install --no-distribution
   ```
   Lệnh này sẽ cài đặt WSL mà không cài đặt bất kỳ bản phân phối Linux nào.

---

## 🧩 Bước 3: Đặt WSL 2 làm phiên bản mặc định
1. Đảm bảo rằng WSL 2 được sử dụng làm phiên bản mặc định:
   ```powershell
   wsl --set-default-version 2
   ```

---

## 🧩 Bước 4: Tải file Ubuntu rootfs (dạng `.tar.gz`)
1. Truy cập trang:  
   👉 [https://cloud-images.ubuntu.com/wsl/](https://cloud-images.ubuntu.com/wsl/)
2. Chọn phiên bản Ubuntu bạn muốn (ví dụ: `focal`, `jammy`, ...).
3. Tải file `.tar.gz` (vd: `ubuntu-jammy-wsl-amd64.tar.gz`) về máy tính của bạn.  
   **Ví dụ:** Lưu tệp tại `D:\Downloads\ubuntu.tar.gz`.

---

## 🧩 Bước 5: Nhập bản phân phối Ubuntu vào ổ D:
1. Mở **PowerShell** với quyền Administrator.
2. Chạy lệnh sau:

   ```powershell
   wsl --import Ubuntu D:\WSL\Ubuntu D:\Downloads\ubuntu.tar.gz --version 2
   ```

   ### Ý nghĩa:
   - `Ubuntu`: Tên phân phối (bạn có thể đặt khác, ví dụ: `Ubuntu22`).
   - `D:\WSL\Ubuntu`: Thư mục cài đặt Ubuntu (toàn bộ hệ thống sẽ nằm ở đây).
   - `D:\Downloads\ubuntu.tar.gz`: File Ubuntu rootfs bạn đã tải.
   - `--version 2`: Chỉ định sử dụng WSL 2 (nhanh hơn, tối ưu hơn WSL 1).

---

## 🧩 Bước 6: Chạy Ubuntu đã cài
1. Để chạy Ubuntu, sử dụng lệnh:
   ```powershell
   wsl -d Ubuntu
   ```
2. Lần đầu tiên chạy, hệ thống sẽ yêu cầu bạn tạo **username** và **mật khẩu**.

---

## 🧩 Bước 7: Chạy WSL
1. Sau khi cài đặt xong, bạn có thể chạy WSL với lệnh mặc định:
   ```powershell
   wsl
   ```
   Lệnh này sẽ mở bản phân phối mặc định mà bạn vừa cài đặt (ví dụ: `Ubuntu`).
2. Nếu bạn có nhiều bản phân phối và muốn kiểm tra danh sách, sử dụng:
   ```powershell
   wsl --list --verbose
   ```
3. Nếu muốn thay đổi bản phân phối mặc định, sử dụng:
   ```powershell
   wsl --set-default <Tên_Bản_Phân_Phối>
   ```
   **Ví dụ:**  
   ```powershell
   wsl --set-default Ubuntu
   ```

---

## ✅ Ưu điểm của cách này:
1. **Kiểm soát vị trí lưu dữ liệu**: Bạn có thể chọn cài đặt trên ổ D, E, hoặc bất kỳ ổ nào khác.
2. **Không phụ thuộc Microsoft Store**: Cài đặt thủ công, không cần tải qua Store.
3. **Dễ dàng sao lưu**: Vì toàn bộ dữ liệu nằm trong một thư mục mà bạn chỉ định.

---

## 🧩 Lưu ý:
1. **Kiểm tra phiên bản Windows**: WSL 2 yêu cầu Windows 10 phiên bản 1903 (bản build 18362) trở lên. Bạn có thể kiểm tra phiên bản Windows của mình bằng cách chạy:
   ```powershell
   winver
   ```

2. **Cập nhật Kernel WSL**: Nếu kernel WSL đã lỗi thời, bạn có thể tải bản cập nhật từ trang Microsoft WSL: [https://aka.ms/wsl2kernel](https://aka.ms/wsl2kernel).

Nếu bạn vẫn gặp lỗi hoặc cần hỗ trợ thêm, hãy cung cấp thêm thông tin để tôi có thể giúp bạn chi tiết hơn!
