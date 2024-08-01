# symphony-node
##Here are the steps

# IF U ARE RUNNOG OTHER NODES OR PROCESSES CHECK IF THE 26656 PORT IS FREE


These command will list all ports in usef if it was not listed in LISTEN then it is free

```console
ss -tuln
```

# check again if that port is free, if its used, it connects if not it will not give any output

```console
nc -zv 127.0.0.1 26656
```


# if the port is free proceed with the below 

## create virtual environment 
# download virtual env. packages
```console
sudo apt update
sudo apt install python3 python3-venv
```

# create env.
```console
cd $HOME
python3 -m venv symphony-venv

```
# actiavte it

```console
cd $HOME
source symphony-venv/bin/activate
```


## install go
```console
sudo rm -rvf /usr/local/go/
wget https://golang.org/dl/go1.21.1.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.1.linux-amd64.tar.gz
rm go1.21.1.linux-amd64.tar.gz
```

## configure go paths

```console
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
```

## install cosmoviser

```console
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0
```

## install node

```console
cd $HOME && git clone https://github.com/Orchestra-Labs/symphony symphony
cd symphony
git checkout v0.2.1
make install
```

## inititalise your node

```console
symphonyd init symphony-node --chain-id symphony-testnet-2
```

## download genesis

```console
wget -O genesis.json https://snapshots.polkachu.com/testnet-genesis/symphony/genesis.json --inet4-only
mv genesis.json ~/.symphonyd/config
```

## adding bootstrap nodes

```console
sed -i 's/seeds = ""/seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:29156"/' ~/.symphonyd/config/config.toml
```

## launching your node

```console
# Create Cosmovisor Folders
mkdir -p ~/.symphonyd/cosmovisor/genesis/bin
mkdir -p ~/.symphonyd/cosmovisor/upgrades

# Load Node Binary into Cosmovisor Folder
cp ~/go/bin/symphonyd ~/.symphonyd/cosmovisor/genesis/bin
```

## create service file


# go to this directory
```console
cd /etc/systemd/system
```
## then create that service file


# run this whole command at once

```console
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

```console
# Enable service
sudo systemctl enable symphony.service

# Start service
sudo service symphony start

# Check logs
sudo journalctl -fu symphony

# use ctrl+c to exit logs
```

## to stop the node

```console
 sudo systemctl stop symphony.service
```

# to start the node again

# activate the virtual env.

```console
cd $HOME
source symphony-venv/bin/activate
```

# start the node 

```
sudo systemctl start symphony.service
```
# to check logs
```
sudo journalctl -fu symphony
```
# use ctrl + c to exit logs
