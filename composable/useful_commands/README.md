## KEY

#### Add new key

```
banksyd keys add wallet
```

#### Recover Existing Key

```
banksyd keys add wallet --recover
```

#### Query Wallet Balance

```
banksyd q bank balances $(Cardchaind keys show wallet -a)
```

## Validator


#### Create new validator
```
banksyd tx staking create-validator \
--amount=1000000upica \
--pubkey=$(banksyd tendermint show-validator) \
--moniker="$NODE_MONIKER" \
--chain-id=banksy-testnet-2 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment=1.5 \
--gas=auto \
--gas-prices=0.1upica \
-y
```

#### Edit existing validator

```
banksyd tx staking edit-validator \
--moniker="$NODE_MONIKER" \
--identity="YOUR_KEYBASE_ID" \
--details="YOUR_DETAILS" \
--chain-id=banksy-testnet-2 \
--commission-rate=0.1 \
--from=wallet \
--gas-prices=0.1upica \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

#### Unjail validator

```
banksyd tx slashing unjail --from wallet --chain-id banksy-testnet-2 --gas-prices 0.1upica --gas-adjustment 1.5 --gas auto -y 
```

#### View validator details

```
banksyd q staking validator $(banksyd keys show wallet --bech val -a) 
```

## Tokens

#### Withdraw commission and rewards from your validator

```
banksyd tx distribution withdraw-all-rewards --from wallet --chain-id banksy-testnet-2 --gas-prices 0.1upica --gas-adjustment 1.5 --gas auto -y 
```

#### Delegate tokens to yourself

```
banksyd tx staking delegate $(banksyd keys show wallet --bech val -a) 1000000upica --from wallet --chain-id banksy-testnet-2 --gas-prices 0.1upica --gas-adjustment 1.5 --gas auto -y 
```

#### Delegate tokens to validator

```
banksyd tx staking delegate YOUR_TO_VALOPER_ADDRESS 1000000upica --from wallet --chain-id banksy-testnet-2 --gas-prices 0.1upica --gas-adjustment 1.5 --gas auto -y 
```

#### Redelegate tokens to validator

```
banksyd tx staking redelegate $(banksyd keys show wallet --bech val -a) YOUR_TO_VALOPER_ADDRESS 1000000upica --from wallet --chain-id banksy-testnet-2 --gas-prices 0.1upica --gas-adjustment 1.5 --gas auto -y 
```

#### Unbond tokens from your validator

```
banksyd tx staking unbond $(banksyd keys show wallet --bech val -a) 1000000upica --from wallet --chain-id banksy-testnet-2 --gas-prices 0.1upica --gas-adjustment 1.5 --gas auto -y 
```

#### Send tokens to wallet

```
banksyd tx bank send wallet YOUR_TO_WALLET_ADDRESS 1000000upica --from wallet --chain-id banksy-testnet-2 --gas-prices 0.1upica --gas-adjustment 1.5 --gas auto -y 
```

## Governance

#### List all proposals

```
banksyd query gov proposals
```

#### View proposal by id

```
banksyd query gov proposal 1
```

#### Vote 'Yes'

```
banksyd tx gov vote 1 yes --from wallet --chain-id banksy-testnet-2 --gas-prices 0.1upica --gas-adjustment 1.5 --gas auto -y 
```

#### Vote 'No'

```
banksyd tx gov vote 1 no --from wallet --chain-id banksy-testnet-2 --gas-prices 0.1upica --gas-adjustment 1.5 --gas auto -y 
```

#### Vote 'Abstain'

```
banksyd tx gov vote 1 abstain --from wallet --chain-id banksy-testnet-2 --gas-prices 0.1upica --gas-adjustment 1.5 --gas auto -y 
```

#### Vote 'No With Veto'

```
banksyd tx gov vote 1 no_with_veto --from wallet --chain-id banksy-testnet-2 --gas-prices 0.1upica --gas-adjustment 1.5 --gas auto -y 
```

## Utility

#### Update ports

```
CUSTOM_PORT=26
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.banksy/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.banksy/config/app.toml
```

#### Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.banksy/config/app.toml
```

#### Get validator info

```
banksyd status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
banksyd status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(banksyd tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.banksy/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//') 
```

#### Remove node

```
sudo systemctl stop banksyd && sudo systemctl disable banksyd && sudo rm /etc/systemd/system/banksyd.service && sudo systemctl daemon-reload && rm -rf $HOME/.banksy && rm -rf $HOME/composable-testnet && sudo rm -rf $(which banksyd) 
```

## Service Management

#### Reload service configuration

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable banksyd
```

#### Disable service

```
sudo systemctl disable banksyd
```

#### Start service

```
sudo systemctl start banksyd
```

#### Stop service

```
sudo systemctl stop banksyd
```

#### Restart service

```
sudo systemctl restart banksyd
```

#### Check service status

```
sudo systemctl status banksyd
```

#### Check service logs

```
sudo journalctl -u banksyd -f --no-hostname -o cat