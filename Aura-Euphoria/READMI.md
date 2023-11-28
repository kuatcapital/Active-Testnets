```bash

### State Sync snapshot ### Aura Euphoria-2 

SNAP_RPC=https://rpc.aura.t.sentrynode.xyz:443
SEEDS="67b6531f01b751e532bc31457b41dbf5477ba53f@194.163.179.176:20657"
cp $HOME/.aura/data/priv_validator_state.json $HOME/.aura/priv_validator_state.json.backup
sed -i -e "/seeds =/ s/= .*/= "$SEEDS"/" $HOME/.aura/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height);
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000));
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).$|\1true| ;
s|^(rpc_servers[[:space:]]+=[[:space:]]+).$|\1"$SNAP_RPC,$SNAP_RPC"| ;
s|^(trust_height[[:space:]]+=[[:space:]]+).$|\1$BLOCK_HEIGHT| ;
s|^(trust_hash[[:space:]]+=[[:space:]]+).$|\1"$TRUST_HASH"| ;
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1""|" $HOME/.aura/config/config.toml
aurad tendermint unsafe-reset-all --home $HOME/.aura --keep-addr-book
sudo systemctl restart aurad && journalctl -u aurad -f -o cat
```

