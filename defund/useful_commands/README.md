## KEY

#### Add new key

```
defundd keys add wallet
```

#### Recover Existing Key

```
defundd keys add wallet --recover
```

#### Query Wallet Balance

```
defundd q bank balances $(defundd keys show wallet -a)
```

## Validator


#### Create new validator
```
defundd tx staking create-validator \
--amount=1000000ufetf \
--pubkey=$(defundd tendermint show-validator)
--moniker="YOUR_MONIKER_NAME" \
--chain-id=orbit-alpha-1 \
--commission-rate=0.07 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment=1.4 \
--gas=auto \
--gas-prices=0ufetf \
-y
```

#### Edit existing validator

```
defundd tx staking edit-validator \
--moniker="YOUR_MONIKER_NAME" \
--identity="YOUR_KEYBASE_ID" \
--details="YOUR_DETAILS" \
--website="YOUR_WEBSITE_URL"
--chain-id=orbit-alpha-1 \
--commission-rate=0.07 \
--from=wallet \
--gas-adjustment=1.4 \
--gas=auto \
--gas-prices=0ufetf \
-y
```

#### Unjail validator

```
defundd tx slashing unjail --from wallet --chain-id orbit-alpha-1 --gas-prices 0ufetf --gas-adjustment 1.4 --gas auto -y  
```

#### View validator details

```
defundd q staking validator $(defundd keys show wallet --bech val -a) 
```

## Tokens

#### Withdraw commission and rewards from your validator

```
defundd tx distribution withdraw-rewards $(defundd keys show wallet --bech val -a) --commission --from wallet --chain-id orbit-alpha-1 --gas-prices 0ufetf --gas-adjustment 1.4 --gas auto -y
```

#### Delegate tokens to yourself

```
defundd tx staking delegate $(defundd keys show wallet --bech val -a) 1000000ufetf --from wallet --chain-id orbit-alpha-1 --gas-prices 0ufetf --gas-adjustment 1.4 --gas auto -y
```

#### Delegate tokens to validator

```
defundd tx staking delegate <TO_VALOPER_ADDRESS> 1000000ufetf --from wallet --chain-id orbit-alpha-1 --gas-prices 0ufetf --gas-adjustment 1.4 --gas auto -y 
```

#### Redelegate tokens to validator
```
defundd tx staking redelegate $(defundd keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000ufetf --from wallet --chain-id orbit-alpha-1 --gas-prices 0ufetf --gas-adjustment 1.4 --gas auto -y 
```

#### Unbond tokens from your validator

```
defundd tx staking unbond $(defundd keys show wallet --bech val -a) 1000000ufetf --from wallet --chain-id orbit-alpha-1 --gas-prices 0ufetf --gas-adjustment 1.4 --gas auto -y
```

#### Send tokens to wallet

```
defundd tx bank send wallet <TO_WALLET_ADDRESS> 1000000ufetf --from wallet --chain-id orbit-alpha-1 --gas-prices 0ufetf --gas-adjustment 1.4 --gas auto -y 
```

## Governance

#### List all proposals

```
defundd query gov proposals
```

#### View proposal by id

```
defundd query gov proposals 1
```

#### Vote 'Yes'

```
defundd tx gov vote 1 yes --from wallet --chain-id orbit-alpha-1 --gas-prices 0ufetf --gas-adjustment 1.4 --gas auto -y  
```

#### Vote 'No'

```
defundd tx gov vote 1 no --from wallet --chain-id orbit-alpha-1 --gas-prices 0ufetf --gas-adjustment 1.4 --gas auto -y
```

#### Vote 'Abstain'

```
defundd tx gov vote 1 abstain --from wallet --chain-id orbit-alpha-1 --gas-prices 0ufetf --gas-adjustment 1.4 --gas auto -y
```

#### Vote 'No With Veto'

```
defundd tx gov vote 1 nowithveto --from wallet --chain-id orbit-alpha-1 --gas-prices 0ufetf --gas-adjustment 1.4 --gas auto -y 
```

## Utility

#### Update ports

```
CUSTOM_PORT=26
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.defund/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.defund/config/app.toml
```

#### Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.defund/config/app.toml
```

#### Get validator info

```
defundd status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
defundd status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(defundd tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.defund/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Remove node

```
sudo systemctl stop defundd && sudo systemctl disable defundd && sudo rm /etc/systemd/system/defundd.service && sudo systemctl daemon-reload && rm -rf $HOME/.defund && rm -rf $HOME/defund && sudo rm $(which defundd) 
```

## Service Management

#### Reload service configuration

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable defundd
```

#### Disable service

```
sudo systemctl disable defundd
```

#### Start service

```
sudo systemctl start defundd
```

#### Stop service

```
sudo systemctl stop defundd
```

#### Restart service

```
sudo systemctl restart defundd
```

#### Check service status

```
sudo systemctl status defundd
```

#### Check service logs

```
sudo journalctl -u defundd -f --no-hostname -o cat
```
