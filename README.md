# Babylon Kurulum Rehberi

[Kaynak](https://docs.babylonchain.io/) / [Explorer](https://babylon.explorers.guru/)
![Babylon-GitHub](https://user-images.githubusercontent.com/102043225/227338716-6b84f35a-d80d-40b5-a00e-9146353e64f1.jpg)

## Bağlantılar
 ✔️ [Website](https://www.quasar.fi/)<br>
 ✔️ [Blockchain Explorer](https://babylon.explorers.guru/)<br>
 ✔️ [Doküman](https://docs.babylonchain.io/)<br>
 ✔️ [Discord](https://discord.gg/babylonchain)<br>

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
* `$BBN_NODENAME` validator adınız
* `$BBN_WALLET` cüzdan adınız
*  Eğer portu başka bir node kullanıyorsa aşağıdan değiştirebilirsiniz.
```shell
echo "export BBN_NODENAME=$BBN_NODENAME"  >> $HOME/.bash_profile
echo "export BBN_WALLET=$BBN_WALLET" >> $HOME/.bash_profile
echo "export BBN_PORT=11" >> $HOME/.bash_profile
echo "export BBN_CHAIN_ID=qsr-questnet-04" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Örnek
Node ve Cüzdan adımızın `Mehmet` olduğunu varsayalım. Kod aşağıdaki şekilde düzenlenecektir. 
```shell
echo "export BBN_NODENAME=Mehmet"  >> $HOME/.bash_profile
echo "export BBN_WALLET=Mehmet" >> $HOME/.bash_profile
echo "export BBN_PORT=11" >> $HOME/.bash_profile
echo "export BBN_CHAIN_ID=qsr-questnet-04" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Babylon'un Kurulması

```shell
git clone https://github.com/babylonchain/babylon
cd babylon || return
git checkout v0.5.0
make install
babylond version
```
Versiyon çıktısı `0.0.2-alpha-11` olacak.

## Uygulamayı Yapılandırma ve Başlatma
```shell
babylond config keyring-backend test
babylond config chain-id $BBN_CHAIN_ID
babylond init --chain-id $BBN_CHAIN_ID $BBN_NODENAME
```

## Genesis ve Addrbook Dosyasının Kopyalanması
```shell
curl -L https://github.com/babylonchain/networks/blob/main/bbn-test1/genesis.tar.bz2?raw=true > genesis.tar.bz2
tar -xjf genesis.tar.bz2
rm -rf genesis.tar.bz2
mv genesis.json ~/.babylond/config/genesis.json
curl -s https://snapshots1-testnet.nodejumper.io/babylon-testnet/addrbook.json > $HOME/.babylond/config/addrbook.json
```

## Minimum GAS Ücretinin Ayarlanması
```shell
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0ubbn"|g' $HOME/.babylond/config/app.toml
```

## Indexer'i Kapatma (Opsiyonel)
```shell
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.babylond/config/config.toml
```

## SEED ve PEERS Ayarlanması
```shell
SEEDS=""
PEERS="6ccfdbe91c06698f0a66cf95a249dbcd88b5aaa4@quasar-testnet-peer.itrocket.net:443,bba6e85e3d1f1d9c127324e71a982ddd86af9a99@88.99.3.158:18256,bcb8d2b5d5464cddbab9ce2705aae3ad3e38aeac@144.76.67.53:2490,1c1043ae487c91209fce8c589a5772a7f3846e7c@136.243.88.91:8080,1112acc7479a8a1afb0777b0b9a39fb1f7e77abd@34.175.69.87:26656,bffb10a5619be7bfa98919e08f4a6bef4f8f6bf0@135.181.210.186:26656,695b9dc49a979e4c5986c5ae9a6effc8bc8597f0@185.197.250.151:27656,8937bdacf1f0c8b2d1ffb4606554eaf08bd55df4@5.75.255.107:26656,a23f002bda10cb90fa441a9f2435802b35164441@38.146.3.203:18256,41ee7632f310c035235828ce03c208dbe1e24d7d@38.146.3.204:18256,966acc999443bae0857604a9fce426b5e09a7409@65.108.105.48:18256"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.babylond/config/config.toml
```

## Prometheus'u Aktif Etme
```shell
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.babylond/config/config.toml
```

## Pruning'i Ayarlama
```shell
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.babylond/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.babylond/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.babylond/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.babylond/config/app.toml
```

## Portları Ayarlama
```shell
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${BBN_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${BBN_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${BBN_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${BBN_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${BBN_PORT}660\"%" $HOME/.babylond/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${BBN_PORT}317\"%; s%^address = \":8080\"%address = \":${BBN_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${BBN_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${BBN_PORT}091\"%" $HOME/.babylond/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${BBN_PORT}657\"%" $HOME/.babylond/config/client.toml
```

## Servis Dosyası Oluşturma
```shell
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
SNAP_NAME=$(curl -s https://snapshots2-testnet.nodejumper.io/quasar-testnet/info.json | jq -r .fileName)
curl "https://snapshots2-testnet.nodejumper.io/quasar-testnet/${SNAP_NAME}" | lz4 -dc - | tar -xf - -C "$HOME/.babylond"
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
[Babylon Discord](https://discord.gg/babylonchain) adresine giderek `#【🌍】faucet ` kanalından `!faucet bbn-cuzdan-adresi` şeklinde mesaj atarak token istiyoruz. 


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
babylond tx staking create-validator \
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
babylond tx staking delegate $BBN_VALOPER_ADDRESS 1000000ubbn --from=$BBN_WALLET --chain-id=$BBN_CHAIN_ID --gas-prices=0.1ubbn --gas-adjustment=1.5 --gas=auto
```

### Mevcut Validatorden Diğer Validatore Stake Etme / Redelegate Etme
<srcValidatorAddress>: Mevcut Stake edilen validatorün adresi
<destValidatorAddress>: Yeni stake edilecek validatorün adresi 
```
babylond tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 1000000ubbn --from=$BBN_WALLET --chain-id=$BBN_CHAIN_ID --gas-prices=0.1ubbn --gas-adjustment=1.5 --gas=auto
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
babylond tx slashing unjail \
  --broadcast-mode=block \
  --from=$BBN_WALLET \
  --chain-id=$BBN_CHAIN_ID \
  --gas=auto
  --gas-adjustment=1.4
```

### Node'u Tamamen Silme 

```
systemctl stop babylond && \
systemctl disable babylond && \
rm /etc/systemd/system/babylond.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .babylond quasar && \
rm -rf $(which babylond) \
sed -i '/BBN_/d' ~/.bash_profile
```


# Hesaplar:

[Anatolian Team](https://anatolianteam.com)

[Twitter](https://twitter.commehmetkoltigin)

[Medium](https://medium.com/@mehmetkoltigin)

[YouTube](https://www.youtube.com/channel/UCmLgaftx5e38BE0E7gpY2dA)

[Discord](https://discordapp.com/users/837933958280904737)

[Telegram](https://t.me/mehmetkoltigin)
