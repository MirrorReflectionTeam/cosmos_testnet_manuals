### Andromeda

<figure><img src="https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_testnet_manuals/main/project_files/andromeda.jpg" width="150" alt=""><figcaption></figcaption></figure>

**[Website](https://andromedaprotocol.io/)** | **[Explorer](https://andromeda.exploreme.pro/)**

### Public endpoints

- **https://rpc.andromeda-testnet.mirror-reflection.com**

- **https://grpc.andromeda-testnet.mirror-reflection.com**

- **https://api.andromeda-testnet.mirror-reflection.com**

### Hardware Requirements

4CPU 16RAM 200GB

**[Rent on Hetzner](https://hetzner.cloud/?ref=AwVksaI2T3Nz)** | **[Rent on Contabo](https://contabo.com/en)**

<div align="center">
  <h1> Automatic Installation </h1>
</div>

```
source <(curl -s https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_scripts/main/andromeda/install.sh)
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
curl -s https://rpc.andromeda-testnet.mirror-reflection.com/genesis | jq -r .result.genesis > $HOME/.andromedad/config/genesis.json
curl -s https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/andromeda-testnet/addrbook.json > $HOME/.andromedad/config/addrbook.json
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
Description=andromeda-testnet node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which andromedad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### Download snapshot

```
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/andromeda-testnet/galileo-3_latest.tar | tar -xf - -C $HOME/.andromedad/data
```

### Start service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable andromedad
sudo systemctl start andromedad
sudo journalctl -u andromedad -f --no-hostname -o cat
```

<div align="center">
  <h1> Management </h1>
</div>

When node is synced you must see **FALSE** after command

```
andromedad status 2>&1 | jq .SyncInfo.catching_up
```

If you see **FALSE**

### Create wallet

```
andromedad keys add wallet
```

SAVE YOUR INPUT SEED PHRASE (24 words)

### SAVE PRIVATE VALIDATOR KEY

```
cat $HOME/.andromedad/config/priv_validator_key.json
```

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
--commission-rate=0.07 \
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
