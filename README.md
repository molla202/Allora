![image](https://github.com/molla202/Allora/assets/91562185/70f5ed92-712d-4195-b924-0e712813165e)



 * [Topluluk kanalımız](https://t.me/corenodechat)<br>
 * [Topluluk Twitter](https://twitter.com/corenodeHQ)<br>
 * [Allora Website](https://www.allora.network/)<br>
 * [Blockchain Explorer](https://explorer.edgenet.allora.network/wallet/suggest)<br>
 * [Discord](https://t.co/AdXUVjS3iF)<br>
 * [Twitter](https://x.com/AlloraNetwork)<br>
 * [APP](https://app.allora.network/points/campaigns)<br>
 * [FAUCET](https://faucet.edgenet.allora.network/)<br>


## 💻 Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	2 çiğdemli |
| RAM	| 4 çikotayt |
| Storage	| 10+ GB SSD |


`UYARI VE BİLGİLENDİRME : BU DÖKÜMAN AŞAĞIDA VERDİĞİM LİNKTEN BAKILAR DÜZENLENMİŞTİR. BAZI PROJELERLE ÇAKIŞMAMASI İÇİN ÖZENLE HAZIRLANMIŞTIR.(PORT DEĞİŞTİRME İÇERİR) tOPLULUĞMUZDAN FURKAN ARKADAŞIMIZIN DÖKÜMANIDIR(KRİPTO UZMANI) BEN SADECE BAZI DEĞİŞİKLİKLER YAPTIM.`

`https://services.rpcdot.com/allora/worker-node`

### Gereklilikler
```
sudo apt update -y && sudo apt upgrade -y
```
```
sudo apt install ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev curl git wget make jq build-essential pkg-config lsb-release libssl-dev libreadline-dev libffi-dev gcc screen unzip lz4 -y
```
### Python3 yükleme
```
sudo apt install python3
sudo apt install python3-pip
```
### Docker yükleme
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```
sudo apt-get update
```
```
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
```
docker version
```
### Docker-Compose yükleme
```
VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)
```
```
curl -L "https://github.com/docker/compose/releases/download/"$VER"/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
```
chmod +x /usr/local/bin/docker-compose
```
```
docker-compose --version
```
### Allora cüzdan
```
git clone https://github.com/allora-network/allora-chain.git
cd allora-chain && make all
```
```
allorad version
```
### Cüzdan olusturma
```
allorad keys add testkey
```
YADA import aşağıdan
```
allorad keys add testkey --recover
```
### Faucet

* Baştaki linklerde faucet var alalım. aynı zamanda baştaki linklerde app var işlemleri bitirdikten sonra cüzdanı keplr ekleyin ve app sitesine bağlanın. bağlanırkene ağı ekler zaten keplr eklemesse explorer kısmından eklersiniz.

### Worker 
```
git clone https://github.com/allora-network/basic-coin-prediction-node
cd basic-coin-prediction-node
mkdir -p worker-data
mkdir -p head-data
```
* Dosya izini ver
```
sudo chmod -R 777 worker-data
sudo chmod -R 777 head-data
```
* Head key olustur
```
sudo docker run -it --entrypoint=bash -v ./head-data:/data alloranetwork/allora-inference-base:latest -c "mkdir -p /data/keys && (cd /data/keys && allora-keys)"
```
```
sudo docker run -it --entrypoint=bash -v ./worker-data:/data alloranetwork/allora-inference-base:latest -c "mkdir -p /data/keys && (cd /data/keys && allora-keys)"
```
### Dosyamızı güncelleyelim
```
rm -rf docker-compose.yml && nano docker-compose.yml
```
NOT: cüzdan kelimelerini yaz kısmına tabiki kelimeleri yaz :D
```
version: '3'

services:
  inference:
    container_name: inference-basic-eth-pred
    build:
      context: .
    command: python -u /app/app.py
    ports:
      - "8001:8000"
    networks:
      eth-model-local:
        aliases:
          - inference
        ipv4_address: 172.25.0.4
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/inference/ETH"]
      interval: 10s
      timeout: 10s
      retries: 12
    volumes:
      - ./inference-data:/app/data

  updater:
    container_name: updater-basic-eth-pred
    build: .
    environment:
      - INFERENCE_API_ADDRESS=http://inference:8000
    command: >
      sh -c "
      while true; do
        python -u /app/update_app.py;
        sleep 24h;
      done
      "
    depends_on:
      inference:
        condition: service_healthy
    networks:
      eth-model-local:
        aliases:
          - updater
        ipv4_address: 172.25.0.5

  worker:
    container_name: worker-basic-eth-pred
    environment:
      - INFERENCE_API_ADDRESS=http://inference:8000
      - HOME=/data
    build:
      context: .
      dockerfile: Dockerfile_b7s
    entrypoint:
      - "/bin/bash"
      - "-c"
      - |
        if [ ! -f /data/keys/priv.bin ]; then
          echo "Generating new private keys..."
          mkdir -p /data/keys
          cd /data/keys
          allora-keys
        fi
        # Change boot-nodes below to the key advertised by your head
        allora-node --role=worker --peer-db=/data/peerdb --function-db=/data/function-db \
          --runtime-path=/app/runtime --runtime-cli=bls-runtime --workspace=/data/workspace \
          --private-key=/data/keys/priv.bin --log-level=debug --port=9011 \
          --boot-nodes=/ip4/172.25.0.100/tcp/9010/p2p/$(cat /root/aloora-chain/basic-coin-prediction-node/head-data/keys/identity) \
          --topic=allora-topic-1-worker \
          --allora-chain-key-name=testkey \
          --allora-chain-restore-mnemonic='cüzdan kelimelerini yaz' \
          --allora-node-rpc-address=https://allora-rpc.edgenet.allora.network/ \
          --allora-chain-topic-id=1
    volumes:
      - ./worker-data:/data
    working_dir: /data
    depends_on:
      - inference
      - head
    networks:
      eth-model-local:
        aliases:
          - worker
        ipv4_address: 172.25.0.10

  head:
    container_name: head-basic-eth-pred
    image: alloranetwork/allora-inference-base-head:latest
    environment:
      - HOME=/data
    entrypoint:
      - "/bin/bash"
      - "-c"
      - |
        if [ ! -f /data/keys/priv.bin ]; then
          echo "Generating new private keys..."
          mkdir -p /data/keys
          cd /data/keys
          allora-keys
        fi
        allora-node --role=head --peer-db=/data/peerdb --function-db=/data/function-db  \
          --runtime-path=/app/runtime --runtime-cli=bls-runtime --workspace=/data/workspace \
          --private-key=/data/keys/priv.bin --log-level=debug --port=9010 --rest-api=:6000
    ports:
      - "6001:6000"
    volumes:
      - ./head-data:/data
    working_dir: /data
    networks:
      eth-model-local:
        aliases:
          - head
        ipv4_address: 172.25.0.100


networks:
  eth-model-local:
    driver: bridge
    ipam:
      config:
        - subnet: 172.25.0.0/24

volumes:
  inference-data:
  worker-data:
  head-data:
```
  
### Çalıştıralım
```
docker compose build
```
```
docker compose up -d
```
* Loglara bakmak için 
```
docker logs -f worker-basic-eth-pred -n 100
```
### Durum control
```
curl --location 'http://localhost:6001/api/v1/functions/execute' \
--header 'Content-Type: application/json' \
--data '{
    "function_id": "bafybeigpiwl3o73zvvl6dxdqu7zqcub5mhg65jiky2xqb4rdhfmikswzqm",
    "method": "allora-inference-function.wasm",
    "parameters": null,
    "topic": "1",
    "config": {
        "env_vars": [
            {
                "name": "BLS_REQUEST_PATH",
                "value": "/api"
            },
            {
                "name": "ALLORA_ARG_PARAMS",
                "value": "ETH"
            }
        ],
        "number_of_nodes": -1,
        "timeout": 2
    }
}'
```
### Update kontrol
```
curl http://localhost:8001/update
```
### Çıkarım kontrol ( 
```
curl http://localhost:8001/inference/ETH
```


















