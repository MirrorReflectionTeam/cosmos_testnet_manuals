## KEY

#### Add new key

```
babylond keys add wallet
```

#### Recover Existing Key

```
babylond keys add wallet --recover
```

#### Query Wallet Balance

```
babylond q bank balances $(babylond keys show wallet -a)
```

## Validator


#### Create new validator
```
babylond tx staking create-validator \
--amount=1000000ubbn \
--pubkey=$(babylond tendermint show-validator) \
--moniker="YOUR_MONIKER_NAME" \
--chain-id=bbn-test1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.1ubbn \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

#### Edit existing validator

```
babylond tx staking edit-validator \
--moniker="YOUR_MONIKER_NAME" \
--identity="YOUR_KEYBASE_ID" \
--details="YOUR_DETAILS" \
--chain-id=bbn-test1 \
--commission-rate=0.1 \
--from=wallet \
--gas-prices=0.1ubbn \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

#### Unjail validator

```
babylond tx slashing unjail --from wallet --chain-id bbn-test1 --gas-prices 0.1ubbn --gas-adjustment 1.5 --gas auto -y 
```

#### View validator details

```
babylond q staking validator $(babylond keys show wallet --bech val -a) 
```

## Tokens

#### Withdraw commission and rewards from your validator

```
babylond tx distribution withdraw-rewards $(babylond keys show wallet --bech val -a) --commission --from wallet --chain-id bbn-test1 --gas-prices 0.1ubbn --gas-adjustment 1.5 --gas auto -y 
```

#### Delegate tokens to yourself

```
babylond tx staking delegate $(babylond keys show wallet --bech val -a) 1000000ubbn --from wallet --chain-id bbn-test1 --gas-prices 0.1ubbn --gas-adjustment 1.5 --gas auto -y 
```

#### Delegate tokens to validator

```
babylond tx staking delegate YOUR_TO_VALOPER_ADDRESS 1000000ubbn --from wallet --chain-id bbn-test1 --gas-prices 0.1ubbn --gas-adjustment 1.5 --gas auto -y 
```

#### Redelegate tokens to validator

```
babylond tx staking redelegate $(babylond keys show wallet --bech val -a) YOUR_TO_VALOPER_ADDRESS 1000000ubbn --from wallet --chain-id bbn-test1 --gas-prices 0.1ubbn --gas-adjustment 1.5 --gas auto -y 
```

#### Unbond tokens from your validator

```
babylond tx staking unbond $(babylond keys show wallet --bech val -a) 1000000ubbn --from wallet --chain-id bbn-test1 --gas-prices 0.1ubbn --gas-adjustment 1.5 --gas auto -y 
```

#### Send tokens to wallet

```
babylond tx bank send wallet YOUR_TO_WALLET_ADDRESS 1000000ubbn --from wallet --chain-id bbn-test1 --gas-prices 0.1ubbn --gas-adjustment 1.5 --gas auto -y 
```

## Governance

#### List all proposals

```
babylond query gov proposals
```

#### View proposal by id

```
babylond query gov proposal 1
```

#### Vote 'Yes'

```
babylond tx gov vote 1 yes --from wallet --chain-id bbn-test1 --gas-prices 0.1ubbn --gas-adjustment 1.5 --gas auto -y 
```

#### Vote 'No'

```
babylond tx gov vote 1 no --from wallet --chain-id bbn-test1 --gas-prices 0.1ubbn --gas-adjustment 1.5 --gas auto -y 
```

#### Vote 'Abstain'

```
babylond tx gov vote 1 abstain --from wallet --chain-id bbn-test1 --gas-prices 0.1ubbn --gas-adjustment 1.5 --gas auto -y 
```

#### Vote 'No With Veto'

```
babylond tx gov vote 1 no_with_veto --from wallet --chain-id bbn-test1 --gas-prices 0.1ubbn --gas-adjustment 1.5 --gas auto -y 
```

## Utility

#### Update ports

```
CUSTOM_PORT=26
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.babylond/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.babylond/config/app.toml
```

#### Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.babylond/config/app.toml
```

#### Get validator info

```
babylond status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
babylond status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(babylond tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.babylond/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//') 
```

#### Remove node

```
sudo systemctl stop babylond && sudo systemctl disable babylond && sudo rm /etc/systemd/system/babylond.service && sudo systemctl daemon-reload && rm -rf $HOME/.babylond && rm -rf $HOME/babylon && sudo rm -rf $(which babylond) 
```

## Service Management

#### Reload service configuration

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable babylond
```

#### Disable service

```
sudo systemctl disable babylond
```

#### Start service

```
sudo systemctl start babylond
```

#### Stop service

```
sudo systemctl stop babylond
```

#### Restart service

```
sudo systemctl restart babylond
```

#### Check service status

```
sudo systemctl status babylond
```

#### Check service logs

```
sudo journalctl -u babylond -f --no-hostname -o cat
```