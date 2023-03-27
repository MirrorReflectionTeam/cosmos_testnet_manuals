<div align="center">
  <h2> Snapshot </h2>
</div>

> #### Snapshots allow the blockchain node to join the network in a matter of minutes by downloading our file which is a backup copy of the node. A snapshot contains a compressed copy of the date directory.
>
> #### Snapshots are taken automatically every 4 hours

```
sudo systemctl stop nibid

cp $HOME/.nibid/data/priv_validator_state.json $HOME/.nibid/priv_validator_state.json.backup

nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/nibiru-testnet/nibiru-itn-1_latest.tar | tar -xf - -C $HOME/.nibid/data

mv $HOME/.nibid/priv_validator_state.json.backup $HOME/.nibid/data/priv_validator_state.json

sudo systemctl start nibid
sudo journalctl -u nibid -f --no-hostname -o cat
```

<div align="center">
  <h2> State Sync </h2>
</div>

> #### State Sync allows a new node to join the network without completely downloading the entire blockchain node. This helps with server memory shortage. State Sync can cut down time to synchronization from days to minutes.


```
sudo systemctl stop nibid

cp $HOME/.nibid/data/priv_validator_state.json $HOME/.nibid/priv_validator_state.json.backup
nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book

SNAP_RPC="https://rpc.nibiru-testnet.mirror-reflection.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="3358dc86108195da5bc8fb6a33e90927f1fd97bb@rpc.nibiru-testnet.mirror-reflection.com:34656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.nibid/config/config.toml

sed -i 's|^enable *=.*|enable = true|' $HOME/.nibid/config/config.toml
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $HOME/.nibid/config/config.toml
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $HOME/.nibid/config/config.toml
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $HOME/.nibid/config/config.toml

mv $HOME/.nibid/priv_validator_state.json.backup $HOME/.nibid/data/priv_validator_state.json

curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/nibiru-testnet/wasm_nibiru-itn-1_.tar | tar -xf - -C $HOME/.nibid/data

sudo systemctl restart nibid
sudo journalctl -u nibid -f --no-hostname -o cat
```

<div align="center">
  <h2> Live Peers </h2>
</div>

```
PEERS="3358dc86108195da5bc8fb6a33e90927f1fd97bb@rpc.nibiru-testnet.mirror-reflection.com:34656,19f8333cbcaf4f946d20a91d9e19d5fc91899023@167.235.74.22:26656,2735e802d9abf70e9d88ba37f8dbab6a26f1c065@38.242.238.46:26656,6f29a743ad237d435aac29b62528cea01ceb3ca9@46.4.91.90:26656,6b3ca7670406494b2b2f579c6658cb32d7f09e47@161.97.146.91:49656,b300cdfbbd9af83bab94fcfc493d43a88ab01acc@80.82.215.214:39656,f9e2c35f5da87933bcc092ab9f14d45b08d3e89d@65.108.145.226:26656,1d2d3e8353043b25675040258912fb04cfc3eff9@162.55.242.81:26656,aafe706e7bb9aac7e8ec2878d775252652594b3e@78.46.97.242:26656,7b8da020f7cf7b4928e34b3ba8d47d1ee5ed5944@45.151.123.51:26656,d1e92a70f1503e3a4488f022bc408f5cd8e207b3@184.174.38.102:26656,7ab4f18d9745548b1005be91cb1a96785a0abf32@185.135.137.123:26656,c8ff97aef44a725c908338a7c67c40383e3e074e@164.90.239.51:26656,9f993f07f3fe0c788f7d55f88b7a031028a378f5@217.76.60.46:26656,4f1af4f62f76c095d844384a3dfa1ad76ad5c078@65.108.206.118:60656,3060a899170ccb3d787d6fd840c5e8b6805f4b4d@155.133.22.140:26656,5dd26cc6a2778cea7c0daed0a53a39c6165a790f@168.119.101.224:26656,f2e99f5a68adfb08c139944a193e2e3a4864b038@167.235.132.74:26656,e8d2c594a6371a5b67778b79bfc10e1d46ae6357@185.206.213.168:26656,bb54d4190da91d6a9c1f7322f9e8e38844a9e6b0@173.212.196.242:26656,e74f1204d65d0264547e2c2d917c23c39fcff774@95.217.107.96:36656,d3624259ec8322cd4b6bce5b39aaf6074f90a2ab@173.212.248.126:26656,aa166a62f6983edccd5a2619b036fe83cb4eb57e@168.119.248.238:26656,30ad7f27c225de1ed881fd355d2d006d445f281a@84.46.248.169:26656,22f1bf214da6c0c1e6c6b78bc6005ac4fc4456da@46.228.205.211:26656,cd44f2d2fc1ded3a63c64f46ed67f783c2d93d57@144.76.223.24:36656,ff4fdb162a01a7e8e36bfdef66d33a9ac7368588@172.105.71.125:39656,23307798ac9ad05dd36e2eeef063d701b213cbbb@185.215.167.62:26656,d26e9d2a81ce80328d3a9aeabb0820fbc343b5f4@161.97.170.91:26656,5a58fa951f65cd2381d0f9a584fbb76fdafc476c@45.10.154.139:18656,52c421c35293a06bf0ccb4a8ffe5f1573a71a812@89.117.59.203:26656,013c07456f39a2ca2de7ceb30ef9d0f2bc2f2d21@65.21.228.226:26656,81351ddd64122e553cf2c10efbd979c8c6e97529@161.97.166.105:26656,4a70de4fffde46382e70250c06f744570ce72ef9@138.201.124.93:26656,af28ead03cd16f4a901cebfdf946968d9a492f73@89.117.60.189:26656,5f3394bae3791bcb71364df80f99f22bd33cc2c0@95.216.7.169:60556,d622efcde775f33bd8c14fa5757ee9fa95d4149e@135.181.203.53:26656,7736aeb85e6dc91d0c952a25ab73b09da49d03ca@136.243.136.241:22656,4ae091976ef83403cbbb55345a1af0a06f3ef524@144.76.101.41:26656,d9bea9db5daa0e2be0c2ee9ccda5168fe80f12d3@45.85.249.73:26656,7a3c1cc4f4ccb40614e6d53a288628c209b54d59@31.220.73.20:36656,eddd4e1775b222e2d52f5207d35d4f28417d09ec@31.220.87.37:26656,7b71facfb46ccd860d4d71696b3a077676a6f2b5@65.109.166.8:26656,b6f2235952069566b0fe4372c46d90b1e5b0e6d3@154.12.249.208:26656,adcb0c33d521b5a26e83834454ed126988493573@95.216.74.4:26656,09eeb97c2f840c13bee3b0f06a4231bbdf249912@81.0.218.21:26656,d0ce7b356e76298ea59aeb78397e9d84f0ed2480@31.220.88.180:26656,0cdd0e30f346f02eee2a0ad01887dac627339dd2@35.247.95.191:26656,66acb18ee9d46a32f75a46296f9fe62474aae4a5@65.108.227.112:12656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.nibid/config/config.toml
```

<div align="center">
  <h2> Address Book </h2>
</div>

```
curl -s https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/nibiru-testnet/addrbook.json > $HOME/.nibid/config/addrbook.json
```

<div align="center">
  <h2> Wasm </h2>
</div>

```
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/nibiru-testnet/wasm_nibiru-itn-1_.tar | tar -xf - -C $HOME/.nibid/data
```
