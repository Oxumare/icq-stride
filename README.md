# Stride için GO V2 Relayer Kullanarak Zincirler arası sorgu İletme

Burada bazı açıklamalarda bulunayım.Stride nodenizde v2 relayer yüklü olmalıdır.Ayrıca ICQ paketi için relayerde kullandığınız cüzdanlarınızı kullanmak zorundasınız.

İşlemlere başlamadan önce aşağıdaki komutları uygulayarak RPC-GRPC adreslerimizi alıp bir kenara not edelim.

Stride node için ;
RPC adresimizi alalım:
```
nano $HOME/.stride/config/config.toml
```
GRPC adresimizi alalım:
```
nano $HOME/.stride/config/app.toml
```

Gaia node için ;
RPC adresimizi alalım:
```
echo "$(curl -s ifconfig.me)$(grep -A 3 "\[rpc\]" ~/.gaia/config/config.toml | egrep -o ":[0-9]+")"
```
GRPC adresimizi alalım:
echo "$(curl -s ifconfig.me)$(grep -A 6 "\[grpc\]" ~/.gaia/config/app.toml | egrep -o ":[0-9]+")"


## 1. Binary dosyasını indirelim.
```
cd $HOME
git clone https://github.com/Stride-Labs/interchain-queries.git
cd interchain-queries
go build
sudo mv interchain-queries /usr/local/bin/icq
```

## 2. ICQ için ana sayfa dizini ve yapılandırma dosyası oluşturalım.
```
cd $HOME && mkdir .icq
sudo tee $HOME/.icq/config.yaml > /dev/null <<EOF
default_chain: stride-testnet
chains:
  gaia-testnet:
    key: wallet
    chain-id: GAIA
    rpc-addr: http://127.0.0.1:23657      # GAI RPC adresimizi yazalım
    grpc-addr: http://127.0.0.1:23090     # GAIA GRPC adresimizi yazalım.
    account-prefix: cosmos
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001uatom
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
  stride-testnet:
    key: wallet
    chain-id: STRIDE-TESTNET-4
    rpc-addr: http://127.0.0.1:16657      # Stride RPC adresimizi yazalım.
    grpc-addr: http://127.0.0.1:16090     # Stride GRPC adresimizi yazalım.
    account-prefix: stride
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001ustrd
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
cl: {}
EOF
```

## 3. Cüzdanlarımzı ekleyelim.
```
icq keys restore --chain stride-testnet <walletadınız>
icq keys restore --chain gaia-testnet <walletadınız>
```

## 4. ICQ servis dosyası oluşturalım.
```
sudo tee /etc/systemd/system/icqd.service > /dev/null <<EOF
[Unit]
Description=Interchain Query Service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which icq) run --debug
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## 5. ICQ servisimizi başlatalım.
```
sudo systemctl daemon-reload
sudo systemctl enable icqd
sudo systemctl restart icqd
```

## 6. ICQ loglarımzı kontrol edelim
```
journalctl -u icqd -f -o cat
```

Aşağıdaki görseldeki çıkyı almanız uzun sürebilir.Yaklaşık 30 dk:

![1](https://user-images.githubusercontent.com/43583832/185791324-6d4367c3-f010-41a0-915e-6e470c16e2cc.png)

Bundan sonra işlemimizi stride explorerde görebiliriz.

![2](https://user-images.githubusercontent.com/43583832/185791458-85bf67f6-7af7-4602-9bb8-90a78c3c33cd.png)


![3](https://user-images.githubusercontent.com/43583832/185791569-e8e2a9d1-c048-416f-853c-9034f3ed8e90.png)

