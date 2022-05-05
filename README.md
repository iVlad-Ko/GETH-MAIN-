# GETH-MAIN-
Light Node Install
# GETH | MAIN Light Node Install

```
# docs
[https://geth.ethereum.org/](https://geth.ethereum.org/)
[https://github.com/ethereum/go-ethereum](https://github.com/ethereum/go-ethereum)

# bootnodes
[https://github.com/ethereum/go-ethereum/blob/master/params/bootnodes.go](https://github.com/ethereum/go-ethereum/blob/master/params/bootnodes.go)

# explorer
[https://etherscan.io](https://ropsten.etherscan.io)
```

## Config ports and data directory

### Set vars

```bash
# set vars
ETH_RAM_USE=4096                            # change to increase performance (default is 4096)
   
ETH_DATA_DIR=$HOME/.ethereum                # 
# ETH_DATA_DIR="/root/.ethereum"            # default data_dir and changed name

ETH_RPC_PORT="8545"                         # default
ETH_WS_PORT="8546"                          # default
ETH_P2P_PORT="30303"                        # default
```

### Save vars

```bash
# save vars
echo 'export ETH_RPC_PORT='${ETH_RPC_PORT} >> $HOME/.bash_profile
echo 'export ETH_WS_PORT='${ETH_WS_PORT} >> $HOME/.bash_profile
echo 'export ETH_P2P_PORT='${ETH_P2P_PORT} >> $HOME/.bash_profile
echo 'export ETH_DATA_DIR='${ETH_DATA_DIR} >> $HOME/.bash_profile
echo 'export ETH_RAM_USE='${ETH_RAM_USE} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## **Install node**

### Install `geth`

```bash
# check latest release first
[https://geth.ethereum.org/downloads/](https://geth.ethereum.org/downloads/)
# install geth
cd $HOME
release="1.10.17-25c9b49f"   # actual at "02 May 2022"
wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-${release}.tar.gz
tar -xvf geth-linux-amd64-${release}.tar.gz
rm -v geth-linux-amd64-${release}.tar.gz

chmod +x $HOME/geth-linux-amd64-${release}/geth
sudo mv $HOME/geth-linux-amd64-${release}/geth /usr/bin/
```

### Create service

```bash

# create service
sudo tee /etc/systemd/system/geth-main-light.service > /dev/null <<EOF
[Unit]
Description=geth-main-light
After=network-online.target
[Service]
User=$USER
ExecStart=$(which geth) \
  --cache $ETH_RAM_USE \
  --syncmode "light" \
  --mainnet \
  --port $ETH_P2P_PORT \
  --http.port $ETH_RPC_PORT \
  --http --http.vhosts "*" --http.api "eth,net,web3" --http.corsdomain '*' --http.addr 0.0.0.0 \
  --ws.port $ETH_WS_PORT \
  --ws --ws.api eth,net,web3 --ws.addr 0.0.0.0 \
  --datadir $ETH_DATA_DIR
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable geth-main-light
sudo systemctl restart geth-main-light && journalctl -u geth-main-light -f

```

## Check node status

```bash
# It takes time to find peers, be patient

# check
geth attach ipc:$ETH_DATA_DIR/geth.ipc
> net.peerCount
# must output more than 0

> eth.syncing
# if outputs **false** that ****means synced

# output example if still syncing
{
  currentBlock: 11474943,
  healedBytecodeBytes: 0,
  healedBytecodes: 0,
  healedTrienodeBytes: 0,
  healedTrienodes: 0,
  healingBytecode: 0,
  healingTrienodes: 0,
  highestBlock: 11766097,
  startingBlock: 11370495,
  syncedAccountBytes: 0,
  syncedAccounts: 0,
  syncedBytecodeBytes: 0,
  syncedBytecodes: 0,
  syncedStorage: 0,
  syncedStorageBytes: 0
}
```

## RPC url

```bash
http://IP:PORT

# EXAMPLE
ETH_RPC_URL="http://xx.xxx.xx.xxx:8545"

# check
curl -X POST $ETH_RPC_URL \
     -H "Content-Type: application/json" \
     -d '{"id":1, "jsonrpc":"2.0", "method": "eth_syncing", "params":[]}'

# correct output {"jsonrpc":"2.0","id":1,"result":**false**}
```

## Ports in use

0.0.0.0:8545     # RPC
0.0.0.0:8546     # WS
0.0.0.0:30303    # P2P
