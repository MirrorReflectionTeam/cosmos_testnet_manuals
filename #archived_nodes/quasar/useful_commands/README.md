## KEY

#### Add new key

```
quasard keys add wallet
```

#### Recover Existing Key

```
quasard keys add wallet --recover
```

#### Query Wallet Balance

```
quasard q bank balances $(quasard keys show wallet -a)
```

## Validator


#### Create new validator
```
quasard tx staking create-validator \
--amount=1000000uqsr \
--pubkey=$(quasard tendermint show-validator) \
--moniker="YOUR_MONIKER_NAME" \
--chain-id=qsr-questnet-04 \
--commission-rate=0.07 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment=1.4 \
--gas=auto \
--gas-prices=0uqsr \
-y
```

#### Edit existing validator

```
quasard tx staking edit-validator \
--moniker="YOUR_MONIKER_NAME" \
--identity="YOUR_KEYBASE_ID" \
--details="YOUR_DETAILS" \
--website="YOUR_WEBSITE_URL"
--chain-id=qsr-questnet-04 \
--commission-rate=0.07 \
--from=wallet \
--gas-adjustment=1.4 \
--gas=auto \
--gas-prices=0uqsr \
-y
```

#### Unjail validator

```
quasard tx slashing unjail --from wallet --chain-id qsr-questnet-04 --gas-adjustment 1.4 --gas auto --gas-prices 0uqsr -y
```

#### View validator details

```
quasard q staking validator $(quasard keys show wallet --bech val -a) 
```

## Tokens

#### Withdraw commission and rewards from your validator

```
quasard tx distribution withdraw-rewards $(quasard keys show wallet --bech val -a) --commission --from wallet --chain-id qsr-questnet-04 --gas-adjustment 1.4 --gas auto --gas-prices 0uqsr -y
```

#### Delegate tokens to yourself

```
quasard tx staking delegate $(quasard keys show wallet --bech val -a) 1000000uqsr --from wallet --chain-id qsr-questnet-04 --gas-adjustment 1.4 --gas auto --gas-prices 0uqsr -y
```

#### Delegate tokens to validator

```
quasard tx staking delegate <TO_VALOPER_ADDRESS> 1000000uqsr --from wallet --chain-id qsr-questnet-04 --gas-adjustment 1.4 --gas auto --gas-prices 0uqsr -y 
```

#### Redelegate tokens to validator

```
quasard tx staking redelegate $(quasard keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000uqsr --from wallet --chain-id qsr-questnet-04 --gas-adjustment 1.4 --gas auto --gas-prices 0uqsr -y
```

#### Unbond tokens from your validator

```
quasard tx staking unbond $(quasard keys show wallet --bech val -a) 1000000uqsr --from wallet --chain-id qsr-questnet-04 --gas-adjustment 1.4 --gas auto --gas-prices 0uqsr -y
```

#### Send tokens to wallet

```
quasard tx bank send wallet <TO_WALLET_ADDRESS> 1000000uqsr --from wallet --chain-id qsr-questnet-04 
```

## Governance

#### List all proposals

```
quasard query gov proposals
```

#### View proposal by id

```
quasard query gov proposal 1
```

#### Vote 'Yes'

```
quasard tx gov vote 1 yes --from wallet --chain-id qsr-questnet-04 --gas-adjustment 1.4 --gas auto --gas-prices 0uqsr -y 
```

#### Vote 'No'

```
quasard tx gov vote 1 no --from wallet --chain-id qsr-questnet-04 --gas-adjustment 1.4 --gas auto --gas-prices 0uqsr -y
```

#### Vote 'Abstain'

```
quasard tx gov vote 1 abstain --from wallet --chain-id qsr-questnet-04 --gas-adjustment 1.4 --gas auto --gas-prices 0uqsr -y
```

#### Vote 'No With Veto'

```
quasard tx gov vote 1 nowithveto --from wallet --chain-id qsr-questnet-04 --gas-adjustment 1.4 --gas auto --gas-prices 0uqsr -y
```

## Utility

#### Update ports

```
CUSTOM_PORT=26
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.quasarnode/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.quasarnode/config/app.toml
```

#### Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.quasarnode/config/app.toml
```

#### Get validator info

```
quasard status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
quasard status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(quasard tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.quasarnode/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Remove node

```
sudo systemctl stop quasard && sudo systemctl disable quasard && sudo rm /etc/systemd/system/quasard.service && sudo systemctl daemon-reload && rm -rf $HOME/.quasarnode  && sudo rm $(which quasard) 
```

## Service Management

#### Reload service configuration

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable quasard
```

#### Disable service

```
sudo systemctl disable quasard
```

#### Start service

```
sudo systemctl start quasard
```

#### Stop service

```
sudo systemctl stop quasard
```

#### Restart service

```
sudo systemctl restart quasard
```

#### Check service status

```
sudo systemctl status quasard
```

#### Check service logs

```
sudo journalctl -u quasard -f --no-hostname -o cat
```
