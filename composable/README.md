### Composable

<figure><img src="https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_testnet_manuals/main/project_files/composable.jpg" width="150" alt=""><figcaption></figcaption></figure>

**[Website](https://www.composable.finance/)** | **[Explorer](https://composable.exploreme.pro/)**

### Public endpoints

- **https://rpc.composable-testnet.mirror-reflection.com**

- **https://grpc.composable-testnet.mirror-reflection.com**

- **https://api.composable-testnet.mirror-reflection.com**


### Hardware Requirements

8CPU 16RAM 200GB

**[Rent on Hetzner](https://hetzner.cloud/?ref=AwVksaI2T3Nz)** | **[Rent on Contabo](https://contabo.com/en)**

<div align="center">
  <h1> Automatic Installation </h1>
</div>

```
source <(curl -s https://raw.githubusercontent.com/MirrorReflectionTeam/cosmos_scripts/main/composable/install.sh)
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
rm -rf composable-testnet
git clone https://github.com/notional-labs/composable-testnet.git
cd composable-testnet
git checkout v2.3.3-testnet2fork
make install
banksyd version
```

### Initialize the node

```
banksyd config chain-id banksy-testnet-2
banksyd init "$NODE_MONIKER" --chain-id banksy-testnet-2
```

### Download genesis and addrbook

```
curl -Ls https://rpc.composable-testnet.mirror-reflection.com/genesis | jq -r .result.genesis > $HOME/.banksy/config/genesis.json
curl -s https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/composable-testnet/addrbook.json > $HOME/.banksy/config/addrbook.json
```

### Add seeds

```
SEEDS="a35221ab84e836c05d5a4689c78322833523233f@rpc.composable-testnet.mirror-reflection.com:32656,3f472746f46493309650e5a033076689996c8881@composable-testnet.rpc.kjnodes.com:15959"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.banksy/config/config.toml
```

### Set pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
  $HOME/.banksy/config/app.toml
```

### Set minimum gas price

```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0ppica\"|" $HOME/.banksy/config/app.toml
```

### Resetting data

```
banksyd tendermint unsafe-reset-all --home $HOME/.banksy --keep-addr-book
```

### Create a service

```
sudo tee /etc/systemd/system/banksyd.service > /dev/null << EOF
[Unit]
Description=Composable Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which banksyd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
WorkingDirectory=$HOME
[Install]
WantedBy=multi-user.target
EOF
```

### Download snapshot

```
curl -L https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/composable-testnet/banksy-testnet-2_latest.tar | tar -xf - -C $HOME/.banksy/data
```

### Start service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable banksyd
sudo systemctl start banksyd
sudo journalctl -u banksyd -f --no-hostname -o cat
```

<div align="center">
  <h1> Management </h1>
</div>

When node is synced you must see **FALSE** after command

```
banksyd status 2>&1 | jq .SyncInfo.catching_up
```

If you see **FALSE**

### Create wallet

```
banksyd keys add wallet
```

SAVE YOUR INPUT SEED PHRASE (24 words)

### SAVE PRIVATE VALIDATOR KEY

```
cat $HOME/.banksy/config/priv_validator_key.json
```

### Go to [discord channel](https://discord.gg/composable) #get validator-role and go to testnet-3-faucet

```
$request YOUR_WALLET_ADDRESS
```

### Create Validator

```
banksyd tx staking create-validator \
--amount=1000000upica \
--pubkey=$(banksyd tendermint show-validator) \
--moniker="$NODE_MONIKER" \
--chain-id=banksy-testnet-2 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment=1.5 \
--gas=auto \
--gas-prices=0.1upica \
-y
```

### View validator details

```
banksyd q staking validator $(banksyd keys show wallet --bech val -a) 
```
