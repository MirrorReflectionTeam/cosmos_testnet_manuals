## KEY

#### Add new key

```
nibid keys add wallet
```

#### Recover Existing Key

```
nibid keys add wallet --recover
```

#### Query Wallet Balance

```
nibid q bank balances $(nibid keys show wallet -a)
```

## Validator


#### Create new validator
```
nibid tx staking create-validator \
--amount=10000000unibi \
--pubkey=$(nibid tendermint show-validator) \
--moniker="YOUR_MONIKER_NAME" \
--chain-id=nibiru-testnet-2 \
--commission-rate=0.05 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment=1.4 \
--gas=auto \
--gas-prices=0.025unibi \
-y
```

#### Edit existing validator

```
nibid tx staking edit-validator \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL"
--chain-id nibiru-testnet-2 \
--commission-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.025unibi \
-y
```

#### Unjail validator

```
nibid tx slashing unjail --from wallet --chain-id nibiru-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

#### View validator details

```
nibid q staking validator $(nibid keys show wallet --bech val -a)
```

## Tokens

#### Withdraw commission and rewards from your validator

```
nibid tx distribution withdraw-rewards $(nibid keys show wallet --bech val -a) --commission --from wallet --chain-id nibiru-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

#### Delegate tokens to yourself

```
nibid tx staking delegate $(nibid keys show wallet --bech val -a) 1000000unibi --from wallet --chain-id nibiru-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

#### Delegate tokens to validator

```
nibid tx staking delegate <TO_VALOPER_ADDRESS> 1000000unibi --from wallet --chain-id nibiru-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

#### Unbond tokens from your validator

```
nibid tx staking unbond $(nibid keys show wallet --bech val -a) 1000000unibi --from wallet --chain-id nibiru-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

#### Send tokens to wallet

```
nibid tx bank send wallet <TO_WALLET_ADDRESS> 1000000unibi --from wallet --chain-id nibiru-testnet-2
```

## Governance

#### List all proposals

```
nibid query gov proposals
```

#### View proposal by id

```
nibid query gov proposal 1
```

#### Vote 'Yes'

```
nibid tx gov vote 1 yes --from wallet --chain-id nibiru-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

#### Vote 'No'

```
nibid tx gov vote 1 no --from wallet --chain-id nibiru-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

#### Vote 'Abstain'

```
nibid tx gov vote 1 abstain --from wallet --chain-id nibiru-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

#### Vote 'No With Veto'

```
nibid tx gov vote 1 nowithveto --from wallet --chain-id nibiru-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

## Utility

#### Update ports

```
CUSTOM_PORT=28
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.nibid/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.nibid/config/app.toml
```

#### Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.nibid/config/app.toml
```

#### Get validator info

```b
nibid status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
nibid status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(nibid tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.nibid/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

