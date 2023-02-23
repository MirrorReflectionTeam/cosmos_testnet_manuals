## KEY

#### Add new key

```
andromedad keys add wallet
```

#### Recover Existing Key

```
andromedad keys add wallet --recover
```

#### Query Wallet Balance

```
andromedad q bank balances $(andromedad keys show wallet -a)
```

## Validator


#### Create new validator
```
andromedad tx staking create-validator \
--amount=1000000uandr \
--pubkey=$(nibid tendermint show-validator) \
--moniker="YOUR_MONIKER_NAME" \
--chain-id=galileo-3 \
--commission-rate=0.05 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment=1.4 \
--gas=auto \
--gas-prices=0.1uandr \
-y
```

#### Edit existing validator

```
andromedad tx staking edit-validator \
--moniker="YOUR_MONIKER_NAME" \
--identity="YOUR_KEYBASE_ID" \
--details="YOUR_DETAILS" \
--website="YOUR_WEBSITE_URL"
--chain-id=galileo-3 \
--commission-rate=0.05 \
--from=wallet \
--gas-adjustment=1.4 \
--gas=auto \
--gas-prices=0.1uandr \
-y
```

#### Unjail validator

```
andromedad tx slashing unjail --from wallet --chain-id galileo-3 --gas-prices 0.1uandr --gas-adjustment 1.4 --gas auto -y 
```

#### View validator details

```
andromedad q staking validator $(andromedad keys show wallet --bech val -a) 
```

## Tokens

#### Withdraw commission and rewards from your validator

```
andromedad tx distribution withdraw-rewards $(andromedad keys show wallet --bech val -a) --commission --from wallet --chain-id galileo-3 --gas-prices 0.1uandr --gas-adjustment 1.4 --gas auto -y
```

#### Delegate tokens to yourself

```
andromedad tx staking delegate $(andromedad keys show wallet --bech val -a) 1000000uandr --from wallet --chain-id galileo-3 --gas-prices 0.1uandr --gas-adjustment 1.4 --gas auto -y
```

#### Delegate tokens to validator

```
andromedad tx staking delegate <TO_VALOPER_ADDRESS> 1000000uandr --from wallet --chain-id galileo-3 --gas-prices 0.1uandr --gas-adjustment 1.4 --gas auto -y 
```

#### Unbond tokens from your validator

```
andromedad tx staking unbond $(andromedad keys show wallet --bech val -a) 1000000uandr --from wallet --chain-id galileo-3 --gas-prices 0.1uandr --gas-adjustment 1.4 --gas auto -y
```

#### Send tokens to wallet

```
andromedad tx bank send wallet <TO_WALLET_ADDRESS> 1000000uandr --from wallet --chain-id galileo-3 --gas-prices 0.1uandr --gas-adjustment 1.4 --gas auto -y 
```

## Governance

#### List all proposals

```
andromedad query gov proposals
```

#### View proposal by id

```
andromedad query gov proposal 1
```

#### Vote 'Yes'

```
andromedad tx gov vote 1 yes --from wallet --chain-id galileo-3 --gas-prices 0.1uandr --gas-adjustment 1.4 --gas auto -y 
```

#### Vote 'No'

```
andromedad tx gov vote 1 no --from wallet --chain-id galileo-3 --gas-prices 0.1uandr --gas-adjustment 1.4 --gas auto -y
```

#### Vote 'Abstain'

```
andromedad tx gov vote 1 abstain --from wallet --chain-id galileo-3 --gas-prices 0.1uandr --gas-adjustment 1.4 --gas auto -y
```

#### Vote 'No With Veto'

```
andromedad tx gov vote 1 nowithveto --from wallet --chain-id galileo-3 --gas-prices 0.1uandr --gas-adjustment 1.4 --gas auto -y 
```

## Utility

#### Update ports

```
CUSTOM_PORT=26
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.andromedad/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.andromedad/config/app.toml
```

#### Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.andromedad/config/app.toml
```

#### Get validator info

```
andromedad status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
andromedad status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(andromedad tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.andromedad/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Remove node

```
sudo systemctl stop andromedad && sudo systemctl disable andromedad && sudo rm /etc/systemd/system/andromedad.service && sudo systemctl daemon-reload && rm -rf $HOME/.andromedad && rm -rf $HOME/andromedad && sudo rm $(which andromedad) 
```

## Service Management

#### Reload service configuration

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable andromedad
```

#### Disable service

```
sudo systemctl disable andromedad
```

#### Start service

```
sudo systemctl start andromedad
```

#### Stop service

```
sudo systemctl stop andromedad
```

#### Restart service

```
sudo systemctl restart andromedad
```

#### Check service status

```
sudo systemctl status andromedad
```

#### Check service logs

```
sudo journalctl -u andromedad -f --no-hostname -o cat
```
