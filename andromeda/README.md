<div align="center">
  <h1> Manual Installation </h1>
</div>

### Andromeda

<figure><img src="https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_testnet_manuals/main/project_files/andromeda.jpg" width="150" alt=""><figcaption></figcaption></figure>

**[Website](https://andromedaprotocol.io/)** | **[Explorer](https://andromeda.explorers.guru/)**

### Hardware Requirements

4CPU 16RAM 200GB

**[Rent on Hetzner](https://hetzner.cloud/?ref=AwVksaI2T3Nz)**

### Install dependencies

#### Install update, build tools and GO if you needed

```
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential
sudo apt -qy upgrade
```

```
sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.19.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
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
cd $HOME
rm -rf andromedad
git clone https://github.com/andromedaprotocol/andromedad.git
cd andromedad
git checkout galileo-3-v1.1.0-beta1
make install
```

### Initialize the node

```
andromedad config chain-id galileo-3
andromedad init "$NODE_MONIKER" --chain-id galileo-3
```

### Download genesis and addrbook

```
curl -Ls https://snapshots.kjnodes.com/andromeda-testnet/genesis.json > $HOME/.andromedad/config/genesis.json
curl -Ls https://snapshots.kjnodes.com/andromeda-testnet/addrbook.json > $HOME/.andromedad/config/addrbook.json
```

### Add seeds

```
SEEDS=""
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.andromedad/config/config.toml
```

### Set pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.andromedad/config/app.toml
```

### Set minimum gas price

```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0001uandr\"|" $HOME/.andromedad/config/app.toml
```

### Resetting data

```
andromedad tendermint unsafe-reset-all --home $HOME/.andromedad --keep-addr-book
```

### Create a service

```
sudo tee /etc/systemd/system/andromedad.service > /dev/null << EOF
[Unit]
Description=andromeda-testnet node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which nibid) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### Download snapshot

```
curl -L https://snapshots.kjnodes.com/andromeda-testnet/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.andromedad
```

### Start service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable andromedad
sudo systemctl start andromedad
sudo journalctl -u andromedad -f --no-hostname -o cat
```

### Management

When node is synced you must see **FALSE** after command

```
curl -s localhost:26657/status | jq .result.sync_info.catching_up
```

If you see **FALSE**

### Create wallet

```
andromedad keys add wallet
```

SAVE YOUR INPUT SEED PHRASE (24 words)

### SAVE PRIVATE VALIDATOR KEY

/.andromedad/config/priv_validator_key.json

### Go to [discord channel](https://discord.gg/EXEYrRUPfT) #faucet-pub and paste

```
!request YOUR_WALLET_ADDRESS
```

### Create Validator

```
andromedad tx staking create-validator \
--amount=1000000uandr \
--pubkey=$(andromedad tendermint show-validator) \
--moniker="$NODE_MONIKER" \
--chain-id=galileo-3 \
--commission-rate=0.05 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.0001uandr \
-y
```

### View validator details

```
andromedad q staking validator $(andromedad keys show wallet --bech val -a)
```
