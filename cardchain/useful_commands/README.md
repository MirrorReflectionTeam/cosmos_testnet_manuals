## KEY

#### Add new key

```
Cardchaind keys add wallet
```

#### Recover Existing Key

```
Cardchaind keys add wallet --recover
```

#### Query Wallet Balance

```
Cardchaind q bank balances $(Cardchaind keys show wallet -a)
```

## Validator


#### Create new validator
```
Cardchaind tx staking create-validator \
--amount=1000000ubpf \
--pubkey=$(Cardchaind tendermint show-validator) \
--moniker="Moniker" \
--chain-id=Testnet3 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.1ubpf \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

#### Edit existing validator

```
Cardchaind tx staking edit-validator \
--moniker="Moniker" \
--identity="YOUR_KEYBASE_ID" \
--details="YOUR_DETAILS" \
--chain-id=Testnet3 \
--commission-rate=0.1 \
--from=wallet \
--gas-prices=0.1ubpf \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

#### Unjail validator

```
Cardchaind tx slashing unjail --from wallet --chain-id Testnet3 --gas-prices 0.1ubpf --gas-adjustment 1.5 --gas auto -y 
```

#### View validator details

```
Cardchaind q staking validator $(Cardchaind keys show wallet --bech val -a) 
```

## Tokens

#### Withdraw commission and rewards from your validator

```
Cardchaind tx distribution withdraw-rewards $(Cardchaind keys show wallet --bech val -a) --commission --from wallet --chain-id Testnet3 --gas-prices 0.1ubpf --gas-adjustment 1.5 --gas auto -y 
```

#### Delegate tokens to yourself

```
Cardchaind tx staking delegate $(Cardchaind keys show wallet --bech val -a) 1000000ubpf --from wallet --chain-id Testnet3 --gas-prices 0.1ubpf --gas-adjustment 1.5 --gas auto -y 
```

#### Delegate tokens to validator

```
Cardchaind tx staking delegate YOUR_TO_VALOPER_ADDRESS 1000000ubpf --from wallet --chain-id Testnet3 --gas-prices 0.1ubpf --gas-adjustment 1.5 --gas auto -y 
```

#### Redelegate tokens to validator

```
Cardchaind tx staking redelegate $(Cardchaind keys show wallet --bech val -a) YOUR_TO_VALOPER_ADDRESS 1000000ubpf --from wallet --chain-id Testnet3 --gas-prices 0.1ubpf --gas-adjustment 1.5 --gas auto -y 
```

#### Unbond tokens from your validator

```
Cardchaind tx staking unbond $(Cardchaind keys show wallet --bech val -a) 1000000ubpf --from wallet --chain-id Testnet3 --gas-prices 0.1ubpf --gas-adjustment 1.5 --gas auto -y 
```

#### Send tokens to wallet

```
Cardchaind tx bank send wallet YOUR_TO_WALLET_ADDRESS 1000000ubpf --from wallet --chain-id Testnet3 --gas-prices 0.1ubpf --gas-adjustment 1.5 --gas auto -y 
```

## Governance

#### List all proposals

```
Cardchaind query gov proposals
```

#### View proposal by id

```
Cardchaind query gov proposal 1
```

#### Vote 'Yes'

```
Cardchaind tx gov vote 1 yes --from wallet --chain-id Testnet3 --gas-prices 0.1ubpf --gas-adjustment 1.5 --gas auto -y 
```

#### Vote 'No'

```
Cardchaind tx gov vote 1 no --from wallet --chain-id Testnet3 --gas-prices 0.1ubpf --gas-adjustment 1.5 --gas auto -y 
```

#### Vote 'Abstain'

```
Cardchaind tx gov vote 1 abstain --from wallet --chain-id Testnet3 --gas-prices 0.1ubpf --gas-adjustment 1.5 --gas auto -y 
```

#### Vote 'No With Veto'

```
Cardchaind tx gov vote 1 no_with_veto --from wallet --chain-id Testnet3 --gas-prices 0.1ubpf --gas-adjustment 1.5 --gas auto -y 
```

## Utility

#### Update ports

```
CUSTOM_PORT=26
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.Cardchain/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.Cardchain/config/app.toml
```

#### Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.Cardchain/config/app.toml
```

#### Get validator info

```
Cardchaind status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
Cardchaind status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(Cardchaind tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.Cardchain/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//') 
```

#### Remove node

```
sudo systemctl stop Cardchaind && sudo systemctl disable Cardchaind && sudo rm /etc/systemd/system/Cardchaind.service && sudo systemctl daemon-reload && rm -rf $HOME/.Cardchain && rm -rf $HOME/Cardchain && sudo rm -rf $(which Cardchaind) 
```

## Service Management

#### Reload service configuration

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable Cardchaind
```

#### Disable service

```
sudo systemctl disable Cardchaind
```

#### Start service

```
sudo systemctl start Cardchaind
```

#### Stop service

```
sudo systemctl stop Cardchaind
```

#### Restart service

```
sudo systemctl restart Cardchaind
```

#### Check service status

```
sudo systemctl status Cardchaind
```

#### Check service logs

```
sudo journalctl -u Cardchaind -f --no-hostname -o cat
```