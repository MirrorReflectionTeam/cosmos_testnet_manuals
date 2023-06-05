### Lava

<figure><img src="https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_testnet_manuals/main/project_files/lava.png" width="150" alt=""><figcaption></figcaption></figure>

**[Website](https://www.lavanet.xyz/)** | **[Explorer](https://lava.exploreme.pro/)**

### Public endpoints

- **https://rpc.lava-testnet.mirror-reflection.com**

- **https://grpc.lava-testnet.mirror-reflection.com**

- **https://api.lava-testnet.mirror-reflection.com**

### Hardware Requirements

4CPU 8RAM 160GB

**[Rent on Hetzner](https://hetzner.cloud/?ref=AwVksaI2T3Nz)** | **[Rent on Contabo](https://contabo.com/en)**

<div align="center">
  <h1> Automatic Installation </h1>
</div>

```
source <(curl -s https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_scripts/main/lava/install.sh)
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
cd $HOME
rm -rf lava
git clone https://github.com/lavanet/lava.git
cd lava
git checkout v0.12.1-hf
make install
lavad version
```

### Initialize the node

```
lavad config chain-id lava-testnet-1
lavad init "$NODE_MONIKER" --chain-id lava-testnet-1
```

### Download genesis and addrbook

```
curl -Ls https://rpc.lava-testnet.mirror-reflection.com/genesis | jq -r .result.genesis > $HOME/.lava/config/genesis.json
curl -Ls https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/lava-testnet/addrbook.json > $HOME/.lava/config/addrbook.json
```

### Add seeds

```
SEEDS="282bdf5c2a47aa09c75cd3bc3d011602eb10556e@rpc.lava-testnet.mirror-reflection.com:29656,3f472746f46493309650e5a033076689996c8881@lava-testnet.rpc.kjnodes.com:44659"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.lava/config/config.toml
```

### Set pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
  $HOME/.lava/config/app.toml
```

### Set minimum gas price

```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025ulava"|g' $HOME/.lava/config/app.toml
```

### Resetting data

```
lavad tendermint unsafe-reset-all --home $HOME/.lava --keep-addr-book
```

### Create a service

```
sudo tee /etc/systemd/system/lavad.service > /dev/null << EOF
[Unit]
Description=Lava Network Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which lavad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```

### Download snapshot

```
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/lava-testnet/lava-testnet-1_latest.tar | tar -xf - -C $HOME/.lava/data
```

### Start service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable lavad
sudo systemctl start lavad
sudo journalctl -u lavad -f --no-hostname -o cat
```

<div align="center">
  <h1> Management </h1>
</div>

When node is synced you must see **FALSE** after command

```
lavad status 2>&1 | jq .SyncInfo.catching_up
```

If you see **FALSE**

### Create wallet

```
lavad keys add wallet
```

SAVE YOUR INPUT SEED PHRASE (24 words)

### SAVE PRIVATE VALIDATOR KEY

```
cat $HOME/.lava/config/priv_validator_key.json
```

### Go to [discord channel](https://discord.gg/lavanetxyz) #faucet and paste

```
$request YOUR_WALLET_ADDRESS
```

### Create Validator

```
lavad tx staking create-validator \
--amount=9000000ulava \
--pubkey=$(lavad tendermint show-validator) \
--moniker="$NODE_MONIKER" \
--chain-id=lava-testnet-1 \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--fees=10000ulava \
--from=wallet \
-y
```

### View validator details

```
lavad q staking validator $(lavad keys show wallet --bech val -a)
```
