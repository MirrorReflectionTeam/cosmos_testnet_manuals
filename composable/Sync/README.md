<div align="center">
  <h2> Snapshot </h2>
</div>

> #### Snapshots allow the blockchain node to join the network in a matter of minutes by downloading our file which is a backup copy of the node. A snapshot contains a compressed copy of the date directory.
>
> #### Snapshots are taken automatically every 4 hours

```
sudo systemctl stop banksyd

cp $HOME/.banksy/data/priv_validator_state.json $HOME/.banksy/priv_validator_state.json.backup 
banksyd tendermint unsafe-reset-all --home $HOME/.banksy --keep-addr-book 

curl -L https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/composable-testnet/banksy-testnet-2_latest.tar | tar -xf - -C $HOME/.banksy/data

mv $HOME/.banksy/priv_validator_state.json.backup $HOME/.banksy/data/priv_validator_state.json 

sudo systemctl restart banksyd
sudo journalctl -u banksyd -f --no-hostname -o cat
```

<div align="center">
  <h2> State Sync </h2>
</div>

> #### State Sync allows a new node to join the network without completely downloading the entire blockchain node. This helps with server memory shortage. State Sync can cut down time to synchronization from days to minutes.

```
sudo systemctl stop banksyd

cp $HOME/.banksy/data/priv_validator_state.json $HOME/.banksy/priv_validator_state.json.backup
banksyd tendermint unsafe-reset-all --home $HOME/.banksy --keep-addr-book

SNAP_RPC="https://rpc.composable-testnet.mirror-reflection.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="a35221ab84e836c05d5a4689c78322833523233f@rpc.composable-testnet.mirror-reflection.com:32656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.banksy/config/config.toml

sed -i 's|^enable *=.*|enable = true|' $HOME/.banksy/config/config.toml
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $HOME/.banksy/config/config.toml
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $HOME/.banksy/config/config.toml
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $HOME/.banksy/config/config.toml

mv $HOME/.banksy/priv_validator_state.json.backup $HOME/.banksy/data/priv_validator_state.json

sudo systemctl restart banksyd
sudo journalctl -u banksyd -f --no-hostname -o cat
```

<div align="center">
  <h2> Live Peers </h2>
</div>

```
PEERS="a35221ab84e836c05d5a4689c78322833523233f@rpc.composable-testnet.mirror-reflection.com:32656,5070c358e90f8005b2049ff2e6d8e95adeb9bdd3@144.76.182.73:40656,a70328c945ac6109503d1f656b2751bc5cda5178@167.114.172.204:15956,8859e665f2eca25da78aaf4d2e541407885b08d8@5.78.72.11:26656,a1bbf456dffa2d23bb9d524382f2f3c8e28a470e@34.142.142.40:26656,234fa34c415ebd65ede285f4098dcd6e762b0882@65.108.230.113:21206,7feb85b27b47c544e757838feca8eb8f382b4274@213.133.103.188:26656,3ba2a08d08adc31c3bce41c556946d052f904cb3@95.214.53.218:10656,367de894f877be9a9592f9d506c3082798b603e9@148.251.82.189:40656,c16bc22759633af69b6f698840cf2ba4d80ad7f9@157.245.154.125:15956,60103909868ef756ebd9f573e091c93b43be6ee5@193.36.132.185:26656,48246fed451b0d553239a1a0ba13d84977584be9@142.132.207.187:26666,7fc16efbb3e56d81245a0828198d580b3f246f58@51.91.30.173:3000,7521d65a4102259fa26816383fea2f8f21a3b1ea@65.109.116.21:11154,156d57dfe94634eaba1c30f9ec2ce5ccee8410e1@65.21.88.12:2000,2ca32b1aba0208008738ddefe44d5239bef2e894@95.217.144.107:22256,068fbf2425c5f024986444f1388fc1cabff3d733@46.4.5.45:22256,5c2a752c9b1952dbed075c56c600c3a79b58c395@185.16.39.172:26976"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.banksy/config/config.toml
```

<div align="center">
  <h2> Address Book </h2>
</div>

```
curl -s https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/composable-testnet/addrbook.json > $HOME/.banksy/config/addrbook.json
```