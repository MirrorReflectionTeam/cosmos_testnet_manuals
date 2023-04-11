<div align="center">
  <h2> Snapshot </h2>
</div>

> #### Snapshots allow the blockchain node to join the network in a matter of minutes by downloading our file which is a backup copy of the node. A snapshot contains a compressed copy of the date directory.
>
> #### Snapshots are taken automatically every 4 hours

```
sudo systemctl stop uptickd

cp $HOME/.uptickd/data/priv_validator_state.json $HOME/.uptickd/priv_validator_state.json.backup 

uptickd tendermint unsafe-reset-all --home $HOME/.uptickd --keep-addr-book 
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/uptick-testnet/uptick_117-1_latest.tar | tar -xf - -C $HOME/.ojo/data

mv $HOME/.uptickd/priv_validator_state.json.backup $HOME/.uptickd/data/priv_validator_state.json 

sudo systemctl restart uptickd
sudo journalctl -u uptickd -f --no-hostname -o cat
```

<div align="center">
  <h2> State Sync </h2>
</div>

> #### State Sync allows a new node to join the network without completely downloading the entire blockchain node. This helps with server memory shortage. State Sync can cut down time to synchronization from days to minutes.

```
sudo systemctl stop uptickd

cp $HOME/.uptickd/data/priv_validator_state.json $HOME/.uptickd/priv_validator_state.json.backup
uptickd tendermint unsafe-reset-all --home $HOME/.uptickd --keep-addr-book

SNAP_RPC="https://rpc.uptick-testnet.mirror-reflection.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="e3b455f295443122f51c3a98a2e92168e402bcc5@rpc.uptick-testnet.mirror-reflection.com:32656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.uptickd/config/config.toml

sed -i 's|^enable *=.*|enable = true|' $HOME/.uptickd/config/config.toml
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $HOME/.uptickd/config/config.toml
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $HOME/.uptickd/config/config.toml
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $HOME/.uptickd/config/config.toml

mv $HOME/.uptickd/priv_validator_state.json.backup $HOME/.uptickd/data/priv_validator_state.json

sudo systemctl restart uptickd
sudo journalctl -u uptickd -f --no-hostname -o cat
```

<div align="center">
  <h2> Live Peers </h2>
</div>

```

PEERS="e3b455f295443122f51c3a98a2e92168e402bcc5@rpc.uptick-testnet.mirror-reflection.com:32656,8f6fbc1a1119f5827e1768aca3577724460fb61f@157.90.213.40:26656,58cf2af0e94d7c55473a1e98225a6ff25baa0402@65.21.4.10:15656,a818920590d15226a206ec4c73b1c5c20c56a435@65.21.134.202:26666,174a57a0d4b914b5a9823a5f3f47ae4b06d9809e@65.108.206.118:60956,e9b37cb6a5743ca1793af119f53b91cf5892fb45@65.109.88.251:34656,72aa8a613e563e85e8a975fa121440487a9b6e05@65.109.32.174:29656,1266d32b49d7472934028ed09454ebae1c7ce09e@65.108.71.80:26656,7831b5c5cc90fa95ea99a0cea5d1ad07dfcc7b9c@185.245.183.187:26656,a3b3712dfd366c5c39f6a6b3265c88c4166da86a@161.97.93.245:26661,99a47965735ea33dc6677efb3b62bb6476661b92@185.144.99.86:26656,3cffe20d473b0bd4451d330da8b741b5d42dcb44@65.21.131.215:26666,8ed9ffbd365e360804c6140e4906a5263c5b608a@116.203.157.163:10656,9e51f3ab29137b4b1c8e0855e5bc1dc8d4f8d2b5@65.21.238.219:26656,a489dcbd4c5b7ef20d77c51dba217e85c631f463@65.108.105.48:20456,b9e0210809b9dfc9cd299c6e83116d7fa45c6e27@65.109.68.93:46656,1bb6d67af0dd1d452e294e9df430d07bccefe502@185.215.167.241:26656,3689cef89c3d87c32a1561b931af5ddd59328f5e@65.109.58.237:36656,0afb5ce897e69eec34fb32bf87f4a2f93f79e0b3@65.109.65.210:30656,bd486ff0635581c0680e28e93453ba8a26fc5fa8@181.214.147.81:10656,7849e4320385434b0828a3e0206a3b69767393f6@65.109.91.227:26656,f58fd7ff25183e7e0dc3c35e667641129a8bc2cd@144.76.27.79:26656,a9bb3d5c36cf62a280c13f3e37c93a4b17707eab@142.132.196.251:46656,45f58ce671967a10933ea3e2279be03f0ebcb42c@85.114.134.219:16656,7a4f1c0baa2ff31c02163fb658c4eb8d119193c7@95.214.52.173:18656,e9fee55fdf6668e4e04927cdd85bbbbc9e9e43b1@209.145.62.101:26656,aff8d7b78840eaafa6c2bafd9a76b76e565b2933@65.108.131.190:25256,52cdb51fe8692dea11de23b8c97c9d947a6eb1c2@51.222.44.116:10656,af5262526a0800a29a0a7194e1488a9fa62d0005@195.3.223.208:26656,d8777278648d8fc93800692a8b96a7f104df4f9a@194.163.135.127:26656,bf626ac1b0c733c0937d70d8c834c94c3e4b9033@65.108.129.29:14656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.uptickd/config/config.toml
```

<div align="center">
  <h2> Address Book </h2>
</div>

```
curl -Ls https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/uptick-testnet/addrbook.json > $HOME/.uptickd/config/addrbook.json
```