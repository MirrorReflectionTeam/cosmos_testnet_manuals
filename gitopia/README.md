<div align="center">
  <h1> Manual Installation </h1>
</div>

### Gitopia

<figure><img src="https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_testnet_manuals/main/project_files/gitopia.png" width="150" alt=""><figcaption></figcaption></figure>

**[Website](https://gitopia.com/)** | **[Explorer](https://gitopia.exploreme.pro/)**

### Public endpoints

- **https://rpc.gitopia-testnet.mirror-reflection.com**

- **https://grpc.gitopia-testnet.mirror-reflection.com**

- **https://api.gitopia-testnet.mirror-reflection.com**

### Hardware Requirements

8CPU 32RAM 200GB

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

### Add Gitopia remote helper
```
curl -Ls https://get.gitopia.com | sudo bash
```

### Download and build binaries

```
cd $HOME
rm -rf gitopia
git clone gitopia://Gitopia/gitopia
cd gitopia 
git checkout v1.2.0
make install
```

### Initialize the node

```
gitopiad config chain-id gitopia-janus-testnet-2
gitopiad init "$NODE_MONIKER" --chain-id gitopia-janus-testnet-2
```

### Download genesis and addrbook

```

curl -s https://server.gitopia.com/raw/gitopia/testnets/master/gitopia-janus-testnet-2/genesis.json.gz > ~/.gitopia/config/genesis.zip
gunzip -c ~/.gitopia/config/genesis.zip > ~/.gitopia/config/genesis.json
rm -rf ~/.gitopia/config/genesis.zip

curl -Ls https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/gitopia-testnet/addrbook.json > $HOME/.gitopia/config/addrbook.json
```

### Add seeds

```
SEEDS="e1ab0573d55ff92fad55d2929e353904f1bbe36f@rpc.gitopia-testnet.mirror-reflection.com:31656,3f472746f46493309650e5a033076689996c8881@gitopia-testnet.rpc.kjnodes.com:41659"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.gitopia/config/config.toml
```

### Set pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
  $HOME/.gitopia/config/app.toml
```

### Set minimum gas price

```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001utlore"|g' $HOME/.gitopia/config/app.toml
```

### Resetting data

```
gitopiad tendermint unsafe-reset-all --home $HOME/.gitopia --keep-addr-book
```

### Create a service

```
sudo tee /etc/systemd/system/gitopiad.service > /dev/null << EOF
[Unit]
Description=Gitopia Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which gitopiad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```

### Download snapshot

```
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/gitopia-testnet/gitopia-janus-testnet-2_latest.tar | tar -xf - -C $HOME/.gitopia/data
```

### Start service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable gitopiad
sudo systemctl start gitopiad
sudo journalctl -u gitopiad -f --no-hostname -o cat
```

### Management

When node is synced you must see **FALSE** after command

```
gitopiad status 2>&1 | jq .SyncInfo.catching_up
```

If you see **FALSE**

### Create wallet

```
gitopiad keys add wallet
```

SAVE YOUR INPUT SEED PHRASE (24 words)

### SAVE PRIVATE VALIDATOR KEY

```
cat $HOME/.gitopia/config/priv_validator_key.json
```

### Go to the [website](https://gitopia.com/login) and connect your wallet for faucet

### Create Validator

```
gitopiad tx staking create-validator \
--amount=9000000utlore \
--pubkey=$(gitopiad tendermint show-validator) \
--moniker="$NODE_MONIKER" \
--chain-id=gitopia-janus-testnet-2 \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--fees=20000utlore \
--from=wallet \
-y
```

### View validator details

```
gitopiad q staking validator $(gitopiad keys show wallet --bech val -a)
```
