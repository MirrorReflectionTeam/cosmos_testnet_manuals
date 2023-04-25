### Babylon

<figure><img src="https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_testnet_manuals/main/project_files/babylon.png" width="150" alt=""><figcaption></figcaption></figure>

**[Website](https://babylonchain.io/)** | **[Explorer](https://babylon.explorers.guru/)**

### Public endpoints

- **https://rpc.babylon-testnet.mirror-reflection.com**

- **https://grpc.babylon-testnet.mirror-reflection.com**

- **https://api.babylon-testnet.mirror-reflection.com**

### Hardware Requirements

4CPU 16RAM 200GB

**[Rent on Hetzner](https://hetzner.cloud/?ref=AwVksaI2T3Nz)** | **[Rent on Contabo](https://contabo.com/en)**

<div align="center">
  <h1> Automatic Installation </h1>
</div>

```
source <(curl -s https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_scripts/main/babylon/install.sh)
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
rm -rf babylon
git clone https://github.com/babylonchain/babylon
cd babylon 
git checkout v0.5.0
make install
babylond version
```

### Initialize the node

```
babylond config chain-id bbn-test1
babylond init "$NODE_MONIKER" --chain-id bbn-test1
```

### Download genesis and addrbook

```
curl -L https://github.com/babylonchain/networks/blob/main/bbn-test1/genesis.tar.bz2?raw=true > genesis.tar.bz2
tar -xjf genesis.tar.bz2
rm -rf genesis.tar.bz2
mv genesis.json ~/.babylond/config/genesis.json
curl -s https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/babylon-testnet/addrbook.json > $HOME/.babylond/config/addrbook.json
```

### Add seeds

```
SEEDS="465cd1d6ada4a6e9643b64d9c2f7f0afcea02685@rpc.babylon-testnet.mirror-reflection.com:34656,03ce5e1b5be3c9a81517d415f65378943996c864@18.207.168.204:26656,a5fabac19c732bf7d814cf22e7ffc23113dc9606@34.238.169.221:26656"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.babylond/config/config.toml
```

### Set pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.babylond/config/app.toml
```

### Set minimum gas price

```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001ubbn"|g' $HOME/.babylond/config/app.toml
```

### Enable prometheus and network

```
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.babylond/config/config.toml
sed -i 's|^network *=.*|network = "mainnet"|g' $HOME/.babylond/config/app.toml
sed -i 's|^checkpoint-tag *=.*|checkpoint-tag = "bbn0"|g' $HOME/.babylond/config/app.toml
```

### Resetting data

```
babylond tendermint unsafe-reset-all --home $HOME/.babylond --keep-addr-book
```

### Create a service

```
sudo tee /etc/systemd/system/babylond.service > /dev/null << EOF
[Unit]
Description=Babylon Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which babylond) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```

### Download snapshot

```
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/babylon-testnet/bbn-test1_latest.tar | tar -xf - -C $HOME/.babylond/data
```

### Start service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable babylond
sudo systemctl start babylond
sudo journalctl -u babylond -f --no-hostname -o cat
```

<div align="center">
  <h1> Management </h1>
</div>

When node is synced you must see **FALSE** after command

```
babylond status 2>&1 | jq .SyncInfo.catching_up
```

If you see **FALSE**

### Create wallet

```
babylond keys add wallet
```

SAVE YOUR INPUT SEED PHRASE (24 words)

### SAVE PRIVATE VALIDATOR KEY

```
cat $HOME/.babylond/config/priv_validator_key.json
```

### Go to [discord channel](https://discord.gg/babylonchain) #faucet and paste

```
!request YOUR_WALLET_ADDRESS
```

### Verify the balance

```
babylond q bank balances $(babylond keys show wallet -a)
```

### Create bls-key

```
babylond create-bls-key $(babylond keys show wallet -a)
sudo systemctl restart babylond
```

### Create Validator

```
babylond tx checkpointing create-validator \
--amount=10ubbn \
--pubkey=$(babylond tendermint show-validator) \
--moniker="$NODE_MONIKER" \
--chain-id=bbn-test1 \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--fees=20ubbn \
--from=wallet \
-y
```

### View validator details

```
babylond q staking validator $(babylond keys show wallet --bech val -a)
```