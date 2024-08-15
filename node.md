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
mkdir -p $HOME/.allora/cosmovisor/genesis/bin
mv /root/allora-chain/build/allorad $HOME/.allora/cosmovisor/genesis/bin/
```
```
sudo ln -s $HOME/.allora/cosmovisor/genesis $HOME/.allora/cosmovisor/current -f
sudo ln -s $HOME/.allora/cosmovisor/current/bin/allorad /usr/local/bin/allorad -f
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
Environment="DAEMON_HOME=$HOME/.allora"
Environment="DAEMON_NAME=allorad"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.allora/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable selfchaind
```
### 🚧 İnit
```
allorad init yaz-bura --chain-id allora-testnet-1 --default-denom uallo
```
```
allorad config set client chain-id allora-testnet-1
allorad config set client keyring-backend test
allorad config set client node tcp://localhost:11757
```
* Moniker adını yaz

### 🚧 Genesis addrbook
```
curl -Ls https://raw.githubusercontent.com/allora-network/networks/main/allora-testnet-1/genesis.json > $HOME/.allora/config/genesis.json
```
### 🚧 Gas ayarı
```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0uallo\"|" $HOME/.selfchain/config/app.toml
```
### 🚧 Peer
```
SEEDS="b825e8d944952e50afe5739fa4afa7cb16f11db9@seed-0-p2p.testnet-1.testnet.allora.network:32110,cc11f2c02f9dea5f3b86eec3e00ae373b7076ea4@seed-1-p2p.testnet-1.testnet.allora.network:32111"
PEERS="11413d234e449ff3fefbad2df285a9b2b2601e0d@peer-0.testnet-1.testnet.allora.network:32120,89ec173c61da9b32c7344aacfe72cc62e0b743a0@peer-1.testnet-1.testnet.allora.network:32121"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.selfchain/config/config.toml
```
### config pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.selfchain/config/app.toml
```
### 🚧 Snap
```
mv $HOME/.selfchain/data/priv_validator_state.json $HOME/.selfchain/priv_validator_state.json.backup 

selfchaind tendermint unsafe-reset-all --home $HOME/.selfchain --keep-addr-book 
curl http://37.120.189.81/selfchain_mainnet/selfchain_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME

cp $HOME/.selfchain/priv_validator_state.json.backup $HOME/.selfchain/data/priv_validator_state.json 
```

### 🚧 Port ayarı
```
CUSTOM_PORT=117

sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.selfchain/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"localhost:9090\"%address = \"localhost:${CUSTOM_PORT}90\"%; s%^address = \"localhost:9091\"%address = \"localhost:${CUSTOM_PORT}91\"%" $HOME/.selfchain/config/app.toml
```
### 🚧 Başlatalım
Not: önce ufak bir peer ayarı gerekli
```
nano $HOME/.selfchain/config/config.toml
```
`handshake_timeout = "120s"`
`dial_timeout = "120s"`
```
sudo systemctl restart selfchaind
journalctl -fu selfchaind -o cat
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
