# ✅ CÁCH CÀI WSL TRỰC TIẾP LÊN Ổ KHÁC (VD: D:\WSL\Ubuntu)

## 🧩 Bước 1: Tải file Ubuntu rootfs (dạng `.tar.gz`)
1. Truy cập trang:  
   👉 [https://cloud-images.ubuntu.com/wsl/](https://cloud-images.ubuntu.com/wsl/)
2. Chọn phiên bản Ubuntu bạn muốn (ví dụ: `focal`, `jammy`, ...).
3. Tải file `.tar.gz` (vd: `ubuntu-jammy-wsl-amd64.tar.gz`) về máy tính của bạn.  
   **Ví dụ:** Lưu tệp tại `D:\Downloads\ubuntu.tar.gz`.

---

## 🧩 Bước 2: Nhập bản phân phối Ubuntu vào ổ D:
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

## 🧩 Bước 3: Chạy Ubuntu đã cài
Sau khi cài đặt xong, bạn có thể chạy Ubuntu bằng lệnh:

```powershell
wsl -d Ubuntu
```

- Lần đầu tiên chạy, hệ thống sẽ yêu cầu bạn tạo **username** và **mật khẩu**.

---

## ✅ Ưu điểm của cách này:
1. **Kiểm soát vị trí lưu dữ liệu**: Bạn có thể chọn cài đặt trên ổ D, E, hoặc bất kỳ ổ nào khác.
2. **Không phụ thuộc Microsoft Store**: Cài đặt thủ công, không cần tải qua Store.
3. **Dễ dàng sao lưu**: Vì toàn bộ dữ liệu nằm trong một thư mục mà bạn chỉ định.

---

Bạn có muốn mình giúp bạn chọn đúng bản Ubuntu `.tar.gz` và cung cấp lệnh đầy đủ phù hợp với hệ thống của bạn không?
