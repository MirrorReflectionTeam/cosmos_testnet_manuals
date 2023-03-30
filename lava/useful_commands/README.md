## KEY

#### Add new key

```
lavad keys add wallet
```

#### Recover Existing Key

```
lavad keys add wallet --recover
```

#### Query Wallet Balance

```
lavad q bank balances $(lavad keys show wallet -a)
```

## Validator

#### Create new validator

```
lavad tx staking create-validator \
--amount 1000000ulava \
--pubkey $(lavad tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id lava-testnet-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0ulava \
-y
```

#### Edit existing validator

```
lavad tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL"
--chain-id lava-testnet-1 \
--commission-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0ulava \
-y
```

#### Unjail validator

```
lavad tx slashing unjail --from wallet --chain-id lava-testnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ulava -y
```

#### View validator details

```
lavad q staking validator $(lavad keys show wallet --bech val -a)
```

## Tokens

#### Withdraw commission and rewards from your validator

```
lavad tx distribution withdraw-rewards $(lavad keys show wallet --bech val -a) --commission --from wallet --chain-id lava-testnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ulava -y
```

#### Delegate tokens to yourself

```
lavad tx staking delegate $(lavad keys show wallet --bech val -a) 1000000ulava --from wallet --chain-id lava-testnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ulava -y
```

#### Delegate tokens to validator

```
lavad tx staking delegate <TO_VALOPER_ADDRESS> 1000000ulava --from wallet --chain-id lava-testnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ulava -y
```

#### Redelegate tokens to validator

```
lavad tx staking redelegate $(lavad keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000ulava --from wallet --chain-id lava-testnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ulava -y
```

#### Unbond tokens from your validator

```
lavad tx staking unbond $(lavad keys show wallet --bech val -a) 1000000ulava --from wallet --chain-id lava-testnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ulava -y
```

#### Send tokens to wallet

```
lavad tx bank send wallet <TO_WALLET_ADDRESS> 1000000ulava --from wallet --chain-id lava-testnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ulava -y
```

## Governance

#### List all proposals

```
lavad query gov proposals
```

#### View proposal by id

```
lavad query gov proposal 1
```

#### Vote 'Yes'

```
lavad tx gov vote 1 yes --from wallet --chain-id lava-testnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ulava -y
```

#### Vote 'No'

```
lavad tx gov vote 1 no --from wallet --chain-id lava-testnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ulava -y
```

#### Vote 'Abstain'

```
lavad tx gov vote 1 abstain --from wallet --chain-id lava-testnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ulava -y
```

#### Vote 'No With Veto'

```
lavad tx gov vote 1 NoWithVeto --from wallet --chain-id lava-testnet-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ulava -y
```

## Utility

#### Update ports

```
CUSTOM_PORT=26
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.lava/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.lava/config/app.toml
```

#### Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.lava/config/app.toml
```

#### Get validator info

```
lavad status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
lavad status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(lavad tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.lava/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Remove node

```
sudo systemctl stop lavad && sudo systemctl disable lavad && sudo rm /etc/systemd/system/lavad.service && sudo systemctl daemon-reload && rm -rf $HOME/.lava && rm -rf $HOME/lava && sudo rm $(which lavad)
```

## Service Management

#### Reload service configuration

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable lavad
```

#### Disable service

```
sudo systemctl disable lavad
```

#### Start service

```
sudo systemctl start lavad
```

#### Stop service

```
sudo systemctl stop lavad
```

#### Restart service

```
sudo systemctl restart lavad
```

#### Check service status

```
sudo systemctl status lavad
```

#### Check service logs

```
sudo journalctl -u lavad -f --no-hostname -o cat
```
