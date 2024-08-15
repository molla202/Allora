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
echo "export A_PORT="51"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
sed -i.bak -e "s%:1317%:${A_PORT}317%g;
s%:8080%:${A_PORT}080%g;
s%:9090%:${A_PORT}090%g;
s%:9091%:${A_PORT}091%g;
s%:8545%:${A_PORT}545%g;
s%:8546%:${A_PORT}546%g;
s%:6065%:${A_PORT}065%g" $HOME/.allorad/config/app.toml
```
```
sed -i.bak -e "s%:26658%:${A_PORT}658%g;
s%:26657%:${A_PORT}657%g;
s%:6060%:${A_PORT}060%g;
s%:26656%:${A_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${A_PORT}656\"%;
s%:26660%:${A_PORT}660%g" $HOME/.allorad/config/config.toml
```
### 🚧 Peer
```
SEEDS=""
PEERS="97807ab57eb139b2a09f6474eadf7ec30e057dd6@84.247.169.217:26656,369720fcad05b77adde5b08d31036cc24efc50c5@34.65.32.55:26656,bbd865bdb404d3221a04372276155acc3c3e9358@89.116.24.22:26656,ad3a6b49bf0c3e082be5ab2b116e622b795b1f1e@188.40.66.173:26756,00e4e087ee0baa61637645c223447fd7ed39bfcb@16.62.224.211:26656,3668a9f02aa50e701d2bb027ec1176100998606b@46.4.95.49:26656,79af04335a0ac10073ef9342edba78e8fb08f9fc@89.58.0.245:37778,9463e19959923af9f04224601d4142562d4e4d29@158.220.122.189:26666,f97ccf1a972491df23709d3e644afe1fdcb025a9@89.58.38.62:26656,6c2304c75fa845077f2896a2dec7e1dbecf6abd2@160.202.130.93:32123,32868d8407cda134f27b3ab605405fc8d4bfb402@45.157.176.33:26656,23b57f3a051e251b16cf7cac7f430f6c2e68f32f@202.61.206.59:26656,59581cba1207d6ae32b6e2699dc82f97b9f74988@185.209.228.12:26656,debfed1b60f8639959a317939f56bd2a5ddae358@185.132.177.83:26656,870d2d7c6235b1bb3603dfd6069e90a8204318f3@95.217.196.224:26656,1ec4d1954ce3631274d57a9b60f5ffb5f9e4d841@66.70.177.125:27656,f8ad8646824b6f81a3c6bd6597852254b2b8d334@34.65.184.195:26656,52e689c56092722c2bc931f07b90cfd669604529@89.58.48.87:26656,54e77ab7453edf9d8fd4287299a6db2a3359eb81@147.135.140.50:26656,4628191c5fdda52f5d9b934b1b16662901c979ef@194.36.146.4:26656,b6c61a01c7ae734040196d1af0444f2793f41d4d@5.180.149.123:26656,70c4fd868dd6c57b4c1b9343f6c37c52a51590e0@51.81.109.67:26656,9c42acec6b573cac517607596fcc6210f3efea61@34.88.56.22:26656,289bff2d79868d0eb2c54c96fda45c89d4c34164@65.108.8.114:26656,0092f0b83c1a62505553eaada6b6659f3f461311@109.205.182.250:26666,7d548f78f0c67d391279c36fa9e127c52ce8b14c@65.108.225.207:55656,11413d234e449ff3fefbad2df285a9b2b2601e0d@160.202.130.73:32120,d3c79122924ff477e941ec0ca1ed775cfb01ca20@66.35.84.140:26656,5ac915ff123f0ae928b0a25e9b176809efbaf79c@89.58.8.108:26656,20b387e6b1190bc26fe8da81bec4eebaf7578ce3@65.109.22.228:26756,085108d28df386fbe850b9bd69fc4c6423cccd4d@85.10.211.28:26656,f3053878379eb2f88265efdf3eeef22a067492b1@2a01:26656,517d84e2dad613776bc08bcb203f240d393af295@148.251.86.17:26756,9cca620ee99e7d733baee084fd7b54273d9d6bdb@35.228.18.126:26656,0f6b64fcd38872d18a78d89e090a5e6928883d52@8.209.116.116:26656,7c681b1498762a9b1d48154f4d2bb6aac6c7503e@5.252.225.148:26656,c13dcf555ef6f71a8982bc38be0762d6d41e6e00@108.209.102.187:26643,f9a23479132eab419f3386ebb1a070bcfdf785ff@148.113.136.62:26656,6bbc4888c720760ebac79c8492411d8c1be1c07a@144.76.201.45:26656,eda4d538d5f3031c8297d71cd8c1bd3c25530a9a@37.120.176.115:26656,04449adf1c41cb8ee598b22e1a53977e43bce3f5@35.228.206.197:26656,e963d6e9740b890bc21fe4957067598f574f52b4@35.228.254.77:26656,e3107810aa46b196fa899e9309aecc7c97272522@34.125.70.129:26656,6ab4b2324edc03805c91a3684bd7d86d6bc5e158@80.64.208.205:26656,80d99208d70da18f8a587bda1023acea96343921@195.189.96.111:57456,7cbe61344cd74e2e2f8828a13bebdfa38f31550e@93.115.25.18:57456,6801766de19ca4231a01af66c145304c647cdd10@188.245.101.212:26656,467188d0291f3de996359c5e37fc28e9e94cf46e@152.53.21.5:26656,fb864d7d5386b010464fce05f5f4f308fe4ff54e@198.7.127.213:26656,3d7e5ccfa3dcc4beaeb7764dea6415d5969ca148@113.162.173.212:26656,1790346935cfff9738ea1aa6d37c5607a455a9f0@84.46.246.67:26656,e0f529d43ab666fa6fedc755b447a0e6bd3a5348@202.61.203.5:26656,fc5b0b6b815813dbaa049ad430a0ecda17317748@212.192.222.18:26656,d46fe907c56150cc9d4b385c7c6b1df91f6a980f@86.195.111.131:26666,79fa094a899f38ad59a5dc0916b95c46e87fd9e3@138.201.131.90:26656,7e14cf4e709b415c3f7cd54af32c397773465edf@202.61.251.222:26656,3c7ebf37080d4a5967aa07a2085f95345be1a87d@138.201.223.218:26656,67362c3ad125dadf3959970a76f8e5a13dbcb952@158.247.233.222:58200,22de02d0bc677c52db63194d764c399314c580a4@46.38.235.79:26656,3a7eb1cdefc0dcbb79eb143837a17260ab88eb87@212.126.35.132:26656,ed0e6f02831ccdc09dfb4e4e9f703d373486bc82@43.157.20.64:26656,3f2ddb4e50bc6aa61322192eeb5f37d10c1d167b@45.142.178.14:26656,d621c679303333bdb2a5effe37784775c64b2eda@170.75.251.230:26656,73a7ca949d425201bfc65fc593ce89e58908d5b8@34.229.252.167:32121,9e7c6c084fcbe500ab5af9852eb9468ff8de1f4d@37.27.56.125:26656,e40801e60d93cda37ca5e163d036231cde0fe924@88.198.52.46:26756,cd2a085c6c5e80e3e95b090584a443d8e184ba4c@94.16.120.206:26656,2d0ca438949c8bf1206359cfcf62ad994690780f@167.235.89.113:26656,801a02d6a51c298d3ddff1431312a8461e7f1b01@76.193.121.38:26656,cdf686d614c1b884a225f34a983789979fc73a85@89.163.133.89:26656,a4cc6b56c11b8f010b5a7dcf29aca58d71a099b2@15.204.101.34:26656,55bd1f4324e6cede1aa4cb7b06716c7a6913f0dd@46.38.233.43:26656,28278b616cb22797255643931a7a69ddb29d073d@89.58.55.104:26656,70cd2e3c0e168ba7face3bd3bc2eb60bee6fd7df@144.76.195.234:26656,89ec173c61da9b32c7344aacfe72cc62e0b743a0@103.88.232.90:32121,504a015fca790f94ee1498979248ce9718185c36@185.144.99.11:46656,a83385ffdaba46428134df2dcd1f9a298f83537a@37.221.193.21:26656,166e3bd8a94e8a7e1cefa180f9ae565afa528336@49.12.125.162:26656,23e700840b4e0e71342ba23353be8ed2be1c6a51@202.61.200.62:26656,9bf241d4193219d6c79885bd12d2be6df836d019@45.159.228.165:26666,38400eaf27e03e8c96fb817e3c4ff6faeadd9d6d@23.22.247.40:32120,4764a306e1dee4290ca7f0b82b4545d742f6c6c3@202.61.239.113:26656,21f44f233527edc5a7e1785a57e7ff27e2f586cf@198.7.119.28:26666,a7424fe43640733a4d153ca7ebf271b104c141f9@202.61.250.0:26656,d91f20e86fd247944f7346b199dbd861cf26cf6d@89.58.46.152:26656"
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
mv $HOME/.allorad/data/priv_validator_state.json $HOME/.allorad/priv_validator_state.json.backup 

allorad tendermint unsafe-reset-all --home $HOME/.allorad --keep-addr-book 
curl https://snapshots.polkachu.com/testnet-snapshots/allora/allora_2130038.tar.lz4 | tar -I lz4 -xf - -C $HOME/.allorad

cp $HOME/.allorad/priv_validator_state.json.backup $HOME/.allorad/data/priv_validator_state.json 
```


### 🚧 Başlatalım
Not: önce ufak bir ayar gerekli. değiştirip ctrl xy enter kaydet çık.
```
nano $HOME/.allorad/config/config.toml
```

max_txs_bytes = 2097152

cache_size = 1000

```
sudo systemctl restart allorad
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
