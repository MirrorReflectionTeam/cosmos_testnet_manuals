## KEY

#### Add new key

```
uptickd keys add wallet
```

#### Recover Existing Key

```
uptickd keys add wallet --recover
```

#### Query Wallet Balance

```
uptickd q bank balances $(uptickd keys show wallet -a)
```

## Validator


#### Create new validator
```
uptickd tx staking create-validator \
--amount 1000000auptick \
--pubkey $(uptickd tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id uptick_7000-2 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas 1000000 \
--fees 1000000000000000auptick \
-y
```

#### Edit existing validator

```
uptickd tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id uptick_7000-2 \
--commission-rate 0.05 \
--from wallet \
--gas 1000000 \
--fees 1000000000000000auptick \
-y
```

#### Unjail validator

```
uptickd tx slashing unjail --from wallet --chain-id uptick_7000-2 --gas 1000000  --fees 1000000000000000auptick -y
```

#### View validator details

```
uptickd q staking validator $(uptickd keys show wallet --bech val -a)
```

## Tokens

#### Withdraw commission and rewards from your validator

```
uptickd tx distribution withdraw-rewards $(uptickd keys show wallet --bech val -a) --commission --from wallet --chain-id uptick_7000-2 --gas 1000000  --fees 1000000000000000auptick -y
```

#### Delegate tokens to yourself

```
uptickd tx staking delegate $(uptickd keys show wallet --bech val -a) 1000000auptick --from wallet --chain-id uptick_7000-2 --gas 1000000  --fees 1000000000000000auptick -y
```

#### Delegate tokens to validator

```
uptickd tx staking delegate <TO_VALOPER_ADDRESS> 1000000auptick --from wallet --chain-id uptick_7000-2 --gas 1000000  --fees 1000000000000000auptick -y
```

#### Redelegate tokens to validator

```
uptickd tx staking redelegate $(uptickd keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000auptick --from wallet --chain-id uptick_7000-2 --gas 1000000  --fees 1000000000000000auptick -y
```

#### Unbond tokens from your validator

```
uptickd tx staking unbond $(uptickd keys show wallet --bech val -a) 1000000auptick --from wallet --chain-id uptick_7000-2 --gas 1000000  --fees 1000000000000000auptick -y
```

#### Send tokens to wallet

```
uptickd tx bank send wallet <TO_WALLET_ADDRESS> 1000000auptick --from wallet --chain-id uptick_7000-2 --gas 1000000  --fees 1000000000000000auptick -y
```

## Governance

#### List all proposals

```
uptickd query gov proposalss
```

#### View proposal by id

```
uptickd query gov proposal 1
```

#### Vote 'Yes'

```
uptickd tx gov vote 1 yes --from wallet --chain-id uptick_7000-2 --gas 1000000  --fees 1000000000000000auptick -y
```

#### Vote 'No'

```
uptickd tx gov vote 1 no --from wallet --chain-id uptick_7000-2 --gas 1000000  --fees 1000000000000000auptick -y
```

#### Vote 'Abstain'

```
uptickd tx gov vote 1 abstain --from wallet --chain-id uptick_7000-2 --gas 1000000  --fees 1000000000000000auptick -y
```

#### Vote 'No With Veto'

```
uptickd tx gov vote 1 NoWithVeto --from wallet --chain-id uptick_7000-2 --gas 1000000  --fees 1000000000000000auptick -y
```

## Utility

#### Update ports

```
CUSTOM_PORT=26
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.uptickd/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.uptickd/config/app.toml
```

#### Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.uptickd/config/app.toml
```

#### Get validator info

```
uptickd status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
uptickd status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(uptickd tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.uptickd/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Remove node

```
sudo systemctl stop uptickd && sudo systemctl disable uptickd && sudo rm /etc/systemd/system/uptickd.service && sudo systemctl daemon-reload && rm -rf $HOME/.uptickd && rm -rf $HOME/uptick && sudo rm $(which uptickd) 
```

## Service Management

#### Reload service configuration

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable uptickd
```

#### Disable service

```
sudo systemctl disable uptickd
```

#### Start service

```
sudo systemctl start uptickd
```

#### Stop service

```
sudo systemctl stop uptickd
```

#### Restart service

```
sudo systemctl restart uptickd
```

#### Check service status

```
sudo systemctl status uptickd
```

#### Check service logs

```
sudo journalctl -u uptickd -f --no-hostname -o cat
```