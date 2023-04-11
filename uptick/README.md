### Uptick

<figure><img src="https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_testnet_manuals/main/project_files/uptick.png" width="150" alt=""><figcaption></figcaption></figure>

**[Website](https://www.uptick.network/)** | **[Explorer](https://uptick-testnet.exploreme.pro/)**

### Public endpoints

- **https://rpc.uptick-testnet.mirror-reflection.com**

- **https://grpc.uptick-testnet.mirror-reflection.com**

- **https://api.uptick-testnet.mirror-reflection.com**

### Hardware Requirements

4CPU 8RAM 160GB

**[Rent on Hetzner](https://hetzner.cloud/?ref=AwVksaI2T3Nz)** | **[Rent on Contabo](https://contabo.com/en)**

<div align="center">
  <h1> Automatic Installation </h1>
</div>

```
source <(curl -s https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_scripts/main/uptick/install.sh)
```

<div align="center">
  <h1> Manual Installation </h1>
</div>

### Install dependencies

#### Install update, build tools and GO if you needed

```
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential
sudo apt -qy upgrade
```

```
sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.19.7.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```

### Setup Validator Name

Replace YOUR_MONIKER_NAME with your node name

```
NODE_MONIKER="YOUR_MONIKER_NAME"
```

### Download and build binaries

```
cd $HOME || return
rm -rf uptick
git clone https://github.com/UptickNetwork/uptick.git
cd uptick || return
git checkout v0.2.6
make build -B
sudo mv build/uptickd /usr/local/bin/uptickd
uptickd version
```

### Initialize the node

```
uptickd config chain-id uptick_7000-2
uptickd init "$NODE_MONIKER" --chain-id uptick_7000-2
```

### Download genesis and addrbook

```
curl -Ls https://rpc.uptick-testnet.mirror-reflection.com/genesis | jq -r .result.genesis > $HOME/.uptickd/config/genesis.json
curl -Ls https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/uptick-testnet/addrbook.json > $HOME/.uptickd/config/addrbook.json
```

### Add seeds

```
SEEDS=""
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.uptickd/config/config.toml
```

### Set pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.uptickd/config/app.toml
```

### Set minimum gas price

```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001auptick"|g' $HOME/.uptickd/config/app.toml
```

### Resetting data

```
uptickd tendermint unsafe-reset-all --home $HOME/.uptickd/ --keep-addr-book
```

### Create a service

```
sudo tee /etc/systemd/system/uptickd.service > /dev/null << EOF
[Unit]
Description=Uptick Network Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which uptickd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```

### Download snapshot

```
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/uptick-testnet/uptick_117-1_latest.tar | tar -xf - -C $HOME/.uptickd/data
```

### Start service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable uptickd
sudo systemctl start uptickd
sudo journalctl -u uptickd -f --no-hostname -o cat
```

### Management

When node is synced you must see **FALSE** after command

```
uptickd status 2>&1 | jq .SyncInfo.catching_up
```

If you see **FALSE**

### Create wallet

```
uptickd keys add wallet
```

SAVE YOUR INPUT SEED PHRASE (24 words)

### SAVE PRIVATE VALIDATOR KEY

```
cat $HOME/.uptickd/config/priv_validator_key.json
```

### Go to [discord channel](https://discord.gg/K2aKQuVH) #faucet and request tokens
```
$faucet YOUR_WALLET_ADDRESS
```

### Create Validator

```
uptickd tx staking create-validator \
--amount=4000000000000000000auptick \
--pubkey=$(uptickd tendermint show-validator) \
--moniker="$NODE_MONIKER" \
--chain-id=uptick_7000-2 \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--fees=20000auptick \
--gas=auto \
--from=wallet \
-y
```

### View validator details

```
uptickd q staking validator $(uptickd keys show wallet --bech val -a)
```