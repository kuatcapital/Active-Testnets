```bash

### State Sync

SNAP_RPC=https://rpc.aura.m.sentrynode.xyz:443
SEEDS="34d759895c5a451488db34c686e74cb954d86723@65.108.135.212:23357"
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
mv $HOME/.aura/priv_validator_state.json.backup $HOME/.aura/data/priv_validator_state.json
wget -O $HOME/.aura/config/addrbook.json "https://github.com/kuatcapital/Active-Testnets/new/main/addrbook.json"
curl -o - -L https://github.com/kuatcapital/Active-Testnets/new/main/wasm-aura.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.aura --strip-components 2
sudo systemctl restart aurad && journalctl -u aurad -f -o cat
```

