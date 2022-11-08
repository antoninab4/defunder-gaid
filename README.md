# defunder-gaid
Установка ноды DeFunder с нуля. Пошаговое руководство.

# DeFund Testnet УСТАНОВКА (ЧИСТАЯ)

![defund](https://user-images.githubusercontent.com/44331529/198265844-d3d66bd2-78cf-4c14-8320-12a00f2aafe0.png)

[WEBSITE](https://defund.app/) \
[GitHub](https://github.com/defund-labs/testnet)
=
[EXPLORER](https://defund.explorers.guru/validators)
=

- **Требования**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Tetsnet   |   4|  8GB | 250GB    |


# 1) Автоустановка script (после него останется должаться синхрона и создать валидатора) Используйте навигацию в скрипте.
```bash
wget -O dfn https://raw.githubusercontent.com/obajay/nodes-Guides/main/DeFund/dfn && chmod +x dfn && ./dfn
```

# 2) Manual installation (если вы используете установку скриптом пропускайте 2 шаг и продолжайте с 3 шага - создание валидатора)



```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.19

```bash
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 03.11.22
```bash
git clone https://github.com/defund-labs/defund
cd defund
git checkout v0.1.0
make install
```
`defundd version`
- version: v0.1.0

```bash
defundd init STAVRguide --chain-id defund-private-3

```    

## Создать/восстановить wallet (или или)
```bash
defundd keys add <walletname>
```
```
defundd keys add <walletname> --recover
```

## Загрузить  Genesis
```bash
wget -O $HOME/.defund/config/genesis.json "https://raw.githubusercontent.com/defund-labs/testnet/main/defund-private-3/genesis.json"
```
`sha256sum $HOME/.defund/config/genesis.json`
+ bec32034b1ca130e2f45c603f42103490df990984fb46528994b4a99a5f77ea6

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ufetf\"/;" ~/.defund/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.defund/config/config.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.defund/config/config.toml
peers="dff3a67755a5832198447224196654374c7ef95d@65.21.170.3:40656,daff7b8cbcae4902c3c4542113ba521f968cc3f8@213.239.217.52:29656,445425e51dc42603cfeac805816bcdda2fb8a6a1@65.109.54.110:26631,f2985029a48319330b99767d676412383e7061bf@194.163.155.84:36656,75cccc67bc20e7e5429b80c4255ffe44ef24bc26@65.109.85.170:33656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.defund/config/config.toml
seeds="85279852bd306c385402185e0125dffeed36bf22@38.146.3.194:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.defund/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.defund/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.defund/config/config.toml

```
### Pruning (по желанию)
```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.defund/config/app.toml
```
### Indexer (по желанию) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.defund/config/config.toml
```

## Загрузить ADDBOOK
```bash
wget -O $HOME/.defund/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/DeFund/addrbook.json"
```

## StateSync
```bash
SNAP_RPC=https://t-defund.rpc.utsa.tech:443
peers="https://t-defund.rpc.utsa.tech:443"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.defund/config/config.toml
defundd tendermint unsafe-reset-all --home $HOME/.defund
systemctl restart defundd && journalctl -u defundd -f -o cat
```

# Создать сервисный файл
```bash
sudo tee /etc/systemd/system/defundd.service > /dev/null <<EOF
[Unit]
Description=defund
After=network-online.target

[Service]
User=$USER
ExecStart=$(which defundd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Запуск
```bash
sudo systemctl daemon-reload
sudo systemctl enable defundd
sudo systemctl restart defundd && sudo journalctl -u defundd -f -o cat
```
# 3)
### Создание валидатора
```bash
defundd tx staking create-validator \
  --amount 1000000ufetf \
  --from <имякошелька> \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(defundd tendermint show-validator) \
  --moniker <имяноды> \
  --chain-id defund-private-3 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Удалить ноду
```bash
sudo systemctl stop defundd && \
sudo systemctl disable defundd && \
rm /etc/systemd/system/defundd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf defund && \
rm -rf .defundd && \
rm -rf $(which defundd)
```

#
### Sync Info
```bash
defundd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```bash
defundd status 2>&1 | jq .NodeInfo
```
### Check node logs
```bash
journalctl -u defundd -f -o cat
```
### Check Balance
```bash
defundd query bank balances defund...addressdefund1yjgn7z09ua9vms259j
```

### проверить блоки
```
defundd status 2>&1 | jq ."SyncInfo"."latest_block_height"
```
### проверить логи
```
journalctl -u defundd -f -o cat
```
```
journalctl -u defundd -f -o cat | grep -v "p2p"
```
### проверить статус
```
curl localhost:26657/status
```
### проверить баланс
```
defundd q bank balances <address>
```
### проверить pubkey валидатора
defundd tendermint show-validator

### проверить валидатора
```
defundd query staking validator <адресвалидатора>
```
```
defundd query staking validators --limit 1000000 -o json | jq '.validators[] | select(.description.moniker=="<name_moniker>")' | jq
```
### проверка информации по TX_HASH
```
defundd query tx <TX_HASH>
```
### параметры сети
```
defundd q staking params
```
```
defundd q slashing params
```
### проверить сколько блоков пропущено валидатором и с какого блока актив
```
defundd q slashing signing-info $(defundd tendermint show-validator)
```
### узнать транзакцию создания валидатора (заменить свой valoper_address)
```
defundd query txs --events='create_validator.validator=<your_valoper_address>' -o=json | jq .txs[0].txhash -r
```
### просмотр активного сета
```
defundd q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```
### просмотр неактивного сета
```
defundd q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

## Транзакции

### собрать реварды со всех валидаторов, которым делегировали (без комиссии)
```
defundd tx distribution withdraw-all-rewards --from <имякошелька> --fees 5000ufetf -y
```
### собрать реварды c отдельного валидатора или реварды + комиссию со своего валидатора
```
defundd tx distribution withdraw-rewards <адресвалидатора> --from <тмякошелька> --fees 5000ufetf --commission -y --chain-id defund-private-3
```
### заделегировать себе в стейк еще (так отправляется 1 монетa)
```
defundd tx staking delegate <адрес валидатора> 1000000ufetf --from <имя кошелька> --fees 5000ufetf -y --chain-id defund-private-3
```
### ределегирование на другого валидатора
```
defundd tx staking redelegate <откуда-validator-addr> <куда-validator-addr> 1000000ufetf --from <name_wallet> --fees 5000ufetf -y --chain-id defund-private-3
```

### Unbond
```
defundd tx staking unbond <addr_valoper> 1000000ufetf --from <name_wallet> --fees 5000ufetf -y --chain-id defund-private-3
```
### отправить монеты на другой адрес
```
defundd tx bank send <name_wallet> <address> 1000000ufetf --fees 5000ufetf -y --chain-id defund-private-3
```

### выбраться из тюрьмы
```
defundd tx slashing unjail --from <name_wallet> --fees 5000ufetf -y --chain-id defund-private-3
```
