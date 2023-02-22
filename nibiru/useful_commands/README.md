## KEY

### Add new key

```
nibid keys add wallet
```

### Recover Existing Key

```
nibid keys add wallet --recover
```

### Query Wallet Balance

```
nibid q bank balances $(nibid keys show wallet -a)
```

## Validator


### Create new validator
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

### Edit existing validator

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

### Unjail validator

```
nibid tx slashing unjail --from wallet --chain-id nibiru-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

### View validator details

```
nibid q staking validator $(nibid keys show wallet --bech val -a)
```

## Tokens

### Withdraw commission and rewards from your validator

```
nibid tx distribution withdraw-rewards $(nibid keys show wallet --bech val -a) --commission --from wallet --chain-id nibiru-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

### Delegate tokens to yourself

```
nibid tx staking delegate $(nibid keys show wallet --bech val -a) 1000000unibi --from wallet --chain-id nibiru-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

### Delegate tokens to validator

```
nibid tx staking delegate <TO_VALOPER_ADDRESS> 1000000unibi --from wallet --chain-id nibiru-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

### Unbond tokens from your validator

```
nibid tx staking unbond $(nibid keys show wallet --bech val -a) 1000000unibi --from wallet --chain-id nibiru-testnet-2 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

### Send tokens to wallet

```
nibid tx bank send wallet <TO_WALLET_ADDRESS> 1000000unibi --from wallet --chain-id nibiru-testnet-2
```


