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
sudo apt install lz4 curl iptables build-essential git golang-go wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```
### A.Cài Go
```bash
sudo -v

# Bước 1: Kiểm tra và tạo thư mục /usr/local/go nếu chưa có
if [ ! -d "/usr/local/go" ]; then
  echo "Directory /usr/local/go does not exist. Creating it now."
  # Tải Go phiên bản 1.20
  wget https://dl.google.com/go/go1.20.linux-amd64.tar.gz
  # Giải nén vào /usr/local
  sudo tar -C /usr/local -xzf go1.20.linux-amd64.tar.gz
  # Xóa tệp tar.gz để giải phóng dung lượng
  rm go1.20.linux-amd64.tar.gz
fi

# Bước 2: Tạo liên kết mềm từ /usr/local/go/bin/go tới /usr/bin/go
sudo ln -sf /usr/local/go/bin/go /usr/bin/go

# Bước 3: Cập nhật biến môi trường PATH
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc

# Bước 4: Áp dụng thay đổi môi trường
source ~/.bashrc

# Bước 5: Kiểm tra phiên bản Go
go version
````
### B. Kiểm tra và tạo thư mục $HOME/go/bin nếu không tồn tại
```bash
# Kiểm tra và tạo thư mục $HOME/go/bin nếu không tồn tại
if [ ! -d "$HOME/go/bin" ]; then
  echo "Directory $HOME/go/bin does not exist. Creating it now."
  mkdir -p "$HOME/go/bin"
fi
````
### C. tải và cài Cosmovisor
```bash
cd ~
git clone https://github.com/cosmos/cosmos-sdk.git
cd cosmos-sdk
make cosmovisor
sudo mv build/cosmovisor /go/bin/
mkdir -p /go/bin/cosmos/{genesis,app_data}

#Cấu hình dịch vụ systemd cho Cosmovisor
sudo tee /etc/systemd/system/cosmovisor.service <<EOF
[Unit]
Description=Your Cosmos SDK Application
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/cosmovisor start --home /var/lib/cosmos
Restart=always
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF

#Kiểm tra hoạt động
sudo systemctl daemon-reload
sudo systemctl enable cosmovisor.service
sudo systemctl start cosmovisor.service
sudo systemctl status cosmovisor.service

````
2.2. **Tải xuống và cài đặt story-geth**
```bash
# Tải xuống và cài đặt
cd $HOME
wget https://github.com/piplabs/story-geth/releases/download/v0.10.1/geth-linux-amd64
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin
if ! grep -q "$HOME/go/bin" $HOME/.profile; then
  echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.profile
fi
chmod +x geth-linux-amd64
mv $HOME/geth-linux-amd64 $HOME/go/bin/story-geth
source $HOME/.profile
story-geth version
````
2.3. **Tải xuống và cài đặt story**
```bash
# Tải xuống và giải nén tệp cài đặt
cd $HOME
rm -rf story-linux-amd64
wget https://github.com/piplabs/story/releases/download/v0.12.1/story-linux-amd64
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin
if ! grep -q "$HOME/go/bin" $HOME/.profile; then
  echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.profile
fi
chmod +x story-linux-amd64
sudo cp $HOME/story-linux-amd64 $HOME/go/bin/story
source $HOME/.profile
story version
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
ExecStart=/root/go/bin/story-geth --odyssey --syncmode full
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
Description=Story Consensus Client
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/story run
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

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
