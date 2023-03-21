## KEY

#### Add new key

```
ojod keys add wallet
```

#### Recover Existing Key

```
ojod keys add wallet --recover
```

#### Query Wallet Balance

```
ojod q bank balances $(ojod keys show wallet -a)
```

## Validator


#### Create new validator
```
ojod tx staking create-validator \
--amount=1000000uojo \
--pubkey=$(ojod tendermint show-validator) \
--moniker="YOUR_MONIKER_NAME" \
--chain-id=ojo-devnet \
--commission-rate=0.07 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment=1.4 \
--gas=auto \
--gas-prices=0uojo \
-y
```

#### Edit existing validator

```
ojod tx staking edit-validator \
--moniker="YOUR_MONIKER_NAME" \
--identity="YOUR_KEYBASE_ID" \
--details="YOUR_DETAILS" \
--website="YOUR_WEBSITE_URL"
--chain-id=ojo-devnet \
--commission-rate=0.07 \
--from=wallet \
--gas-adjustment=1.4 \
--gas=auto \
--gas-prices=0uojo \
-y
```

#### Unjail validator

```
ojod tx slashing unjail --from wallet --chain-id ojo-devnet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo -y
```

#### View validator details

```
ojod q staking validator $(ojod keys show wallet --bech val -a) 
```

## Tokens

#### Withdraw commission and rewards from your validator

```
ojod tx distribution withdraw-rewards $(ojod keys show wallet --bech val -a) --commission --from wallet --chain-id ojo-devnet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo -y
```

#### Delegate tokens to yourself

```
ojod tx staking delegate $(ojod keys show wallet --bech val -a) 1000000uojo --from wallet --chain-id ojo-devnet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo -y
```

#### Delegate tokens to validator

```
ojod tx staking delegate <TO_VALOPER_ADDRESS> 1000000uojo --from wallet --chain-id ojo-devnet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo -y 
```

#### Redelegate tokens to validator

```
ojod tx staking redelegate $(ojod keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000uojo --from wallet --chain-id ojo-devnet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo -y
```

#### Unbond tokens from your validator

```
ojod tx staking unbond $(ojod keys show wallet --bech val -a) 1000000uojo --from wallet --chain-id ojo-devnet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo -y
```

#### Send tokens to wallet

```
ojod tx bank send wallet <TO_WALLET_ADDRESS> 1000000uojo --from wallet --chain-id ojo-devnet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo -y 
```

## Governance

#### List all proposals

```
ojod query gov proposals
```

#### View proposal by id

```
ojod query gov proposal 1
```

#### Vote 'Yes'

```
ojod tx gov vote 1 yes --from wallet --chain-id ojo-devnet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo -y```

#### Vote 'No'

```
ojod tx gov vote 1 no --from wallet --chain-id ojo-devnet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo -y```

#### Vote 'Abstain'

```
ojod tx gov vote 1 abstain --from wallet --chain-id ojo-devnet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo -y```

#### Vote 'No With Veto'

```
ojod tx gov vote 1 NoWithVeto --from wallet --chain-id ojo-devnet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo -y```

## Utility

#### Update ports

```
CUSTOM_PORT=26
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.ojo/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.ojo/config/app.toml
```

#### Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.ojo/config/app.toml
```

#### Get validator info

```
ojod status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
ojod status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(ojod tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.ojo/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Remove node

```
sudo systemctl stop ojod && sudo systemctl disable ojod && sudo rm /etc/systemd/system/ojod.service && sudo systemctl daemon-reload && rm -rf $HOME/.ojo && rm -rf $HOME/ojo && sudo rm $(which ojod) 
```

## Service Management

#### Reload service configuration

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable ojod
```

#### Disable service

```
sudo systemctl disable ojod
```

#### Start service

```
sudo systemctl start ojod
```

#### Stop service

```
sudo systemctl stop ojod
```

#### Restart service

```
sudo systemctl restart ojod
```

#### Check service status

```
sudo systemctl status ojod
```

#### Check service logs

```
sudo journalctl -u ojod -f --no-hostname -o cat
```
