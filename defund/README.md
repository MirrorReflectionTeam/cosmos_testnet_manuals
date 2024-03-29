### DeFund

<figure><img src="https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_testnet_manuals/main/project_files/defund.jpg" width="150" alt=""><figcaption></figcaption></figure>

**[Website](https://www.defund.app/)** | **[Explorer](https://defund.explorers.guru/)**

### Public endpoints

- **https://rpc.defund-testnet.mirror-reflection.com**

- **https://grpc.defund-testnet.mirror-reflection.com**

- **https://api.defund-testnet.mirror-reflection.com**


### Hardware Requirements

8CPU 32RAM 200GB

**[Rent on Hetzner](https://hetzner.cloud/?ref=AwVksaI2T3Nz)** | **[Rent on Contabo](https://contabo.com/en)**

<div align="center">
  <h1> Automatic Installation </h1>
</div>

```
source <(curl -s https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_scripts/main/defund/install.sh)
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
rm -rf defund
git clone https://github.com/defund-labs/defund.git
cd defund
git checkout v0.2.6
make install
```

### Initialize the node

```
defundd config chain-id orbit-alpha-1
defundd init "$NODE_MONIKER" --chain-id orbit-alpha-1
```

### Download genesis and addrbook

```
curl -Ls https://rpc.defund-testnet.mirror-reflection.com/genesis | jq -r .result.genesis > $HOME/.defund/config/genesis.json
curl -s https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/defund-testnet/addrbook.json > $HOME/.defund/config/addrbook.json
```

### Add seeds

```
SEEDS="2c8c6a233351c22f3cd0151a14fef55dd3dd3d87@rpc.defund-testnet.mirror-reflection.com:26656,3f472746f46493309650e5a033076689996c8881@defund-testnet.rpc.kjnodes.com:40659"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.defund/config/config.toml
```

### Set pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.defund/config/app.toml
```

### Set minimum gas price

```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0ufetf\"|" $HOME/.defund/config/app.toml
```

### Resetting data

```
defundd tendermint unsafe-reset-all --home $HOME/.defund --keep-addr-book
```

### Create a service

```
sudo tee /etc/systemd/system/defundd.service > /dev/null << EOF
[Unit]
Description=defund-testnet node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which defundd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### Download snapshot

```
curl -L https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/defund-testnet/orbit-alpha-1_latest.tar | tar -xf - -C $HOME/.defund/data
```

### Start service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable defundd
sudo systemctl start defundd
sudo journalctl -u defundd -f --no-hostname -o cat
```

<div align="center">
  <h1> Management </h1>
</div>

When node is synced you must see **FALSE** after command

```
defundd status 2>&1 | jq .SyncInfo.catching_up
```

If you see **FALSE**

### Create wallet

```
defundd keys add wallet
```

SAVE YOUR INPUT SEED PHRASE (24 words)

### SAVE PRIVATE VALIDATOR KEY

```
cat $HOME/.defund/config/priv_validator_key.json
```

### Go to [discord channel](https://discord.gg/cw5N3P5z5M) #faucet and paste

```
!faucet YOUR_WALLET_ADDRESS
```

### Create Validator

```
defundd tx staking create-validator \
--amount=10000000ufetf \
--pubkey=$(defundd tendermint show-validator) \
--moniker="$NODE_MONIKER" \
--chain-id=orbit-alpha-1 \
--commission-rate=0.07 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices=0ufetf \
-y
```

### View validator details

```
defundd q staking validator $(defundd keys show wallet --bech val -a)
```
