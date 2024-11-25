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
| **RAM**     | 16 GB         |
| **Ổ đĩa**   | 240 GB       |
| **Băng thông** | 10 MBit/s  |

### **Ghi chú**

- Đảm bảo hệ thống của bạn đáp ứng các yêu cầu trên để đảm bảo hiệu suất tối ưu cho việc chạy phần mềm và dịch vụ.
- Các yêu cầu này có thể thay đổi tùy theo phiên bản phần mềm và khối lượng công việc.
- Hệ điều hành: Ubuntu 22.04
- Các công cụ cơ bản: `curl`, `tar`, `wget`, `clang`, `pkg-config`, `git`, `make`, `ncdu`, `gcc`, `jq`, `chrony`

## 2. Cài đặt

### 2.1. Cài đặt các công cụ cần thiết
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

### 2.1. Cài đặt Tmux
```bash
sudo apt update
sudo apt install -y tmux
```
- Sử dụng cơ bản
```bash
Tạo phiên mới:

tmux
```
```bash
Đặt tên cho phiên:

tmux new -s ten_phien
```

```bash
Danh sách các phiên đang chạy:

tmux ls
```
```bash
Kết nối lại phiên:

tmux attach -t ten_phien
```
- Thoát mà không đóng phiên: Nhấn Ctrl+b rồi nhấn d.
### 2.3. Cài Go 1.22.2
```bash
ver="1.22.2"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
````
**2.2. Tải xuống và cài đặt story-geth,story**
```bash
cd $HOME && mkdir -p go/bin/
wget -O geth https://github.com/piplabs/story-geth/releases/download/v0.10.1/geth-linux-amd64
chmod +x geth
mv ~/geth ~/go/bin/geth
git clone https://github.com/piplabs/story
cd $HOME/story
git checkout v0.12.1
go build -o story ./client
sudo mv $HOME/story/story $HOME/go/bin/
mkdir -p ~/.story/story
mkdir -p ~/.story/geth
````

**2.2.1 Check story version**
```bash
story version

Result(Kết quả)
- version: v0.12.1-stable
- git commit: 20fed5e
````
**2.2.1 Check story version**
```bash
geth version

Result(Kết quả)
- version: 0.10.1-stable
- git commit: b60a3ba8d47e60a6c78ca0570f7dac66e8976d93
- git commit date: 20241025
````

**2.3 Khởi tạo Story**
```bash
# Thay "moniker" thành tên muốn đặt
story init --network odyssey --moniker <moniker>
````

**2.4 Tải xuống genesis**
```bash
wget -O $HOME/.story/story/config/genesis.json https://raw.githubusercontent.com/Shoni-O/files/refs/heads/main/testnet-files/story/genesis.json

#Check
sha256sum $HOME/.story/story/config/genesis.json

Result(Kết quả)
d332e9082222cc0dd6fe4e9943eafc89b2ce5e118a75ffa01b77e549fdd12587
````

**2.5 Tải xuống addrbook**
```bash
wget -O $HOME/.story/story/config/addrbook.json https://raw.githubusercontent.com/Shoni-O/files/refs/heads/main/testnet-files/story/addrbook.json
````

**2.6 Bật prometheus và tắt lập chỉ mục**
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.story/story/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.story/story/config/config.toml
````

**3. Cấu hình Dịch vụ**  
**3.1. Cấu hình dịch vụ story-geth**
```bash
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/geth --odyssey --syncmode full --http --http.api eth,net,web3,engine --http.vhosts '*' --http.addr 127.0.0.1 --http.port 8545 --ws --ws.api eth,web3,net,txpool --ws.addr 127.0.0.1 --ws.port 8546
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
````

**3.2. Cấu hình dịch vụ story**
```bash
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Story Consensus Client
After=network.target

[Service]
User=$USER
WorkingDirectory=$HOME/.story/story
ExecStart=$HOME/go/bin/story run
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
````

**4. Khởi động dịch vụ**  
```bash
sudo systemctl daemon-reload
sudo systemctl enable story-geth
sudo systemctl enable story
sudo systemctl restart story-geth && sudo journalctl -fu story-geth -o cat
sudo systemctl restart story && sudo journalctl -fu story -o cat
````

**5. Validator**  
**5.1 Tạo**
- Để tạo trình xác thực, bạn cần có 1024 mã thông báo.

- Làm thế nào để nhận được 1024 mã thông báo?

- Bạn cần chạy nút, để nó đồng bộ hóa mạng. Sau đó đợi biểu mẫu vào ngày 25 và điền vào nó. Nếu bạn được chọn, bạn sẽ nhận được đủ để tạo / đăng ký trình xác thực của mình vào mạng Odyssey.
```bash
story validator create --stake 1500000000000000000000 --moniker STAVR_Guide --private-key $(cat $HOME/.story/story/config/private_key.txt | grep "PRIVATE_KEY" | awk -F'=' '{print $2}')
````

**5.1 Kiểm tra trạng thái nút**
```bash
curl localhost:26657/status | jq
or
story status
````

**5.2 Kiểm tra trạng thái đồng bộ hóa**
```bash
curl localhost:26657/status | jq
or
story status
````

**5.1 Ví**
```bash
story validator export
````

**5.1 Xóa nút**
```bash
sudo systemctl stop story-geth story
sudo systemctl disable story-geth story
rm /etc/systemd/system/story-geth.service /etc/systemd/system/story.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .story story
rm -rf $(which story)
rm -rf $(which geth)
````
