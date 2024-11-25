# Story Odyssey Setup

## **Story Repositories**
- [Story Releases](https://github.com/piplabs/story/releases/)
- [Story-Geth Releases](https://github.com/piplabs/story-geth/releases)

## **Faucet**
- [Story faucet](https://faucet.story.foundation/)
- [Faucet Me](https://story.faucetme.pro/)
---

# Hướng dẫn Cài đặt và Chạy Node cho Dự án Story

Trước khi bắt đầu, hãy đảm bảo rằng bạn đã cài đặt các yêu cầu hệ thống sau:

### **Thông số Phần cứng**

| **Yêu cầu** | **Chi tiết** |
|-------------|--------------|
| **CPU**     | 4 Cores      |
| **RAM**     | 8 GB         |
| **Ổ đĩa**   | 200 GB       |
| **Băng thông** | 10 MBit/s  |

### **Ghi chú**

- Đảm bảo hệ thống của bạn đáp ứng các yêu cầu trên để đảm bảo hiệu suất tối ưu cho việc chạy phần mềm và dịch vụ.
- Các yêu cầu này có thể thay đổi tùy theo phiên bản phần mềm và khối lượng công việc.
- Hệ điều hành: Ubuntu 22.04
- Các công cụ cơ bản: `curl`, `git`, `make`, `jq`, `build-essential`, `gcc`, `unzip`, `wget`, `lz4`, `aria2`

## 2. Cài đặt

### 2.1. Cài đặt các công cụ cần thiết

```bash
sudo apt update
sudo apt-get update
sudo apt install lz4 curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```
2.1.1 # **Tạo thư mục $HOME/go/bin nếu nó chưa tồn tại**
```bash
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin

# Thêm $HOME/go/bin vào PATH nếu nó chưa có trong .bashrc
if ! grep -q "$HOME/go/bin" $HOME/.bashrc; then
  echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.bashrc
  source $HOME/.bashrc
fi
````
2.2. **Tải xuống và cài đặt story-geth**
```bash
# Tải xuống và cài đặt
wget https://github.com/piplabs/story-geth/releases/download/v0.10.1/geth-linux-amd64 && \
mv $HOME/geth-linux-amd64 $HOME/go/bin/story-geth && \
chmod +x $HOME/go/bin/story-geth && \
story-geth version
````
2.3. **Tải xuống và cài đặt story**
```bash
# Tải xuống và giải nén tệp cài đặt
wget https://github.com/piplabs/story/releases/download/v0.12.0/story-linux-amd64 -O /root/go/bin/story && \
chmod +x /root/go/bin/story && \
/root/go/bin/story version
````

2.4. **Khởi tạo Story**
```bash
# Thay "Your_moniker_name" thành tên muốn đặt
story init --network iliad --moniker "Your_moniker_name"
````
3. **Cấu hình Dịch vụ**
3.1. **Cấu hình dịch vụ story-geth**
```bash
# lưu ý check gõ pwd là gì thì đổi user="" và pach đổi/root/ thành /user/
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/story-geth --iliad --syncmode full
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
````
3.2. **Cấu hình dịch vụ story**
```bash
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Story Service
After=network.target

[Service]
ExecStart=/root/go/bin/story run
WorkingDirectory=/root
User=root
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
````
4. **Quản lý Dịch vụ**  
4.1. **Khởi động dịch vụ**
```bash
sudo systemctl daemon-reload
sudo systemctl start story-geth.service
sudo systemctl start story.service
````
4.1.1 **Check trạng thái**
```bash
#check trạng thái của  story-geth
sudo systemctl status story-geth.service

#check trạng thái của  story
sudo systemctl status story.service
````
4.2. **Dừng dịch vụ**
```bash
sudo systemctl stop story-geth.service
sudo systemctl stop story.service
````
4.3. **Kích hoạt dịch vụ tự động khởi động cùng hệ thống**
```bash
sudo systemctl enable story-geth.service
sudo systemctl enable story.service
````
6. **Khắc phục sự cố**  
Lỗi không tìm thấy lệnh story-geth hoặc story: Kiểm tra rằng biến môi trường
```bash
PATH
đã được thiết lập đúng cách và story-geth và story đã được sao chép vào thư mục $HOME/go/bin chưa?
````
Lỗi khi khởi động dịch vụ: Kiểm tra logs dịch vụ bằng lệnh 
```bash
journalctl -u story-geth.service
journalctl -u story.service
````
