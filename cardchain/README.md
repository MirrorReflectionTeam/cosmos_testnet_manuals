### Cardchain

<figure><img src="https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_testnet_manuals/main/project_files/cardchain.png" width="150" alt=""><figcaption></figcaption></figure>

**[Website](https://crowdcontrol.network/#/)** | **[Explorer](https://explorer.stavr.tech/cardchain)**

### Public endpoints

- **https://rpc.cardchain-testnet.mirror-reflection.com**

- **https://grpc.cardchain-testnet.mirror-reflection.com**

- **https://api.cardchain-testnet.mirror-reflection.com**

### Hardware Requirements

4CPU 16RAM 200GB

**[Rent on Hetzner](https://hetzner.cloud/?ref=AwVksaI2T3Nz)** | **[Rent on Contabo](https://contabo.com/en)**

<div align="center">
  <h1> Automatic Installation </h1>
</div>

```
source <(curl -s https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_scripts/main/cardchain/install.sh)
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
cd || return
curl -L https://github.com/DecentralCardGame/Cardchain/releases/download/v0.81/Cardchain_latest_linux_amd64.tar.gz > Cardchain_latest_linux_amd64.tar.gz
tar -xvzf Cardchain_latest_linux_amd64.tar.gz
chmod +x Cardchaind
mkdir -p $HOME/go/bin
mv Cardchaind $HOME/go/bin
rm Cardchain_latest_linux_amd64.tar.gz
```

### Initialize the node

```
Cardchaind config chain-id testnet3
Cardchaind init "$NODE_MONIKER" --chain-id testnet3
```

### Download genesis and addrbook

```
curl -s https://raw.githubusercontent.com/DecentralCardGame/Testnet/main/genesis.json > $HOME/.Cardchain/config/genesis.json
curl -s https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/cardchain-testnet/addrbook.json > $HOME/.Cardchain/config/addrbook.json
```

### Add seeds

```
SEEDS=""
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.Cardchain/config/config.toml
```

### Set pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.Cardchain/config/app.toml
```

### Set minimum gas price

```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001ubpf"|g' $HOME/.Cardchain/config/app.toml
```

### Resetting data

```
Cardchaind unsafe-reset-all
```

### Create a service

```
sudo tee /etc/systemd/system/Cardchaind.service > /dev/null << EOF
[Unit]
Description=Cardchain Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which Cardchaind) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```

### Download snapshot

```
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/cardchain-testnet/Testnet3_latest.tar | tar -xf - -C $HOME/.Cardchain/data
curl -s https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/cardchain-testnet/addrbook.json > $HOME/.Cardchain/config/addrbook.json
```

### Start service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable Cardchaind
sudo systemctl start Cardchaind
sudo journalctl -u Cardchaind -f --no-hostname -o cat
```

<div align="center">
  <h1> Management </h1>
</div>

When node is synced you must see **FALSE** after command

```
Cardchaind status 2>&1 | jq .SyncInfo.catching_up
```

If you see **FALSE**

### Create wallet

```
Cardchaind keys add wallet
```

SAVE YOUR INPUT SEED PHRASE (24 words)

### SAVE PRIVATE VALIDATOR KEY

```
cat $HOME/.Cardchain/config/priv_validator_key.json
```

### Go to [website](https://crowdcontrol.network/#/about) and import your wallet address using seed phase above
### Verify your balance
```
Cardchaind q bank balances $(Cardchaind keys show wallet -a)
```

### Create Validator

```
Cardchaind tx staking create-validator \
--amount=4000000ubpf \
--pubkey=$(Cardchaind tendermint show-validator) \
--moniker="$NODE_MONIKER" \
--chain-id=Testnet3 \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--from=wallet \
-y
```

### View validator details

```
Cardchaind q staking validator $(Cardchaind keys show wallet --bech val -a)
```
