<h1 align="center"> KiiChain </h1>

![KiiChain Logo](https://hebbkx1anhila5yf.public.blob.vercel-storage.com/Imagen_Built_Large.jpg-obaj7YZ940LR5bCo3VGhDumhU6lq23.jpeg)

> Kiichain Testnet Oro Kurulum Kılavuzu

 * [Topluluk kanalımız](https://t.me/corenodechat)<br>
 * [Topluluk Twitter](https://twitter.com/corenodeHQ)<br>
 * [KiiChain Website](https://kiichain.io/)<br>
 * [Blockchain Explorer](https://explorer.corenodehq.com/kiichain)<br>
 * [Discord](https://discord.gg/y3MTxntHAd)<br>
 * [Twitter](https://x.com/KiiChainio)<br>

## 💻 Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	4+ |
| RAM	| 8+ GB |
| Storage	| +200 GB SSD |

### 🚧 Gerekli kurulumlar

```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### 🚧 Go kurulumu

```
cd $HOME
VER="1.22.10"
wget "https://go.dev/dl/go$VER.linux-amd64.tar.gz"
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
rm -rf kiichain
git clone https://github.com/KiiChain/kiichain.git
cd kiichain
git checkout v3.0.0
make build
```

```
cd
mkdir -p $HOME/.kiichain3/cosmovisor/upgrades/v3.0.0/bin
```

```
mv $HOME/kiichain/build/kiichaind $HOME/.kiichain3/cosmovisor/upgrades/v3.0.0/bin/
```

```
sudo ln -s $HOME/.kiichain3/cosmovisor/upgrades/v3.0.0 $HOME/.kiichain3/cosmovisor/current -f
sudo ln -s $HOME/.kiichain3/cosmovisor/current/bin/kiichaind /usr/local/bin/kiichaind -f
```

```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0
```

### 🚧 Servis oluşturalım

```
sudo tee /etc/systemd/system/kiichaind.service > /dev/null << EOF
[Unit]
Description=KiiChain node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --x-crisis-skip-assert-invariants --home $HOME/.kiichain3
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.kiichain3"
Environment="DAEMON_NAME=kiichaind"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.kiichain3/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable kiichaind
```

### 🚧 Init

```
kiichaind init "isim-yaz" --chain-id kiichain3
```

### 🚧 Port ayarı (İsteğe bağlı)

#### Port numarasını özelleştirmek isterseniz:

```
echo "export KIICHAIN_PORT="17"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### App toml port ayarları

```
sed -i.bak -e "s%:1317%:${KIICHAIN_PORT}317%g;
s%:8080%:${KIICHAIN_PORT}080%g;
s%:9090%:${KIICHAIN_PORT}090%g;
s%:9091%:${KIICHAIN_PORT}091%g;
s%:8545%:${KIICHAIN_PORT}545%g;
s%:8546%:${KIICHAIN_PORT}546%g;
s%:6065%:${KIICHAIN_PORT}065%g" $HOME/.kiichain3/config/app.toml
```

#### Config toml port ayarları

```
sed -i.bak -e "s%:26658%:${KIICHAIN_PORT}658%g;
s%:26657%:${KIICHAIN_PORT}657%g;
s%:6060%:${KIICHAIN_PORT}060%g;
s%:26656%:${KIICHAIN_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${KIICHAIN_PORT}656\"%;
s%:26660%:${KIICHAIN_PORT}660%g" $HOME/.kiichain3/config/config.toml
```

#### Client toml port ayarları

```
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${KIICHAIN_PORT}657\"|" $HOME/.kiichain3/config/client.toml
```

### 🚧 Genesis

```
wget https://raw.githubusercontent.com/KiiChain/testnets/refs/heads/main/testnet_oro/genesis.json -O $HOME/.kiichain3/config/genesis.json
```

### 🚧 Minimum gas price ayarı

```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.025ukii"|g' $HOME/.kiichain3/config/app.toml
```

### 🚧 Peer ayarları

```
SEEDS=""
PEERS="c541892972a552bdb6402ae6e2a4d9812021f39c@88.99.162.99:19656,5b6aa55124c0fd28e47d7da091a69973964a9fe1@uno.sentry.testnet.v3.kiivalidator.com:26656,5e6b283c8879e8d1b0866bda20949f9886aff967@dos.sentry.testnet.v3.kiivalidator.com:26656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.kiichain3/config/config.toml
```

### 🚧 Pruning ayarları

```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.kiichain3/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.kiichain3/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" $HOME/.kiichain3/config/app.toml
```

### 🚧 Indexer ayarları

```
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.kiichain3/config/config.toml
```

### 🚧 DB ve validator ayarları

#### Validator moduna geçiş

```
sed -i 's/mode = "full"/mode = "validator"/g' $HOME/.kiichain3/config/config.toml
```

#### DB ayarları

```
sed -i -e "s|^occ-enabled *=.*|occ-enabled = true|" $HOME/.kiichain3/config/app.toml
sed -i -e "s|^sc-enable *=.*|sc-enable = true|" $HOME/.kiichain3/config/app.toml
sed -i -e "s|^ss-enable *=.*|ss-enable = true|" $HOME/.kiichain3/config/app.toml
sed -i -e 's/^# concurrency-workers = 20$/concurrency-workers = 500/' $HOME/.kiichain3/config/app.toml
```

### 🚧 Snapshot

```
kiichaind tendermint unsafe-reset-all --home $HOME/.kiichain3
if curl -s --head curl http://37.120.189.81/kiichain_testnet/kiichainsnapshot.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl http://37.120.189.81/kiichain_testnet/kiichainsnapshot.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.kiichain3
else
  echo "Snapshot bulunamadı"
fi
```

### 🚧 Başlatalım

```
sudo systemctl restart kiichaind
journalctl -fu kiichaind -o cat
```

### 🚧 Cüzdan oluşturalım

```
kiichaind keys add cüzdan-adi
```

### 🚧 Cüzdan import

```
kiichaind keys add cüzdan-adi --recover
```

### 🚧 Validator oluşturma

NOT: Faucet için [Discord](https://discord.gg/y3MTxntHAd) kanalını ziyaret edin

#### validator dosyası oluştur

```
kiichaind tx staking create-validator \
  --from cüzdan-adi \
  --chain-id kiichain3 \
  --pubkey "$(kiichaind tendermint show-validator)" \
  --amount "1000000ukii" \
  --moniker "nodename" \
  --identity "" \
  --website "" \
  --details "details" \
  --commission-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-max-change-rate "0.01" \
  --min-self-delegation "1" \
  --gas auto \
  --gas-adjustment 1.5 \
  --gas-prices 0.025ukii
```

### 🚧 Delege

```
kiichaind tx staking delegate $(kiichaind keys show cüzdan-adi --bech val -a) 1000000ukii \
    --chain-id kiichain3 \
    --from "cüzdan-adi" \
    --gas auto --gas-adjustment 1.5 --gas-prices 0.025ukii
```

### 🚧 Node durumunu kontrol etme

#### Senkronizasyon durumu

`kiichaind status 2>&1 | jq .SyncInfo`

#### Validator durumu

`kiichaind status 2>&1 | jq .ValidatorInfo`

#### Node bilgileri

`kiichaind status 2>&1 | jq .NodeInfo`


### 🚧 Komple Silme
```
sudo systemctl stop kiichaind
sudo systemctl disable kiichaind
sudo rm -rf /etc/systemd/system/kiichaind.service
sudo rm $(which kiichaind)
sudo rm -rf $HOME/.kiichain3
sed -i "/KIICHAIN_/d" $HOME/.bash_profile
```
