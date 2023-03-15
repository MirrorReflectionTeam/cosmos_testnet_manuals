<div align="center">
  <h1> Manual Installation </h1>
</div>

### DeFund

<figure><img src="https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_testnet_manuals/main/project_files/defund.jpg" width="150" alt=""><figcaption></figcaption></figure>

**[Website](https://www.defund.app/)** | **[Explorer](https://defund.explorers.guru/)**

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
rm -rf defund
git clone https://github.com/defund-labs/defund.git
cd defund
git checkout v0.2.4
make install
```

### Initialize the node

```
defundd config chain-id defund-private-4
defundd init "$NODE_MONIKER" --chain-id defund-private-4
```

### Download genesis and addrbook

```
curl -Ls https://snapshots.kjnodes.com/defund-testnet/genesis.json > $HOME/.defund/config/genesis.json
curl -Ls https://snapshots.kjnodes.com/defund-testnet/addrbook.json > $HOME/.defund/config/addrbook.json
```

### Add seeds

```
SEEDS="d837b7f78c03899d8964351fb95c78e84128dff6@174.83.6.129:30791,f03f3a18bae28f2099648b1c8b1eadf3323cf741@162.55.211.136:26656"
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
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001ufetf"|g' $HOME/.defund/config/app.toml
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
curl "https://snapshots2-testnet.nodejumper.io/defund-testnet/${SNAP_NAME}" | lz4 -dc - | tar -xf - -C "$HOME/.defund"
```

### Start service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable defundd
sudo systemctl start defundd
sudo journalctl -u defundd -f --no-hostname -o cat
```

### Management

When node is synced you must see **FALSE** after command

```
curl -s localhost:26657/status | jq .result.sync_info.catching_up
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
--chain-id=defund-private-4 \
--commission-rate=0.07 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices=0.1ufetf \
-y
```

### View validator details

```
defundd q staking validator $(defundd keys show wallet --bech val -a)
```
