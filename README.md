# symphony-node
##Here are the steps
## install go
```
sudo rm -rvf /usr/local/go/
wget https://golang.org/dl/go1.21.1.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.1.linux-amd64.tar.gz
rm go1.21.1.linux-amd64.tar.gz
```

## configure go paths

```
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
```

## install cosmoviser

```
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0
```

## install node

```
cd $HOME && git clone https://github.com/Orchestra-Labs/symphony symphony
cd symphony
git checkout v0.2.1
make install
```

## inititalise your node

```
symphonyd init symphony-node --chain-id symphony-testnet-2
```

## download genesis

```
wget -O genesis.json https://snapshots.polkachu.com/testnet-genesis/symphony/genesis.json --inet4-only
mv genesis.json ~/.symphonyd/config
```

## adding bootstrap nodes

```
sed -i 's/seeds = ""/seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:29156"/' ~/.symphonyd/config/config.toml
```

## launching your node

```
# Create Cosmovisor Folders
mkdir -p ~/.symphonyd/cosmovisor/genesis/bin
mkdir -p ~/.symphonyd/cosmovisor/upgrades

# Load Node Binary into Cosmovisor Folder
cp ~/go/bin/symphonyd ~/.symphonyd/cosmovisor/genesis/bin
```

## create service file


# go to this directory
```
cd /etc/systemd/system
```
# then create that service file
```
sudo nano symphony.service
```
# then paste this file
```
[Unit]
Description="symphony node"
After=network-online.target

[Service]
User=USER
ExecStart=/home/USER/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=symphonyd"
Environment="DAEMON_HOME=/home/USER/.symphonyd"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```
# change ur name in that file

```
# run this whole command at once

USERNAME=$(whoami)
sudo tee /etc/systemd/system/symphony.service > /dev/null <<EOF
[Unit]
Description="symphony node"
After=network-online.target

[Service]
User=$USERNAME
ExecStart=/home/$USERNAME/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=symphonyd"
Environment="DAEMON_HOME=/home/$USERNAME/.symphonyd"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
EOF
 
```
 ## strat your node

```
# Enable service
sudo systemctl enable symphony.service

# Start service
sudo service symphony start

# Check logs
sudo journalctl -fu symphony

# use ctrl+c to exit logs
```

## to stop the node

```
 sudo systemctl stop symphony.service
```

