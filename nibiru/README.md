<div align="center">
  <h1> Manual Installation </h1>
</div>

### Nibiru

<figure><img src="https://github.com/MirrorReflectionTeam/cosmos_testnet_manuals/blob/main/project_files/nibiru.jpg" width="150" alt=""><figcaption></figcaption></figure>

**[Website](https://nibiru.fi/)** | **[Explorer](https://nibiru.explorers.guru/)**

### Public endpoints

- **https://rpc.nibiru-testnet.mirror-reflection.com**

- **https://grpc.nibiru-testnet.mirror-reflection.com**

- **https://api.nibiru-testnet.mirror-reflection.com**

### Hardware Requirements

8CPU 32RAM 200GB

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
rm -rf nibiru
git clone https://github.com/NibiruChain/nibiru.git
cd nibiru
git checkout v0.19.2
make install
nibid version
```

### Initialize the node

```
nibid config chain-id nibiru-itn-1
nibid init "$NODE_MONIKER" --chain-id nibiru-itn-1
```

### Download genesis and addrbook

```
curl -s https:/rpc.nibiru-testnet.mirror-reflection.com/genesis | jq -r .result.genesis > $HOME/.nibid/config/genesis.json
curl -s https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/nibiru-testnet/addrbook.json > $HOME/.nibid/config/addrbook.json
```

### Add seeds

```
SEEDS="df8596fa04abeff1d15b79570ff8c3eba85ed87a@35.185.8.9:26656,4a81486786a7c744691dc500360efcdaf22f0840@15.235.46.50:26656,c709cad9e11b315644fe8f1d2e90c03c5cba685c@34.91.8.241:26656,930b1eb3f0e57b97574ed44cb53b69fb65722786@144.76.30.36:15662,ad002a4592e7bcdfff31eedd8cee7763b39601e7@65.109.122.105:36656"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.nibid/config/config.toml
```

### Set pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.nibid/config/app.toml
```

### Set minimum gas price

```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025unibi"|g' $HOME/.nibid/config/app.toml
```

### Resetting data

```
nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book
```

### Create a service

```
sudo tee /etc/systemd/system/nibid.service > /dev/null << EOF
[Unit]
Description=Nibiru Node
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
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/nibiru-testnet/nibiru-itn-1_latest.tar | tar -xf - -C $HOME/.nibid/data
```

### Start service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable nibid
sudo systemctl start nibid
sudo journalctl -u nibid -f --no-hostname -o cat
```

### Management

When node is synced you must see **FALSE** after command

```
nibid status 2>&1 | jq .SyncInfo.catching_up
```

If you see **FALSE**

### Create wallet

```
nibid keys add wallet
```

SAVE YOUR INPUT SEED PHRASE (24 words)

### SAVE PRIVATE VALIDATOR KEY

```
cat $HOME/.nibid/config/priv_validator_key.json
```

### Go to [discord channel](https://discord.gg/nibiru) #faucet and paste

```
!request YOUR_WALLET_ADDRESS
```

**or go https://app.nibiru.fi/faucet**

### Create Validator

```
nibid tx staking create-validator \
--amount=10000000unibi \
--pubkey=$(nibid tendermint show-validator) \
--moniker="$NODE_MONIKER" \
--chain-id=nibiru-itn-1 \
--commission-rate=0.07 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment=1.4 \
--gas=auto \
--gas-prices=0.025unibi \
-y
```

### View validator details

```
nibid q staking validator $(nibid keys show wallet --bech val -a)
```
