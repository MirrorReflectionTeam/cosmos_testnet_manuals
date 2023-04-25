<div align="center">
  <h2> Snapshot </h2>
</div>

> #### Snapshots allow the blockchain node to join the network in a matter of minutes by downloading our file which is a backup copy of the node. A snapshot contains a compressed copy of the date directory.
>
> #### Snapshots are taken automatically every 4 hours

```
sudo systemctl stop babylond

cp $HOME/.babylond/data/priv_validator_state.json $HOME/.babylond/priv_validator_state.json.backup 

babylond tendermint unsafe-reset-all --home $HOME/.babylond --keep-addr-book 
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/babylon-testnet/bbn-test1_latest.tar | tar -xf - -C $HOME/.babylond/data

mv $HOME/.babylond/priv_validator_state.json.backup $HOME/.babylond/data/priv_validator_state.json 

sudo systemctl restart babylond
sudo journalctl -u babylond -f --no-hostname -o cat
```

<div align="center">
  <h2> State Sync </h2>
</div>

> #### State Sync allows a new node to join the network without completely downloading the entire blockchain node. This helps with server memory shortage. State Sync can cut down time to synchronization from days to minutes.

```
sudo systemctl stop babylond

cp $HOME/.babylond/data/priv_validator_state.json $HOME/.babylond/priv_validator_state.json.backup
babylond tendermint unsafe-reset-all --home $HOME/.babylond --keep-addr-book

SNAP_RPC="https://rpc.babylon-testnet.mirror-reflection.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="465cd1d6ada4a6e9643b64d9c2f7f0afcea02685@rpc.babylon-testnet.mirror-reflection.com:34656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.babylond/config/config.toml

sed -i 's|^enable *=.*|enable = true|' $HOME/.babylond/config/config.toml
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $HOME/.babylond/config/config.toml
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $HOME/.babylond/config/config.toml
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $HOME/.babylond/config/config.toml

mv $HOME/.babylond/priv_validator_state.json.backup $HOME/.babylond/data/priv_validator_state.json

sudo systemctl restart babylond
sudo journalctl -u babylond -f --no-hostname -o cat
```

<div align="center">
  <h2> Live Peers </h2>
</div>

```
PEERS="465cd1d6ada4a6e9643b64d9c2f7f0afcea02685@rpc.babylon-testnet.mirror-reflection.com:34656,88bed747abef320552d84d02947d0dd2b6d9c71c@babylon-testnet.nodejumper.io:44656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.babylond/config/config.toml
```

<div align="center">
  <h2> Address Book </h2>
</div>

```
curl -s https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/babylon-testnet/addrbook.json > $HOME/.babylond/config/addrbook.json
```