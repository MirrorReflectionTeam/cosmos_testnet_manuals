<div align="center">
  <h2> Snapshot </h2>
</div>

> #### Snapshots allow the blockchain node to join the network in a matter of minutes by downloading our file which is a backup copy of the node. A snapshot contains a compressed copy of the date directory.
>
> #### Snapshots are taken automatically every 4 hours

```
sudo systemctl stop Cardchaind

cp $HOME/.Cardchain/data/priv_validator_state.json $HOME/.Cardchain/priv_validator_state.json.backup 

Cardchaind tendermint unsafe-reset-all --home $HOME/.Cardchain --keep-addr-book 
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/cardchain-testnet/Testnet3_latest.tar | tar -xf - -C $HOME/.Cardchain/data

mv $HOME/.Cardchain/priv_validator_state.json.backup $HOME/.Cardchain/data/priv_validator_state.json 

sudo systemctl restart Cardchaind
sudo journalctl -u Cardchaind -f --no-hostname -o cat
```

<div align="center">
  <h2> State Sync </h2>
</div>

> #### State Sync allows a new node to join the network without completely downloading the entire blockchain node. This helps with server memory shortage. State Sync can cut down time to synchronization from days to minutes.

```
sudo systemctl stop Cardchaind

cp $HOME/.Cardchain/data/priv_validator_state.json $HOME/.Cardchain/priv_validator_state.json.backup
Cardchaind tendermint unsafe-reset-all --home $HOME/.Cardchain --keep-addr-book

SNAP_RPC="https://rpc.cardchain-testnet.mirror-reflection.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="9b8887d5d7be3d41fdcc5e797cb9bd30f0465bea@rpc.cardchain-testnet.mirror-reflection.com:33656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.Cardchain/config/config.toml

sed -i 's|^enable *=.*|enable = true|' $HOME/.Cardchain/config/config.toml
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $HOME/.Cardchain/config/config.toml
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $HOME/.Cardchain/config/config.toml
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $HOME/.Cardchain/config/config.toml

mv $HOME/.Cardchain/priv_validator_state.json.backup $HOME/.Cardchain/data/priv_validator_state.json

sudo systemctl restart Cardchaind
sudo journalctl -u Cardchaind -f --no-hostname -o cat
```

<div align="center">
  <h2> Live Peers </h2>
</div>

```
PEERS="9b8887d5d7be3d41fdcc5e797cb9bd30f0465bea@rpc.cardchain-testnet.mirror-reflection.com:33656,6bf0b58306de698ebea190078cb4a4d92407bfa7@51.222.248.153:36656,d5fbf52331f8a8851557cd0eabf444850cef5646@135.181.133.16:26656,539122b48938fc6fb0b3adf76af03b285d8abf95@144.76.97.251:26786,345d8149cbda76ae42142a78df7bbc90f5fc26f2@149.102.142.176:26656,77b7f84c6cabda94b570224b29acbf72bfe63b00@65.109.144.236:26656,e6df085fdb0a52b98e4bbb20056fcccd17bdeb24@149.102.142.179:26656,b0c3fc593b073adf13d624f259ec3fc19342b6da@149.102.142.178:26656,dbeb4731d1571584e94eac33be3c76dd1192954a@149.102.142.175:26656,e1dcb3c72d7f6e6eb56d5e9fee32f95603e17253@209.126.81.240:26646,404206a21ff907422ab4443bea5cb3c8b974dea2@149.102.142.177:26656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.Cardchain/config/config.toml
```

<div align="center">
  <h2> Address Book </h2>
</div>

```
curl -s https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/cardchain-testnet/addrbook.json > $HOME/.Cardchain/config/addrbook.json
```
