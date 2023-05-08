## KEY

#### Add new key

```
gitopiad keys add wallet
```

#### Recover Existing Key

```
gitopiad keys add wallet --recover
```

#### Query Wallet Balance

```
gitopiad q bank balances $(gitopiad keys show wallet -a)
```

## Validator

#### Create new validator

```
gitopiad tx staking create-validator \
--amount 1000000utlore \
--pubkey $(gitopiad tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id gitopia-janus-testnet-2 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.001utlore \
-y
```

#### Edit existing validator

```
gitopiad tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL"
--chain-id gitopia-janus-testnet-2 \
--commission-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.001utlore \
-y
```

#### Unjail validator

```
gitopiad tx slashing unjail --from wallet --chain-id gitopia-janus-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.001utlore -y
```

#### View validator details

```
gitopiad q staking validator $(gitopiad keys show wallet --bech val -a)
```

## Tokens

#### Withdraw commission and rewards from your validator

```
gitopiad tx distribution withdraw-rewards $(gitopiad keys show wallet --bech val -a) --commission --from wallet --chain-id gitopia-janus-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.001utlore -y
```

#### Delegate tokens to yourself

```
gitopiad tx staking delegate $(gitopiad keys show wallet --bech val -a) 1000000utlore --from wallet --chain-id gitopia-janus-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.001utlore -y
```

#### Delegate tokens to validator

```
gitopiad tx staking delegate <TO_VALOPER_ADDRESS> 1000000utlore --from wallet --chain-id gitopia-janus-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.001utlore -y
```

#### Redelegate tokens to validator

```
gitopiad tx staking redelegate $(gitopiad keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000utlore --from wallet --chain-id gitopia-janus-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.001utlore -y
```

#### Unbond tokens from your validator

```
gitopiad tx staking unbond $(gitopiad keys show wallet --bech val -a) 1000000utlore --from wallet --chain-id gitopia-janus-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.001utlore -y
```

#### Send tokens to wallet

```
gitopiad tx bank send wallet <TO_WALLET_ADDRESS> 1000000utlore --from wallet --chain-id gitopia-janus-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.001utlore -yy
```

## Governance

#### List all proposals

```
gitopiad query gov proposals
```

#### View proposal by id

```
gitopiad query gov proposal 1
```

#### Vote 'Yes'

```
gitopiad tx gov vote 1 yes --from wallet --chain-id gitopia-janus-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.001utlore -y
```

#### Vote 'No'

```
gitopiad tx gov vote 1 no --from wallet --chain-id gitopia-janus-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.001utlore -y
```

#### Vote 'Abstain'

```
gitopiad tx gov vote 1 abstain --from wallet --chain-id gitopia-janus-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.001utlore -y
```

#### Vote 'No With Veto'

```
gitopiad tx gov vote 1 NoWithVeto --from wallet --chain-id gitopia-janus-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.001utlore -y
```

## Utility

#### Update ports

```
CUSTOM_PORT=26
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.gitopia/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.gitopia/config/app.toml
```

#### Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.gitopia/config/app.toml
```

#### Get validator info

```
gitopiad status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
gitopiad status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(gitopiad tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.gitopia/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Remove node

```
sudo systemctl stop gitopiad && sudo systemctl disable gitopiad && sudo rm /etc/systemd/system/gitopiad.service && sudo systemctl daemon-reload && rm -rf $HOME/.gitopia && rm -rf $HOME/gitopia && sudo rm $(which gitopia)
```

## Service Management

#### Reload service configuration

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable gitopiad
```

#### Disable service

```
sudo systemctl disable gitopiad
```

#### Start service

```
sudo systemctl start gitopiad
```

#### Stop service

```
sudo systemctl stop gitopiad
```

#### Restart service

```
sudo systemctl restart gitopiad
```

#### Check service status

```
sudo systemctl status gitopiad
```

#### Check service logs

```
sudo journalctl -u gitopiad -f --no-hostname -o cat
```
