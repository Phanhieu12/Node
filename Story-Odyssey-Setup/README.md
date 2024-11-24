# Story Odyssey Setup

## **Story Repositories**
- [Story Releases](https://github.com/piplabs/story/releases/)
- [Story-Geth Releases](https://github.com/piplabs/story-geth/releases)

---

## **I. Prerequisites**

1. **Update system and install dependencies**
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
   
## **II. Prerequisites**
1. **Create story folder and move there**
   ```bash
   mkdir story && \
   cd story

2. **Create go/bin folders, if needed:**
   ```bash
   mkdir -p $HOME/go/bin

3. **Install pre-built Story-Geth binary:**
   ```bash
   wget https://github.com/piplabs/story-geth/releases/download/v0.10.1/geth-linux-amd64 &&\
   mv $HOME/story/geth-linux-amd64 $HOME/go/bin/story-geth && \
   chmod +x $HOME/go/bin/story-geth && \
   story-geth version

4. **Install pre-built Story binary:**
   ```bash
   wget https://github.com/piplabs/story/releases/download/v0.13.0/story-linux-amd64 && \
   tar -xvf story-linux-amd64 && \
   mv $HOME/story/story-linux-amd64/story $HOME/go/bin/story && \
   rm -rf story* && \
   story version

5. **Initialize node:**
   Change `"MONIKER"` for your node name.
   ```bash
   story init --moniker "MONIKER" --network iliad

6. **Check genesis:**
   ```bash
   sha256sum ~/.story/story/config/genesis.json

- [Output should be:](18ab598bbaefaa5af5e998abe14e8660ff6fa3c63a9453f5f40f472b213ed091 /root/.story/story/config/genesis.json)
   
7. **Check validator state:**
   ```bash
   cd && cat .story/story/data/priv_validator_state.json

   Should be at the beginning:

   {
     "height": "0",
     "round": 0,
     "step": 0
   }
   If not, run:
   
   story tendermint unsafe-reset-all --home $HOME/.story/story

8. **Thiết lập địa chỉ bên ngoài**
   ```bash
   external_address=$(wget -qO- eth0.me)
   sed -i.bak -e "s/^external_address *=.*/external_address =                         \"$external_address:26656\"/" $HOME/.story/story/config/config.toml

10. **Thêm seeds & peers**
   ```bash
   eers="90161a7f82ce5dbfbed1a2a9d40d4103730cff0f@5.9.87.231:26656,2f372238bf86835e8ad68c0db12351833c40e8ad@story-testnet-peer.itrocket.net:26656,14ab123d59ddf69769627b3f0e7438320f7a280a@100.42.180.223:26656,e96d4dfe2871aa44a5d97bca9ac585ad16647503@84.46.255.69:26656,bb84a8e391ff9ae2d95a3ad1ab10682d39cae583@109.123.241.100:26656,ddec0d321e85749763b89a0d7fbb58f2e065fe5e@195.133.0.86:26656,cbb1693adf93b389fc66aa1443f8b542798b564a@194.233.90.165:26656,58d9968cce8cc34f3c7aa81fa51db8af4eed0e11@62.112.10.13:29657,ef9d67cd77cec42e934ee571d6092341be4ed67b@65.109.36.231:14656,cf547fa20d73025357103133043d4c0a1da7f56d@188.245.121.171:26656"
   sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$peers\"|" $HOME/.story/story/config/config.toml
   seeds="6a07e2f396519b55ea05f195bac7800b451983c0@story-seed.mandragora.io:26656"
   sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.story/story/config/config.toml

11. **Thiết lập addrbook bởi ITRocket**
wget -O $HOME/.story/story/config/addrbook.json https://server-5.itrocket.net/testnet/story/addrbook.json

12. **Thêm số lượng tối đa inbound/outbound peers**
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 40/g' $HOME/.story/story/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.story/story/config.toml

13. **Thêm lọc peers "xấu"**
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.story/story/config.toml

14. **Tạo tệp dịch vụ story**
tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Story Consensus Client
After=network.target
[Service]
User=$USER
WorkingDirectory=$HOME/.story/story
ExecStart=$HOME/go/bin/story run
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

15. **Tạo tệp dịch vụ story-geth**
tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target
[Service]
User=$USER
ExecStart=$HOME/go/bin/story-geth --iliad --syncmode full --http --http.api eth,net,web3,engine --http.vhosts '*' --http.addr 127.0.0.1 --http.port 8545 --ws --ws.api eth,web3,net,txpool --ws.addr 127.0.0.1
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

16. **Tải lại daemon, kích hoạt story & story-geth, khởi động story & story-geth**
systemctl daemon-reload && \
systemctl enable story && \
systemctl enable story-geth && \
systemctl start story && \
systemctl start story-geth

17. **Kiểm tra trạng thái story**
systemctl status story

18. **Kiểm tra trạng thái story-geth**
systemctl status story-geth

19. **Kiểm tra nhật ký**
journalctl -u story -f -o cat
journalctl -u story-geth -f -o cat

20. **Snapshot bởi UTSA**
systemctl stop story && \
systemctl stop story-geth && \
cp $HOME/.story/story/data/priv_validator_state.json $HOME/.story/story/priv_validator_state.json.backup && \
rm -rf $HOME/.story/story/data && \
rm -rf $HOME/.story/geth/iliad/geth/chaindata && \
curl -o - -L https://share102.utsa.tech/story/story_testnet.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.story/story/ && \
curl -o - -L https://share102.utsa.tech/story/story_geth_testnet.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.story/geth/iliad/geth/ && \
mv $HOME/.story/story/priv_validator_state.json.backup $HOME/.story/story/data/priv_validator_state.json && \
systemctl daemon-reload && \
systemctl restart story && \
systemctl restart story-geth

## **III. Prerequisites**
Kiểm tra khóa EVM và xuất khóa riêng vào thư mục cấu hình
story validator export --export-evm-key

Tạo tệp .env để thực hiện các giao dịch như validator
story validator export --export-evm-key --evm-key-path $HOME/.story/story/config/.env

Nhận token từ Story faucet hoặc Faucet Me

Tạo validator
story validator create --stake 1000000000000000000

Các lệnh hữu ích
Kiểm tra trạng thái node
curl localhost:26657/status | jq .result.sync_info

Staking Validator
story validator stake \
   --validator-pubkey ${VALIDATOR_PUB_KEY_IN_BASE64} \
   --stake ${AMOUNT_TO_STAKE_IN_WEI}

Unstaking Validator
story validator unstake \
  --validator-pubkey ${VALIDATOR_PUB_KEY_IN_BASE64} \
  --unstake ${AMOUNT_TO_UNSTAKE_IN_WEI}

Stake-on-behalf Validator
story validator stake-on-behalf \
  --delegator-pubkey ${DELEGATOR_PUB_KEY_IN_BASE64} \
  --validator-pubkey ${VALIDATOR_PUB_KEY_IN_BASE64} \
  --stake ${AMOUNT_TO_STAKE_IN_WEI}

Unstake-on-behalf Validator
story validator unstake-on-behalf \
  --delegator-pubkey ${DELEGATOR_PUB_KEY_IN_BASE64} \
  --validator-pubkey ${VALIDATOR_PUB_KEY_IN_BASE64} \
  --unstake ${AMOUNT_TO_UNSTAKE_IN_WEI}

Thêm Operator
story validator add-operator \
  --operator ${OPERATOR_EVM_ADDRESS}

Xóa Operator
story validator remove-operator \
  --operator ${OPERATOR_EVM_ADDRESS}

Xóa Node
sudo systemctl stop story && \
sudo systemctl stop story-geth && \
sudo systemctl disable story && \
sudo systemctl disable story-geth && \
sudo rm /etc/systemd/system/story.service && \
sudo rm /etc/systemd/system/story-geth.service && \
sudo systemctl
