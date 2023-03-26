<div align="center">
  <h1> Manual Installation </h1>
</div>

### Ojo

<figure><img src="https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_testnet_manuals/main/project_files/ojo.jpg" width="150" alt=""><figcaption></figcaption></figure>

**[Website](https://ojo.network/)** | **[Explorer](https://ojo.explorers.guru/)**

### Public endpoints

- **https://rpc.ojo-testnet.mirror-reflection.com**

- **https://grpc.ojo-testnet.mirror-reflection.com**

- **https://api.ojo-testnet.mirror-reflection.com**

### Hardware Requirements

4CPU 16RAM 200GB

**[Rent on Hetzner](https://hetzner.cloud/?ref=AwVksaI2T3Nz)** | **[Rent on Contabo](https://contabo.com/en)**

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
cd || return
rm -rf ojo
git clone https://github.com/ojo-network/ojo.git
cd ojo || return
git checkout v0.1.2
make install
ojod version
```

### Initialize the node

```
ojod config chain-id ojo-devnet
ojod init "$NODE_MONIKER" --chain-id ojo-devnet
```

### Download genesis and addrbook

```
curl -Ls https://snapshots.kjnodes.com/ojo-testnet/genesis.json > $HOME/.ojo/config/genesis.json
curl -Ls https://snapshots.kjnodes.com/ojo-testnet/addrbook.json > $HOME/.ojo/config/addrbook.json
```

### Add seeds

```
sed -i -e "s|^seeds *=.*|seeds = \"3f472746f46493309650e5a033076689996c8881@ojo-testnet.rpc.kjnodes.com:50659\"|" $HOME/.ojo/config/config.toml
```

### Set pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.ojo/config/app.toml
```

### Set minimum gas price

```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0uojo\"|" $HOME/.ojo/config/app.toml
```

### Resetting data

```
ojod tendermint unsafe-reset-all --home $HOME/.ojo --keep-addr-book
```

### Create a service

```
sudo tee /etc/systemd/system/ojod.service > /dev/null << EOF
[Unit]
Description=ojo-testnet node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which ojod) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### Download snapshot

```
curl -L https://snapshots.kjnodes.com/ojo-testnet/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.ojo
```

### Start service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable ojod
sudo systemctl start ojod
sudo journalctl -u ojod -f --no-hostname -o cat
```

### Management

When node is synced you must see **FALSE** after command

```
ojod status 2>&1 | jq .SyncInfo.catching_up
```

If you see **FALSE**

### Create wallet

```
ojod keys add wallet
```

SAVE YOUR INPUT SEED PHRASE (24 words)

### SAVE PRIVATE VALIDATOR KEY

```
cat $HOME/.ojo/config/priv_validator_key.json
```

### Go to [discord channel](https://discord.gg/vBejQSntNh) #general-chat and request tokens

### Create Validator

```
ojod tx staking create-validator \
--amount=1000000uojo \
--pubkey=$(ojod tendermint show-validator) \
--moniker="$NODE_MONIKER" \
--chain-id=ojo-devnet \
--commission-rate=0.07 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment=1.4 \
--gas=auto \
--gas-prices 0uojo \
-y
```

### View validator details

```
ojod q staking validator $(ojod keys show wallet --bech val -a)
```

### Install pricefeeder

```
cd || return
rm -rf price-feeder
git clone https://github.com/ojo-network/price-feeder
cd price-feeder || return
git checkout v0.1.1
make install
price-feeder version # version: HEAD-5d46ed438d33d7904c0d947ebc6a3dd48ce0de59
```

### Create directory for pricefeeder

```
mkdir -p $HOME/.price-feeder
curl -s https://raw.githubusercontent.com/ojo-network/price-feeder/main/price-feeder.example.toml > $HOME/.price-feeder/price-feeder.toml
```

### Create new wallet for pricefeeder

```
ojod keys add pricefeeder-wallet --keyring-backend os
```

SAVE YOUR INPUT SEED PHRASE (24 words)

### Fund the pricefeeder-wallet with some testnet tokens

```
ojod tx bank send wallet YOUR_FEEDER_ADDRESS 10000000uojo --from wallet --chain-id ojo-devnet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo -y
```

### View the balance

```
ojod q bank balances $(ojod keys show feeder-wallet --keyring-backend os -a)
```

### Set up variables

```
KEYRING_PASSWORD=YOUR_KEYRING_PASSWORD
CHAIN_ID=ojo-devnet
GRPC="localhost:9090"
RPC="http://localhost:26657"
```
```
WALLET_ADDRESS=$(ojod keys show wallet -a)
FEEDER_ADDRESS=$(ojod keys show feeder-wallet --keyring-backend os -a)
VALIDATOR_ADDRESS=$(ojod keys show wallet --bech val -a)
```

### Delegate pricefeeder responsibility

```
ojod tx oracle delegate-feed-consent $WALLET_ADDRESS $FEEDER_ADDRESS --from wallet --gas-adjustment 1.4 --gas auto --gas-prices 0uojo -y
```

### Check linked pricefeeder address

```
ojod q oracle feeder-delegation $VALIDATOR_ADDRESS
```

### Set pricefeeder configuration values

```
sed -i '/^dir *=.*/a pass = ""' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^address *=.*|address = "'$FEEDER_ADDRESS'"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^chain_id *=.*|chain_id = "'$CHAIN_ID'"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^validator *=.*|validator = "'$VALIDATOR_ADDRESS'"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^backend *=.*|backend = "os"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^dir *=.*|dir = "'$HOME/.ojo'"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^pass *=.*|pass = "'$KEYRING_PASSWORD'"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^grpc_endpoint *=.*|grpc_endpoint = "'$GRPC'"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^tmrpc_endpoint *=.*|tmrpc_endpoint = "'$RPC'"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^global-labels *=.*|global-labels = [["chain_id", "'$CHAIN_ID'"]]|g' $HOME/.price-feeder/price-feeder.toml
```

### Setup the systemd service

```
sudo tee /etc/systemd/system/price-feeder.service > /dev/null << EOF
[Unit]
Description=Ojo Price Feeder
After=network-online.target
[Service]
User=$USER
ExecStart=$(which price-feeder) $HOME/.price-feeder/price-feeder.toml --log-level debug
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="PRICE_FEEDER_PASS=$KEYRING_PASSWORD"
[Install]
WantedBy=multi-user.target
EOF
```

### Start the systemd service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable price-feeder
sudo systemctl start price-feeder
sudo journalctl -u price-feeder -f --no-hostname -o cat
```
