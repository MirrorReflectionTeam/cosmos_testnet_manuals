<div align="center">
  <h2> Snapshot </h2>
</div>

> #### Snapshots allow the blockchain node to join the network in a matter of minutes by downloading our file which is a backup copy of the node. A snapshot contains a compressed copy of the date directory.
>
> #### Snapshots are taken automatically every 4 hours

```
sudo systemctl stop andromedad

cp $HOME/.andromedad/data/priv_validator_state.json $HOME/.andromedad/priv_validator_state.json.backup

andromedad tendermint unsafe-reset-all --home $HOME/.andromedad --keep-addr-book
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/andromeda-testnet/galileo-3_latest.tar | tar -xf - -C $HOME/.andromedad/data

mv $HOME/.andromedad/priv_validator_state.json.backup $HOME/.andromedad/data/priv_validator_state.json

sudo systemctl start andromedad
sudo journalctl -u andromedad -f --no-hostname -o cat
```

<div align="center">
  <h2> State Sync </h2>
</div>

> #### State Sync allows a new node to join the network without completely downloading the entire blockchain node. This helps with server memory shortage. State Sync can cut down time to synchronization from days to minutes.

```
sudo systemctl stop andromedad

cp $HOME/.andromedad/data/priv_validator_state.json $HOME/.andromedad/priv_validator_state.json.backup
andromedad tendermint unsafe-reset-all --home $HOME/.andromedad

SNAP_RPC="https://rpc.andromeda-testnet.mirror-reflection.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="b9a90655c4235894fe6e1dda4f6697e69648e942@rpc.andromeda-testnet.mirror-reflection.com:35656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.andromedad/config/config.toml

sed -i 's|^enable *=.*|enable = true|' $HOME/.andromedad/config/config.toml
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $HOME/.andromedad/config/config.toml
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $HOME/.andromedad/config/config.toml
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $HOME/.andromedad/config/config.toml

mv $HOME/.andromedad/priv_validator_state.json.backup $HOME/.andromedad/data/priv_validator_state.json

curl -L https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/andromeda-testnet/wasm_galileo-3_.tar  | tar -xf - -C $HOME/.andromedad

sudo systemctl restart andromedad
sudo journalctl -u andromedad -f --no-hostname -o cat
```

<div align="center">
  <h2> Live Peers </h2>
</div>

```
PEERS="b9a90655c4235894fe6e1dda4f6697e69648e942@rpc.andromeda-testnet.mirror-reflection.com:35656,dff203d0633c98eea4a228c5e913f22236043d89@23.88.69.101:16656,3f9594221efe3e9cd4d0de31f71993fc0f12bf01@65.21.245.252:26656,b594f01b5b49a11b6d2e97c3b6358dc1388a1039@65.108.108.52:26656,fc1c12503b0fd8dfcef4a9ccd0af7a26f9d0738f@51.91.153.78:32705,e5a2bcbcea7d3b2ea7d7feb75da1c115125b665f@65.109.112.178:31656,433cc64756cb7f00b5fb4b26de97dc0db72b27ca@65.108.216.219:6656,d5519e378247dfb61dfe90652d1fe3e2b3005a5b@65.109.68.190:47656,b9836aff6d8e79b9a04b4a2a80d6007bf33a526b@198.244.179.125:32069"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.andromedad/config/config.toml
```

<div align="center">
  <h2> Address Book </h2>
</div>

```
curl -s https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/andromeda-testnet/addrbook.json > $HOME/.andromedad/config/addrbook.json
```

<div align="center">
  <h2> Wasm </h2>
</div>

```
curl -L https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/andromeda-testnet/wasm_galileo-3_.tar  | tar -xf - -C $HOME/.andromedad
```
