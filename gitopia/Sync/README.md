<div align="center">
  <h2> Snapshot </h2>
</div>

> #### Snapshots allow the blockchain node to join the network in a matter of minutes by downloading our file which is a backup copy of the node. A snapshot contains a compressed copy of the date directory.
>
> #### Snapshots are taken automatically every 4 hours

```
sudo systemctl stop gitopiad

cp $HOME/.gitopia/data/priv_validator_state.json $HOME/.gitopia/priv_validator_state.json.backup

gitopiad tendermint unsafe-reset-all --home $HOME/.gitopia --keep-addr-book
curl https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/gitopia-testnet/gitopia-janus-testnet-2_latest.tar | tar -xf - -C $HOME/.gitopia/data
mv $HOME/.gitopia/priv_validator_state.json.backup $HOME/.gitopia/data/priv_validator_state.json

sudo systemctl start gitopiad
sudo journalctl -u gitopiad -f --no-hostname -o cat
```

<div align="center">
  <h2> State Sync </h2>
</div>

> #### State Sync allows a new node to join the network without completely downloading the entire blockchain node. This helps with server memory shortage. State Sync can cut down time to synchronization from days to minutes.

```
sudo systemctl stop gitopiad

cp $HOME/.gitopia/data/priv_validator_state.json $HOME/.gitopia/priv_validator_state.json.backup
gitopiad tendermint unsafe-reset-all --home $HOME/.gitopia --keep-addr-book

SNAP_RPC="https://rpc.gitopia-testnet.mirror-reflection.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="e1ab0573d55ff92fad55d2929e353904f1bbe36f@rpc.gitopia-testnet.mirror-reflection.com:31656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.gitopia/config/config.toml

sed -i 's|^enable *=.*|enable = true|' $HOME/.gitopia/config/config.toml
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $HOME/.gitopia/config/config.toml
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $HOME/.gitopia/config/config.toml
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $HOME/.gitopia/config/config.toml

mv $HOME/.gitopia/priv_validator_state.json.backup $HOME/.gitopia/data/priv_validator_state.json

sudo systemctl restart gitopiad
sudo journalctl -u gitopiad -f --no-hostname -o cat
```

<div align="center">
  <h2> Live Peers </h2>
</div>

```
PEERS="e1ab0573d55ff92fad55d2929e353904f1bbe36f@rpc.gitopia-testnet.mirror-reflection.com:31656,955c997a67a82cbd005e5b2b7010a1de3ac54355@38.242.241.74:26656,f314268ef1886e4ad2801c8443ea0b0c8143a246@95.214.55.25:30656,d318a60a25b7a84322a8083709ff8e8bbe82ddb7@65.108.13.154:26656,3b0956b482f89b361dd350f1c6b3743096897446@65.108.124.219:35656,0eb70bf5e2403694109f9bba184570074c2dfdd5@38.242.235.255:26656,d5519e378247dfb61dfe90652d1fe3e2b3005a5b@65.109.68.190:41656,6ea375302fdd319ef64e013f469e286faf739da8@213.239.207.165:20086,59cdd2944de1d9eccdbb013123f2dc8b6a5770f8@178.18.254.33:26656,ac606e28c081c679dc23d9a94c29842be8f8b1f1@45.85.249.133:656,f7fcda07044dc64cec2f6dca9da0c37a254bbae8@138.201.127.91:26676,1f0f03a1c845e810e5cfeb0d960639c637d049fe@154.26.131.130:36656,399d4e19186577b04c23296c4f7ecc53e61080cb@34.143.189.236:26656,9bb344d83fc1fafc4bce6b8e4a95b82f37ac4f31@82.208.20.136:26656,f0b8227e40f25eaec0e25b9e91ca199d2d9a1ecb@167.86.94.177:656,61c85d47e1dd86d5a5849450b849078d4d13184b@85.239.244.123:26656,eaa9978430e55663346eb61312cd5ecc21448b25@38.242.139.153:656,5c2a752c9b1952dbed075c56c600c3a79b58c395@195.3.220.140:27036,b745e0c6a1e0c7ec248ec274cfd038ed4bc4c2cf@65.21.134.202:26356,820024c34989e7605d9367847e1fc2d01ad763bd@65.109.92.235:30656,4cd60a4dd4211d38d948a86a614f1fd8d3d274eb@75.119.153.139:656,95fbdc6d62be17db6688222b15b57d3e795ed07a@167.86.84.102:656,ed177ff3cf334df1a6c190438b0c7b5dd64b423a@45.151.122.140:656,d2975b49708dc92ee3b7da1d72e3eee3119d1d0c@167.86.105.216:656,5c74fe6868cda2003926c0a6299c9cebec5c4d1a@65.21.239.60:41656,4e0e57bcac8aa2bc3188d5b7845eeee61a61f3f0@194.163.170.165:26656,9912d5c8d59b7736b0702b18aeb386efe7e46f3f@164.68.111.239:656,05182a9b6121c9fcbb493f9bb3843e20e076e479@38.242.231.113:656,007d2419fea80aee707d009af0153f5105c53379@38.242.139.164:656,e17763e03ef6819b6f549b97abe9da7a1a7eeac8@164.68.121.241:656,99a730ccb60933b5f9a282e899af7eb2e61d6ccc@81.30.157.35:26656,f1c042fca05e4bfb9a6da1cccaa5108a26ea1e0f@65.108.104.167:28656,4d9eb3521956f8611f18f8538ab40e82d64b3c03@85.239.243.221:26656,314ee8896c9f9e39450dc25623f8019cf316ed60@38.242.135.124:26656,8bec864d68a2542233ba37ac94c723fdf0b8e175@45.151.122.136:656,f9b892ea2e8ed8aa83f7b98e7e47371c23b01924@213.239.207.175:36656,bbc6a1e115185d5bffcbbf5520dca1c3d626e599@109.123.255.50:26656,9c265cb98c21d6748822ca2bed0accacdd8449db@38.242.205.25:26656,7d819fa869f7c5b42c2c7a9538e1a9e7a52cfdee@65.108.226.26:24656,d5006b48f6d89a8c803d87ae8788b4ce3b45bb0a@65.109.116.110:26656,03073657e8bc5bcf71e7fd8df281ab8dcbc8821a@45.151.122.130:656,d7759756161cb05711c4313f4e58ca57511f406a@148.251.91.185:28656,59a99a10a28baeda8535598acef9abb706ec5dbc@45.85.249.132:656,4ed110a5b1ebad62d1e92e8cdabfc9160e2ca4db@65.109.92.148:46656,0c31077af45cb4f0424e58c91b0a917c36a90fd9@65.108.195.235:16656,ffb4f7d43d6449c292d4e60c8a48eb3d31c39691@38.242.139.100:656,511eec6601a42d0b299b3e33c9deb1028d51855b@64.227.176.9:26656,bc688b2be879ba5bfa34587e096a9c9a4df2e6d4@45.151.122.116:656,15bb9edc16710d321163e7ef8b9a44959dd7e657@65.108.126.46:30656,c84906b19dc7dc7bda94ab2167d4b0af64a28b49@45.151.122.191:656,f026faf0dfc8a19f8029c6ec08d1e8454a2c9475@149.102.133.56:26656,35c829910f80387ee825da9fb69efbcbf8e2149e@164.68.118.227:26656,619a23818cddd40d0b9f57e9754b719da13609bc@65.108.108.52:24656,37c3d29df83da59e5a258d413e2f89365ab05711@85.239.243.12:656,98bdfc67810bf7ac8f5c45b2c677b4bf199eb42e@185.193.67.65:41656,5b1c25f4dff541f77f1532c457f73ca7ee2e4c18@194.163.170.225:26656,247dbc8048be7c024c5f5deee45c18bd2f19bc93@116.203.35.46:36656,1989ced6b71ce676a5ab4d0586d85e38fd41fbd2@136.243.88.91:7070,66f94651fb02f277c90c605a38df549d3c0a9269@75.119.151.217:26656,7da6c90fe420bca73b5274884236134acf49d565@35.168.32.254:26656"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.gitopia/config/config.toml
```

<div align="center">
  <h2> Address Book </h2>
</div>

```
curl -Ls https://snapshots-cosmos.mirror-reflection.com/cosmos-testnet/gitopia-testnet/addrbook.json > $HOME/.gitopia/config/addrbook.json
```
