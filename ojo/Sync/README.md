<div align="center">
  <h2> Snapshot </h2>
</div>

> #### Snapshots allow the blockchain node to join the network in a matter of minutes by downloading our file which is a backup copy of the node. A snapshot contains a compressed copy of the date directory.
>
> #### Snapshots are taken automatically every 4 hours

```
sudo systemctl stop ojod

cp $HOME/.ojo/data/priv_validator_state.json $HOME/.ojo/priv_validator_state.json.backup

ojod tendermint unsafe-reset-all --home $HOME/.ojo --keep-addr-book
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/ojo-testnet/ojo-devnet_latest.tar | tar -xf - -C $HOME/.ojo/data

mv $HOME/.ojo/priv_validator_state.json.backup $HOME/.ojo/data/priv_validator_state.json

sudo systemctl start ojod
sudo journalctl -u ojod -f --no-hostname -o cat
```

<div align="center">
  <h2> State Sync </h2>
</div>

> #### State Sync allows a new node to join the network without completely downloading the entire blockchain node. This helps with server memory shortage. State Sync can cut down time to synchronization from days to minutes.

```
sudo systemctl stop ojod

cp $HOME/.ojo/data/priv_validator_state.json $HOME/.ojo/priv_validator_state.json.backup
ojod tendermint unsafe-reset-all --home $HOME/.ojo --keep-addr-book

SNAP_RPC="https://rpc.ojo-testnet.mirror-reflection.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="b314955720069e8c98acf1cf1e896b68a3e306f9@rpc.ojo-testnet.mirror-reflection.com:27656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.ojo/config/config.toml

sed -i 's|^enable *=.*|enable = true|' $HOME/.ojo/config/config.toml
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $HOME/.ojo/config/config.toml
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $HOME/.ojo/config/config.toml
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $HOME/.ojo/config/config.toml

mv $HOME/.ojo/priv_validator_state.json.backup $HOME/.ojo/data/priv_validator_state.json

sudo systemctl restart ojod
sudo journalctl -u ojod -f --no-hostname -o cat
```

<div align="center">
  <h2> Live Peers </h2>
</div>

```
PEERS="b314955720069e8c98acf1cf1e896b68a3e306f9@rpc.ojo-testnet.mirror-reflection.com:27656,239caa37cb0f131b01be8151631b649dc700cd97@95.217.200.36:46656,d9df87e2e26db62ef4014ce6e8705ee11bda304f@176.124.220.21:4669,2c40b0aedc41b7c1b20c7c243dd5edd698428c41@138.201.85.176:26696,7d6706d7ee674e2b2c38d3eb47d85ec6e376c377@49.12.123.87:56656,9ebe723eef929e9eff748f4046d6130ee349a398@65.108.203.149:24017,8fbfa810cb666ddef1c9f4405e933ef49138f35a@65.108.199.120:54656,0d4dc8d9e80df99fdf7fbb0e44fbe55e0f8dde28@65.108.205.47:14756,b6c75d1fbdc9c39daaaf52a4c0937b9f06975808@167.235.198.193:46656,7bf4e4a18bf2006f79f50c79903f77d4e2a5a303@65.21.77.175:33307,3147dcfa5c320f31b083a1d1319cbcb010108f9f@65.108.233.220:17656,bdd24cab3246503ae261aea82f077ffb66d56ce3@95.216.39.183:28656,4640b6c775c05b6146a708a3b5ec2241c1688588@161.97.147.255:50656,406466cf9c390c0fd1784a197f2bb38a786da35e@185.111.159.115:28656,9bcec17faba1b8f6583d37103f20bd9b968ac857@38.146.3.230:21656,97a388be825fc69fca40a8a3de75aa5794602abb@95.217.225.212:36656,2720b3b9e1a318d5b4abdad65714c55e09f43965@82.208.21.78:36656,f702b19a4dae5ad813dabe3f529bf31c160a74e0@5.189.176.202:26656,f474a520009496972515f843cdb835fc7d663779@65.109.23.114:21656,d5b2ae8815b09a30ab253957f7eca052dde3101d@65.108.9.164:24656,a3a9014f82cb69fe0494ea3bc49990027d081a5a@65.108.126.35:36656,21f35146e8905d6786b69486b1d27b680fb3c548@65.109.89.5:40656,da369d44c00dba309237b21391806504353d188f@194.163.187.175:50656,c37e444f67af17545393ad16930cd68dc7e3fd08@95.216.7.169:61156,9ea0473b3684dbf1f2cf194f69f746566dab6760@78.46.99.50:22656,394430197ba609d954922303b33c22fd2ad1d8dc@210.211.108.12:26656,9dc1f555bd37d6840237f32a2cd4d79ba1c80cb5@65.108.227.112:31656,0ac9841750afe017b882768b0e29e72b8296d6b0@104.194.8.68:46656,d5519e378247dfb61dfe90652d1fe3e2b3005a5b@65.109.68.190:50656,4cb932af43e2c64a0277516d96410a05294653de@75.119.148.69:26656,67c653cec3e0e116939841b9c601b43daecae47e@116.202.170.159:24656,ae3621c022cddc8c05d7640c14147d257746fb74@185.215.166.73:26656,1b5c5927e6e3685b3e9fc278ca4c9d7002d4cc10@65.21.134.250:17656,855fc154f9054ce4055719e09ce6f7f1d0ecd9fb@85.10.198.171:36656,18300f0a5973798c3900fe51ff255bb6bca982f9@65.109.65.248:36656,5d9be9cf3d5161e4891b96a956c3c83de6c0ae49@5.78.75.124:26656,d97bd8a6ca6766efd8f5a5fb5af96a92a4f2fe4e@195.3.222.189:26656,2cd65afacab69b6ebfa4f9a8435f5f7df5fbb735@135.181.160.61:32656,5a4201370808de8fe5926db82767d8be44c9d288@51.222.42.89:50656,4e007fe2793172797eff893abf91ab685549ee11@65.109.235.2:26656,da9e028814ff30ec24e94bec6887f4686f692b86@173.212.222.167:30656,50ad0e558d9da6fce98ae4527cd49ee3e8d19940@94.250.202.215:26656,cb706ebe1d7a1f1d3e281bf46a78d84251f50810@95.216.14.72:26656,8e2ea63e2ff7ecbe75aef6012c4df050d5e1de0f@65.21.139.155:28656,d1c5c6bf4641d1800e931af6858275f08c20706d@23.88.5.169:18656,98981d7eef057a01274473363addb7f0b17e06fa@84.21.171.25:26656,bd90b71f1f982ebb18857da8cb777883d6ca687e@185.209.223.68:26656,7186f24ace7f4f2606f56f750c2684d387dc39ac@65.108.231.124:12656,b6b4a4c720c4b4a191f0c5583cc298b545c330df@65.109.28.219:21656,124439d1c16b1ee7ca1a39961f02fadf8539cb81@38.102.85.10:26656,b133dde2713a216a017399920419fcb1e084cdb2@136.243.88.91:7330"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.ojo/config/config.toml
```

<div align="center">
  <h2> Address Book </h2>
</div>

```
curl -Ls https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/ojo-testnet/addrbook.json > $HOME/.ojo/config/addrbook.json
```
