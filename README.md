# Babylon Kurulum Rehberi

![Babylon-GitHub](https://user-images.githubusercontent.com/102043225/227338716-6b84f35a-d80d-40b5-a00e-9146353e64f1.jpg)

## Bağlantılar
 ✔️ [Website](https://www.babylonchain.io/)<br>
 ✔️ [Blockchain Explorer](https://babylon.explorers.guru/)<br>
 ✔️ [Doküman](https://docs.babylonchain.io/)<br>
 ✔️ [GitHub](https://github.com/babylonchain)<br>
 ✔️ [Discord](https://discord.gg/XcCtr53g8F)<br> 
 ✔️ [Zealy Görevleri](https://zealy.io/c/babylonchain/invite/H74AmwvpVNPGL8suCk1xL)

## Gereksinimler 
| Bileşenler | Minimum Gereksinimler | **Tavsiye Edilen Gereksinimler** | 
| ------------ | ------------ | ------------ |
| CPU |	4 | 4 |
| RAM	| 8 GB | 16 GB |
| Storage	| 250 GB SSD | 500 GB SSD |

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
* `$BBN_NODENAME` validator adınız
* `$BBN_WALLET` cüzdan adınız
*  Eğer portu başka bir node kullanıyorsa aşağıdan değiştirebilirsiniz.
```shell
echo "export BBN_NODENAME=$BBN_NODENAME"  >> $HOME/.bash_profile
echo "export BBN_WALLET=$BBN_WALLET" >> $HOME/.bash_profile
echo "export BBN_PORT=11" >> $HOME/.bash_profile
echo "export BBN_CHAIN_ID=bbn-test-2" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Örnek
Node ve Cüzdan adımızın `Mehmet` olduğunu varsayalım. Kod aşağıdaki şekilde düzenlenecektir. 
```shell
echo "export BBN_NODENAME=Mehmet"  >> $HOME/.bash_profile
echo "export BBN_WALLET=Mehmet" >> $HOME/.bash_profile
echo "export BBN_PORT=11" >> $HOME/.bash_profile
echo "export BBN_CHAIN_ID=bbn-test-2" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Babylon'un Kurulması
```shell
git clone https://github.com/babylonchain/babylon
cd babylon || return
git checkout v0.7.2
make install
babylond version
```
Versiyon çıktısı `v0.7.2` olacak.

## Uygulamayı Yapılandırma ve Başlatma
Aşağıdaki kodlarda herhangi bir değişilik yapmadan kopyalayıp yapıştırıyoruz.
```shell
babylond config keyring-backend test
babylond config chain-id $BBN_CHAIN_ID
babylond init --chain-id $BBN_CHAIN_ID $BBN_NODENAME

# Genesis ve Addrbook Dosyasının Kopyalanması
curl -L https://github.com/babylonchain/networks/raw/main/bbn-test-2/genesis.tar.bz2 > genesis.tar.bz2
tar -xjf genesis.tar.bz2
rm genesis.tar.bz2
mv genesis.json ~/.babylond/config/genesis.json
curl -s https://raw.githubusercontent.com/koltigin/Babylon-Kurulum-Rehberi/main/addrbook.json > $HOME/.babylond/config/addrbook.json

# Minimum GAS Ücretinin Ayarlanması
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.00001ubbn"|g' $HOME/.babylond/config/app.toml

# Indexer'i Kapatma (Opsiyonel)
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.babylond/config/config.toml

# SEED ve PEERS Ayarlanması
SEEDS="8da45f9ff83b4f8dd45bbcb4f850999637fbfe3b@seed0.testnet.babylonchain.io:26656,4b1f8a774220ba1073a4e9f4881de218b8a49c99@seed1.testnet.babylonchain.io:26656"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.babylond/config/config.toml

# Prometheus'u Aktif Etme
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.babylond/config/config.toml

# Pruning'i Ayarlama
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.babylond/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.babylond/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.babylond/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.babylond/config/app.toml

# app.toml ve config.toml Düzenleme
sed -i -e "s|^key-name *=.*|key-name = \"$BBN_WALLET\"|" $HOME/.babylond/config/app.toml
sed -i -e "s|^timeout_commit *=.*|timeout_commit = \"10s\"|" $HOME/.babylond/config/config.toml

# Portları Ayarlama
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${BBN_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${BBN_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${BBN_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${BBN_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${BBN_PORT}660\"%; s%^timeout_commit = \"5s\"%timeout_commit = \"10s\"%" $HOME/.babylond/config/config.toml

sed -i.bak -e "s%^address = \"tcp://localhost:1317\"%address = \"tcp://localhost:${BBN_PORT}317\"%; s%^address = \":8080\"%address = \":${BBN_PORT}080\"%; s%^address = \"localhost:9090\"%address = \"localhost:${BBN_PORT}090\"%; s%^address = \"localhost:9091\"%address = \"localhost:${BBN_PORT}091\"%" $HOME/.babylond/config/app.toml

sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${BBN_PORT}657\"%" $HOME/.babylond/config/client.toml


# Servis Dosyası Oluşturma

sudo tee /etc/systemd/system/babylond.service > /dev/null << EOF
[Unit]
Description=Babylon Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which babylond) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```

## Snapshot Yükleme (NodeJumper)
```shell
SNAP_NAME=$(curl -s https://snapshots1-testnet.nodejumper.io/babylon-testnet/info.json | jq -r .fileName)
curl "https://snapshots1-testnet.nodejumper.io/babylon-testnet/${SNAP_NAME}" | lz4 -dc - | tar -xf - -C "$HOME/.babylond"
```

## Servisi Başlatma
```shell
sudo systemctl daemon-reload
sudo systemctl enable babylond
sudo systemctl start babylond
```

## Logları Kontrol Etme
```shell
journalctl -u babylond -f -o cat
```  

## Cüzdan Oluşturma

### Yeni Cüzdan Oluşturma
`$BBN_WALLET` bölümünü değiştirmiyoruz kurulumun başında cüzdanımıza değişkenler ile isim belirledik.
```shell 
babylond keys add $BBN_WALLET
```  

### Var Olan Cüzdanı İçeri Aktarma
```shell
babylond keys add $BBN_WALLET --recover
```

## Cüzdan ve Valoper Bilgileri
Burada cüzdan ve valoper bilgilerimizi değişkene ekliyoruz.

```shell
BBN_WALLET_ADDRESS=$(babylond keys show $BBN_WALLET -a)
BBN_VALOPER_ADDRESS=$(babylond keys show $BBN_WALLET --bech val -a)
```

```shell
echo 'export BBN_WALLET_ADDRESS='${BBN_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export BBN_VALOPER_ADDRESS='${BBN_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Faucet
[Babylon Discord](https://discord.gg/XcCtr53g8F) adresine giderek `#【🌍】faucet ` kanalından `!faucet bbn-cuzdan-adresi` şeklinde mesaj atarak token istiyoruz. 

### Cüzdan Bakiyesine Bakma
```
babylond query bank balances $BBN_WALLET_ADDRESS
```

🔴 **BU AŞAMADAN SONRA NODE'UMUZUN EŞLEŞMESİNİ BEKLİYORUZ.**

## Senkronizasyonu Kontrol Etme
`false` çıktısı almadıkça bir sonraki yani validator oluşturma adımına geçmiyoruz.
```shell
babylond status 2>&1 | jq .SyncInfo
```

🔴 **Eşleşme tamamlandıysa aşağıdaki adıma geçiyoruz.**

## BLS Key Oluşturma ve Yeniden Başlatma

```shell
babylond create-bls-key $(babylond keys show $BBN_WALLET -a)

sudo systemctl restart babylond
```

## Validator Oluşturma
 Aşağıdaki komutta aşağıda berlirttiğim yerler dışında bir değişiklik yapmanız gerekmez;
   - `identity`  burada `XXXX1111XXXX1111` yazan yere [keybase](https://keybase.io/) sitesine üye olarak size verilen kimlik numaranızı yazıyorsunuz.
   - `details` `Always forward with the Anatolian Team 🚀` yazan yere kendiniz hakkında bilgiler yazabilirsiniz.
   - `website`  `https://anatolianteam.com` yazan yere varsa bir siteniz ya da twitter vb. adresinizi yazabilirsiniz.
   - `security-contact`  E-posta adresiniz.
 ```shell 
babylond tx checkpointing create-validator \
--amount=1000000ubbn \
--pubkey=$(babylond tendermint show-validator) \
--moniker=$BBN_NODENAME \
--chain-id=$BBN_CHAIN_ID \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.05 \
--min-self-delegation="1" \
--gas-prices=0.1ubbn \
--gas-adjustment=1.5 \
--gas=auto \
--from=$BBN_WALLET \
--details="Always forward with the Anatolian Team 🚀" \
--security-contact="xxxxxxx@gmail.com" \
--website="https://anatolianteam.com" \
--identity="XXXX1111XXXX1111" \
--yes
```

## YARARLI KOMUTLAR

## Logları Kontrol Etme 
```
journalctl -fu babylond -o cat
```

### Sistemi Başlatma

```
systemctl start babylond
```

### Sistemi Durdurma
```
systemctl stop babylond
```

### Sistemi Yeniden Başlatma
```
systemctl restart babylond
```

### Node Senkronizasyon Durumu
```
babylond status 2>&1 | jq .SyncInfo
```
```
curl -s localhost:26657/status | jq .result.sync_info
```

### Validator Bilgileri
```
babylond status 2>&1 | jq .ValidatorInfo
```

### Node Bilgileri

```
babylond status 2>&1 | jq .NodeInfo
```

### Node ID Öğrenme

```
babylond tendermint show-node-id
```


### Node IP Adresini Öğrenme

```
curl icanhazip.com
```

### Cüzdanların Listesine Bakma

```
babylond keys list
```

### Cüzdan Adresini Görme

```
babylond keys show $BBN_WALLET --bech val -a
```

### Cüzdanı İçeri Aktarma

```
babylond keys add $BBN_WALLET --recover
```

### Cüzdanı Silme

```
babylond keys delete $BBN_WALLET
```

### Cüzdan Bakiyesine Bakma

```
babylond query bank balances $BBN_WALLET_ADDRESS
```

### Bir Cüzdandan Diğer Bir Cüzdana Transfer Yapma

```
babylond tx bank send $BBN_WALLET_ADDRESS GONDERILECEK_CUZDAN_ADRESI 100000000uheart
```

### Proposal Oylamasına Katılma
```
babylond tx gov vote 1 yes --from $BBN_WALLET --chain-id=$BBN_CHAIN_ID
```


### Validatore Stake Etme / Delegate Etme

```
babylond tx epoching delegate $BBN_VALOPER_ADDRESS 1000000ubbn --from=$BBN_WALLET --chain-id=$BBN_CHAIN_ID --gas-prices=0.1ubbn --gas-adjustment=1.5 --gas=auto
```

### Mevcut Validatorden Diğer Validatore Stake Etme / Redelegate Etme
<srcValidatorAddress>: Mevcut Stake edilen validatorün adresi
<destValidatorAddress>: Yeni stake edilecek validatorün adresi 
```
babylond tx epoching redelegate <srcValidatorAddress> <destValidatorAddress> 1000000ubbn --from=$BBN_WALLET --chain-id=$BBN_CHAIN_ID --gas-prices=0.1ubbn --gas-adjustment=1.5 --gas=auto
```

### Ödülleri Çekme

```
babylond tx distribution withdraw-all-rewards --from=$BBN_WALLET --chain-id=$BBN_CHAIN_ID --gas=auto
```

### Komisyon Ödüllerini Çekme

```
babylond tx distribution withdraw-rewards $VALOPER_ADDRESS --from=$BBN_WALLET --commission --chain-id=$BBN_CHAIN_ID
```

### Validator İsmini Değiştirme
NEWNODENAME yazan yere yeni validator/moniker isminizi yazınız. TR karakçer içermemelidir.

```
babylond tx staking edit-validator \
--moniker=NEWNODENAME \
--chain-id=$BBN_CHAIN_ID \
--from=$BBN_WALLET
```

### Validator Komisyon Oranını Degiştirme
```
babylond tx staking edit-validator --commission-rate "0.02" --moniker=$BBN_NODENAME --chain-id=$BBN_CHAIN_ID --from=$BBN_WALLET
```

### Validator Bilgilerinizi Düzenleme
Bu bilgileri değiştirmeden önce https://keybase.io/ adresine kayıt olarak aşağıdaki kodda görüldüğü gibi 16 haneli (XXXX0000XXXX0000) kodunuzu almalısınız. Ayrıca profil resmi vs. ayarları da yapabilirsiniz. 
$NODENAME: Yeni node adınız (moniker)
$BBN_WALLET: Cüzdan adınız, değiştirmeniz gerekmez. Değişkenlere ekledik çünkü.
```
babylond tx staking edit-validator \
--moniker=$NODENAME \
--identity=XXXX0000XXXX0000 \
--website="VARSA WEBSITENIZI YAZABILIRSINIZ" \
--details="BU BOLUME KENDINIZI TANITAN BIR CUMLE YAZABILIRSINIZ" \
--chain-id=$BBN_CHAIN_ID \
--from=$BBN_WALLET
```

### Validatoru Jail Durumundan Kurtarma 

```
babylond tx slashing unjail --from $BBN_WALLET --chain-id $BBN_CHAIN_ID --gas-prices 0.00001ubbn --gas-adjustment 1.5 --gas auto -y
```

### Node'u Tamamen Silme 

```
systemctl stop babylond && \
systemctl disable babylond && \
rm /etc/systemd/system/babylond.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .babylond babylon && \
rm -rf $(which babylond)
sed -i '/BBN_/d' ~/.bash_profile
```


# Hesaplar:

[Anatolian Team](https://anatolianteam.com)

[Twitter](https://twitter.commehmetkoltigin)

[Medium](https://medium.com/@mehmetkoltigin)

[YouTube](https://www.youtube.com/channel/UCmLgaftx5e38BE0E7gpY2dA)

[Discord](https://discordapp.com/users/837933958280904737)

[Telegram](https://t.me/mehmetkoltigin)
