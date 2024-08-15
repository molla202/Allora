BTİMEDİ

## 💻 Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	4 or 8 |
| RAM	| 8+ or 16+ GB |
| Storage	| 400 GB SSD |

### Explorer



### Public RPC and Explorer

SOON...

### 🚧Gerekli kurulumlar
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### 🚧 Go kurulumu
```
cd $HOME
VER="1.22.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### 🚧 Dosyaları çekelim ve kuralım

```
cd $HOME
git clone https://github.com/allora-network/allora-chain.git
cd allora-chain
make build
```
```
mkdir -p $HOME/.allorad/cosmovisor/genesis/bin
mv /root/allora-chain/build/allorad $HOME/.allorad/cosmovisor/genesis/bin/
```
```
sudo ln -s $HOME/.allorad/cosmovisor/genesis $HOME/.allorad/cosmovisor/current -f
sudo ln -s $HOME/.allorad/cosmovisor/current/bin/allorad /usr/local/bin/allorad -f
```
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```
### 🚧 Servis oluşturalım
```
sudo tee /etc/systemd/system/allorad.service > /dev/null << EOF
[Unit]
Description=allora node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.allorad"
Environment="DAEMON_NAME=allorad"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.allorad/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable allorad
```
### 🚧 İnit
```
allorad init yaz-bura --chain-id allora-testnet-1 --default-denom uallo
```
```
allorad config set client chain-id allora-testnet-1
allorad config set client keyring-backend test
allorad config set client node tcp://localhost:51657
```
* Moniker adını yaz

### 🚧 Genesis addrbook
```
curl -Ls https://raw.githubusercontent.com/allora-network/networks/main/allora-testnet-1/genesis.json > $HOME/.allorad/config/genesis.json
```
### 🚧 Gas ayarı
```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0uallo\"|" $HOME/.allorad/config/app.toml
```
### 🚧 Port ayarı
```
echo "export G_PORT="51"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
sed -i.bak -e "s%:1317%:${G_PORT}317%g;
s%:8080%:${G_PORT}080%g;
s%:9090%:${G_PORT}090%g;
s%:9091%:${G_PORT}091%g;
s%:8545%:${G_PORT}545%g;
s%:8546%:${G_PORT}546%g;
s%:6065%:${G_PORT}065%g" $HOME/.allorad/config/app.toml
```
```
sed -i.bak -e "s%:26658%:${G_PORT}658%g;
s%:26657%:${G_PORT}657%g;
s%:6060%:${G_PORT}060%g;
s%:26656%:${G_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${G_PORT}656\"%;
s%:26660%:${G_PORT}660%g" $HOME/.allorad/config/config.toml
```
### 🚧 Peer
```
SEEDS="b825e8d944952e50afe5739fa4afa7cb16f11db9@seed-0-p2p.testnet-1.testnet.allora.network:32110,cc11f2c02f9dea5f3b86eec3e00ae373b7076ea4@seed-1-p2p.testnet-1.testnet.allora.network:32111"
PEERS="11413d234e449ff3fefbad2df285a9b2b2601e0d@peer-0.testnet-1.testnet.allora.network:32120,89ec173c61da9b32c7344aacfe72cc62e0b743a0@peer-1.testnet-1.testnet.allora.network:32121"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.allorad/config/config.toml
```
### config pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
  $HOME/.allorad/config/app.toml
```
### 🚧 Snap
```
mv $HOME/.selfchain/data/priv_validator_state.json $HOME/.selfchain/priv_validator_state.json.backup 

selfchaind tendermint unsafe-reset-all --home $HOME/.selfchain --keep-addr-book 
curl http://37.120.189.81/selfchain_mainnet/selfchain_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME

cp $HOME/.selfchain/priv_validator_state.json.backup $HOME/.selfchain/data/priv_validator_state.json 
```


### 🚧 Başlatalım
Not: önce ufak bir peer ayarı gerekli
```
nano $HOME/.allorad/config/config.toml
```
mempool.max_txs_bytes 2097152
mempool.size 1000
```
sudo systemctl restart alloradd
journalctl -fu allorad -o cat
```


### 🚧 Cüzdan olusturalım
```
selfchaind keys add cüzdan-adi
```
### 🚧 Validator Olusturma
```
selfchaind tx staking create-validator \
    --amount=1000000uslf \
    --pubkey=$(selfchaind tendermint show-validator) \
    --moniker="moniker-adi-yaz" \
    --website="" \
    --details="" \
    --chain-id="self-1" \
    --commission-rate="0.10" \
    --commission-max-rate="0.15" \
    --commission-max-change-rate="0.05" \
    --min-self-delegation="1" \
    --gas="auto" \
    --gas-adjustment="1.4" \
    --gas-prices="0.005uslf" \
    --from="cüzdan-adi" \
    -y
```
### Komple Silme
```
sudo systemctl stop selfchaind
sudo systemctl disable selfchaind
sudo rm -rf /etc/systemd/system/selfchaind.service
sudo rm $(which selfchaind)
sudo rm -rf $HOME/.selfchain
sed -i "/SELFCHAIN_/d" $HOME/.bash_profile
```
