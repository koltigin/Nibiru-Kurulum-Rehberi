# Nibiru Kurulum Rehberi

[Kaynak](https://nibiru.fi/docs/run-nodes/testnet/) / [Explorer](https://nibiru.explorers.guru/)
![Nibiru-GitHub](https://user-images.githubusercontent.com/102043225/221709752-3577df02-2f9d-42cd-b12d-a1593ae33ad3.jpg)

## Bağlantılar
 ✔️ [Website](https://nibiru.fi/)<br>
 ✔️ [Blockchain Explorer](https://nibiru.explorers.guru/)<br>
 ✔️ [Doküman](https://nibiru.fi/docs/run-nodes/testnet/)<br>
 ✔️ [Discord](https://discord.gg/nibiru)<br>
 ✔️ [Testnet Detayları](https://nibiru.fi/blog/posts/007-itn-1.html)<br>
 
## [Görevler](https://github.com/koltigin/Nibiru-Kurulum-Rehberi/blob/main/G%C3%B6revler.md)

## Gereksinimler 
| Bileşenler | Minimum Gereksinimler | **Tavsiye Edilen Gereksinimler** | 
| ------------ | ------------ | ------------ |
| CPU |	4 | 4 |
| RAM	| 8 GB | 16 GB |
| Storage	| 250 GB SSD | 1 TB SSD |

## Sistemi Güncelleme
```shell
apt update && apt upgrade -y
```

## Gerekli Kütüphanelerin Kurulması
```shell
apt install make clang pkg-config libssl-dev libclang-dev build-essential git curl ntp jq llvm tmux htop screen gcc lz4 -y < "/dev/null"
```

## Go Kurulumu
```shell
ver="1.20"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
rm -rf /usr/local/go
tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm -rf "go$ver.linux-amd64.tar.gz"
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

## Değişkenleri Yükleme
aşağıda değiştirmeniz gereken yerleri yazıyorum.
* `$NIB_NODENAME` validator adınız
* `$NIB_WALLET` cüzdan adınız
*  Eğer portu başka bir node kullanıyorsa aşağıdan değiştirebilirsiniz.
```shell
echo "export NIB_NODENAME=$NIB_NODENAME"  >> $HOME/.bash_profile
echo "export NIB_WALLET=$NIB_WALLET" >> $HOME/.bash_profile
echo "export NIB_PORT=11" >> $HOME/.bash_profile
echo "export NIB_CHAIN_ID=nibiru-itn-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Örnek
Node ve Cüzdan adımızın `Mehmet` olduğunu varsayalım. Kod aşağıdaki şekilde düzenlenecektir. 
```shell
echo "export NIB_NODENAME=Mehmet"  >> $HOME/.bash_profile
echo "export NIB_WALLET=Mehmet" >> $HOME/.bash_profile
echo "export NIB_PORT=18" >> $HOME/.bash_profile
echo "export NIB_CHAIN_ID=nibiru-itn-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Nibiru'un Kurulması

```shell
git clone https://github.com/NibiruChain/nibiru
cd nibiru || return
git checkout v0.19.2
make install
nibid version
```

## Uygulamayı Yapılandırma ve Başlatma
```shell
nibid config keyring-backend test
nibid config chain-id $NIB_CHAIN_ID
nibid init --chain-id $NIB_CHAIN_ID $NIB_NODENAME
```

## Genesis Dosyasının Kopyalanması
```shell
curl -s https://rpc.itn-1.nibiru.fi/genesis | jq -r .result.genesis > $HOME/.nibid/config/genesis.json
curl -s https://snapshots2-testnet.nodejumper.io/nibiru-testnet/addrbook.json > $HOME/.nibid/config/addrbook.json
```

## Minimum GAS Ücretinin Ayarlanması
```shell
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001unibi"|g' $HOME/.nibid/config/app.toml
```

## Indexer'i Kapatma (Opsiyonel)
```shell
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.nibid/config/config.toml
```

## SEED ve PEERS Ayarlanması
```shell
SEEDS="3f472746f46493309650e5a033076689996c8881@nibiru-testnet.rpc.kjnodes.com:39659,a431d3d1b451629a21799963d9eb10d83e261d2c@seed-1.itn-1.nibiru.fi:26656,6a78a2a5f19c93661a493ecbe69afc72b5c54117@seed-2.itn-1.nibiru.fi:26656"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.nibid/config/config.toml
```

## Prometheus'u Aktif Etme
```shell
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.nibid/config/config.toml
```

## Pruning'i Ayarlama
```shell
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.nibid/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.nibid/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.nibid/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.nibid/config/app.toml
```

## Portları Ayarlama
```shell
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${NIB_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${NIB_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${NIB_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${NIB_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${NIB_PORT}660\"%" $HOME/.nibid/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${NIB_PORT}317\"%; s%^address = \":8080\"%address = \":${NIB_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${NIB_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${NIB_PORT}091\"%" $HOME/.nibid/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${NIB_PORT}657\"%" $HOME/.nibid/config/client.toml
```

## Servis Dosyası Oluşturma
```shell
sudo tee /etc/systemd/system/nibid.service > /dev/null << EOF
[Unit]
Description=Nibiru Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which nibid) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```

## State Sync (Nodejumper)
```shell
cp $HOME/.nibid/data/priv_validator_state.json $HOME/.nibid/priv_validator_state.json.backup
nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book

SNAP_RPC="https://nibiru-testnet.nodejumper.io:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="a1b96d1437fb82d3d77823ecbd565c6268f06e34@nibiru-testnet.nodejumper.io:27656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.nibid/config/config.toml

sed -i 's|^enable *=.*|enable = true|' $HOME/.nibid/config/config.toml
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $HOME/.nibid/config/config.toml
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $HOME/.nibid/config/config.toml
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $HOME/.nibid/config/config.toml

mv $HOME/.nibid/priv_validator_state.json.backup $HOME/.nibid/data/priv_validator_state.json

curl -s https://snapshots2-testnet.nodejumper.io/nibiru-testnet/wasm.lz4 | lz4 -dc - | tar -xf - -C $HOME/.nibid/data
```

## Servisi Başlatma
```shell
sudo systemctl daemon-reload
sudo systemctl enable nibid
sudo systemctl start nibid
```

## Logları Kontrol Etme
```shell
journalctl -u nibid -f -o cat
```  

## Cüzdan Oluşturma

### Yeni Cüzdan Oluşturma
`$NIB_WALLET` bölümünü değiştirmiyoruz kurulumun başında cüzdanımıza değişkenler ile isim belirledik.
```shell 
nibid keys add $NIB_WALLET
```  

### Var Olan Cüzdanı İçeri Aktarma
```shell
nibid keys add $NIB_WALLET --recover
```

## Cüzdan ve Valoper Bilgileri
Burada cüzdan ve valoper bilgilerimizi değişkene ekliyoruz.

```shell
NIB_WALLET_ADDRESS=$(nibid keys show $NIB_WALLET -a)
NIB_VALOPER_ADDRESS=$(nibid keys show $NIB_WALLET --bech val -a)
```

```shell
echo 'export NIB_WALLET_ADDRESS='${NIB_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export NIB_VALOPER_ADDRESS='${NIB_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Faucet
[Nibiru](https://discord.gg/nibiru) adresine giderek `##💦︲faucet ` kanalından `$request cuzdan-adresi` şeklinde mesaj atarak token istiyoruz. 

🔴 **BU AŞAMADAN SONRA NODE'UMUZUN EŞLEŞMESİNİ BEKLİYORUZ.**

## Senkronizasyonu Kontrol Etme
`false` çıktısı almadıkça bir sonraki yani validator oluşturma adımına geçmiyoruz.
```shell
nibid status 2>&1 | jq .SyncInfo
```

🔴 **Eşleşme tamamlandıysa aşağıdaki adıma geçiyoruz.**


## Validator Oluşturma
 Aşağıdaki komutta aşağıda berlirttiğim yerler dışında bir değişiklik yapmanız gerekmez;
   - `identity`  burada `XXXX1111XXXX1111` yazan yere [keybase](https://keybase.io/) sitesine üye olarak size verilen kimlik numaranızı yazıyorsunuz.
   - `details` `Always forward with the Anatolian Team 🚀` yazan yere kendiniz hakkında bilgiler yazabilirsiniz.
   - `website`  `https://anatolianteam.com` yazan yere varsa bir siteniz ya da twitter vb. adresinizi yazabilirsiniz.
   - `security-contact`  E-posta adresiniz.
 ```shell 
nibid tx staking create-validator \
--amount=10000000unibi \
--pubkey=$(nibid tendermint show-validator) \
--moniker=$NIB_NODENAME \
--chain-id=$NIB_CHAIN_ID \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation="1" \
--gas-prices=0.1unibi \
--gas-adjustment=1.5 \
--gas=auto \
--from=$NIB_WALLET \
--details="Always forward with the Anatolian Team 🚀" \
--security-contact="xxxxxxx@gmail.com" \
--website="https://anatolianteam.com" \
--identity="XXXX1111XXXX1111" \
--yes
```

## Validator Rolü Alma
[Discord'da](https://discord.gg/nibiru) `##validator-role-request` sayfasına giderek validatorumuze ait exlporer linkini gönderiyoruz.

## Pricefeeder Kurulumu

### Dosyaları İndirme
```
curl -s https://get.nibiru.fi/pricefeeder! | bash
```

### Servis Dosyasını Hazırlama

Aşağıdaki kodları girmeden önce şunları düzenlemelisiniz;<br>
`WEBSOCKET_ENDPOINT` eğer portunuz farklı ise düzemleyiniz.
`FEEDER_MNEMONIC` bu bölüme cüzdan kelimelerinizi (mnemonic) yazacaksınız
`VALIDATOR_ADDRESS` bu kısıma da `nibi1valoper` ile başlayan validator adresinizi yazacaksınız.
```
export CHAIN_ID="nibiru-itn-1"
export GRPC_ENDPOINT="localhost:9090"
export WEBSOCKET_ENDPOINT="ws://localhost:26657/websocket"
export EXCHANGE_SYMBOLS_MAP='{ "bitfinex": { "ubtc:uusd": "tBTCUSD", "ueth:uusd": "tETHUSD", "uusdt:uusd": "tUSTUSD" }, "binance": { "ubtc:uusd": "BTCUSD", "ueth:uusd": "ETHUSD", "uusdt:uusd": "USDTUSD", "uusdc:uusd": "USDCUSD", "uatom:uusd": "ATOMUSD", "ubnb:uusd": "BNBUSD", "uavax:uusd": "AVAXUSD", "usol:uusd": "SOLUSD", "uada:uusd": "ADAUSD", "ubtc:unusd": "BTCUSD", "ueth:unusd": "ETHUSD", "uusdt:unusd": "USDTUSD", "uusdc:unusd": "USDCUSD", "uatom:unusd": "ATOMUSD", "ubnb:unusd": "BNBUSD", "uavax:unusd": "AVAXUSD", "usol:unusd": "SOLUSD", "uada:unusd": "ADAUSD" } }'
export FEEDER_MNEMONIC="car apple deep ....."
export VALIDATOR_ADDRESS="nibi1valoper..."
```

Aşağıdaki kodda herhangi bir değişiklik yapmıyoruz.
```
sudo tee /etc/systemd/system/pricefeeder.service<<EOF
[Unit]
Description=Nibiru Pricefeeder
Requires=network-online.target
After=network-online.target

[Service]
Type=exec
User=<your user>
Group=<your group>
ExecStart=/usr/local/bin/pricefeeder
Restart=on-failure
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGTERM
PermissionsStartOnly=true
LimitNOFILE=65535
Environment=CHAIN_ID=$CHAIN_ID'
Environment=GRPC_ENDPOINT='$GRPC_ENDPOINT'
Environment=WEBSOCKET_ENDPOINT='$WEBSOCKET_ENDPOINT'
Environment=EXCHANGE_SYMBOLS_MAP='$EXCHANGE_SYMBOLS_MAP'
Environment=FEEDER_MNEMONIC='$FEEDER_MNEMONIC'

[Install]
WantedBy=multi-user.target
EOF
```

### Pricefeeder'ı Başlatma
```
sudo systemctl daemon-reload && \
sudo systemctl enable pricefeeder && \
sudo systemctl start pricefeeder
```

### Pricefeeder Loglarına Bakma
```
journalctl -fu pricefeeder
```

## YARARLI KOMUTLAR

## Logları Kontrol Etme 
```
journalctl -fu nibid -o cat
```

### Sistemi Başlatma

```
systemctl start nibid
```

### Sistemi Durdurma
```
systemctl stop nibid
```

### Sistemi Yeniden Başlatma
```
systemctl restart nibid
```

### Node Senkronizasyon Durumu
```
nibid status 2>&1 | jq .SyncInfo
```
```
curl -s localhost:26657/status | jq .result.sync_info
```

### Validator Bilgileri
```
nibid status 2>&1 | jq .ValidatorInfo
```

### Node Bilgileri

```
nibid status 2>&1 | jq .NodeInfo
```

### Node ID Öğrenme

```
nibid tendermint show-node-id
```


### Node IP Adresini Öğrenme

```
curl icanhazip.com
```

### Cüzdanların Listesine Bakma

```
nibid keys list
```

### Cüzdan Adresini Görme

```
nibid keys show $NIB_WALLET --bech val -a
```

### Cüzdanı İçeri Aktarma

```
nibid keys add $NIB_WALLET --recover
```

### Cüzdanı Silme

```
nibid keys delete $NIB_WALLET
```

### Cüzdan Bakiyesine Bakma

```
nibid query bank balances $NIB_WALLET_ADDRESS
```

### Bir Cüzdandan Diğer Bir Cüzdana Transfer Yapma

```
nibid tx bank send $NIB_WALLET_ADDRESS GONDERILECEK_CUZDAN_ADRESI 100000000unibi
```

### Proposal Oylamasına Katılma
```
nibid tx gov vote 1 yes --from $NIB_WALLET --chain-id=$NIB_CHAIN_ID
```


### Validatore Stake Etme / Delegate Etme

```
nibid tx staking delegate $VALOPER_ADDRESS 100000000unibi --from=$NIB_WALLET --chain-id=$NIB_CHAIN_ID --gas-prices=0.1unibi --gas-adjustment=1.5 --gas=auto 
```



### Mevcut Validatorden Diğer Validatore Stake Etme / Redelegate Etme
<srcValidatorAddress>: Mevcut Stake edilen validatorün adresi
<destValidatorAddress>: Yeni stake edilecek validatorün adresi 
```
nibid tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 100000000unibi --from=$NIB_WALLET --chain-id=$NIB_CHAIN_ID --gas-prices=0.1unibi --gas-adjustment=1.5 --gas=auto
```

### Ödülleri Çekme

```
nibid tx distribution withdraw-all-rewards --from=$NIB_WALLET --chain-id=$NIB_CHAIN_ID --gas=auto
```

### Komisyon Ödüllerini Çekme

```
nibid tx distribution withdraw-rewards $VALOPER_ADDRESS --from=$NIB_WALLET --commission --chain-id=$NIB_CHAIN_ID
```

### Validator İsmini Değiştirme
NEWNODENAME yazan yere yeni validator/moniker isminizi yazınız. TR karakçer içermemelidir.

```
nibid tx staking edit-validator \
--moniker=NEWNODENAME \
--chain-id=$NIB_CHAIN_ID \
--from=$NIB_WALLET
```

### Validator Komisyon Oranını Degiştirme
```
nibid tx staking edit-validator --commission-rate "0.02" --moniker=$NIB_NODENAME --chain-id=$NIB_CHAIN_ID --from=$NIB_WALLET
```

### Validator Bilgilerinizi Düzenleme
Bu bilgileri değiştirmeden önce https://keybase.io/ adresine kayıt olarak aşağıdaki kodda görüldüğü gibi 16 haneli (XXXX0000XXXX0000) kodunuzu almalısınız. Ayrıca profil resmi vs. ayarları da yapabilirsiniz. 
$NODENAME: Yeni node adınız (moniker)
$NIB_WALLET: Cüzdan adınız, değiştirmeniz gerekmez. Değişkenlere ekledik çünkü.
```
nibid tx staking edit-validator \
--moniker=$NODENAME \
--identity=XXXX0000XXXX0000 \
--website="VARSA WEBSITENIZI YAZABILIRSINIZ" \
--details="BU BOLUME KENDINIZI TANITAN BIR CUMLE YAZABILIRSINIZ" \
--chain-id=$NIB_CHAIN_ID \
--from=$NIB_WALLET
```

### Validatoru Jail Durumundan Kurtarma 

```
nibid tx slashing unjail \
  --broadcast-mode=block \
  --from=$NIB_WALLET \
  --chain-id=$NIB_CHAIN_ID \
  --gas=auto
  --gas-adjustment=1.4
```

### Node'u Tamamen Silme 

```
systemctl stop nibid && \
systemctl disable nibid && \
rm /etc/systemd/system/nibid.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .nibid nibiru && \
rm -rf $(which nibid) \
sed -i '/NIB_/d' ~/.bash_profile
```


# Hesaplar:

[Anatolian Team](https://anatolianteam.com)

[Twitter](https://twitter.commehmetkoltigin)

[Medium](https://medium.com/@mehmetkoltigin)

[YouTube](https://www.youtube.com/channel/UCmLgaftx5e38BE0E7gpY2dA)

[Discord](https://discordapp.com/users/837933958280904737)

[Telegram](https://t.me/mehmetkoltigin)
