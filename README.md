# symphony-node
##Here are the steps

# IF YOU ARE RUNNING OTHER NODES OR PROCESSES ON THIS PORT CHECK IF THE PORT 26656 IS FREE


These command will list all ports in usef if it was not listed in LISTEN then it is free

```console
ss -tuln
```

# check again if that port is free, if so, it connects, if not it doesn't give any output

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
# create cosmovisor folders
mkdir -p ~/.symphonyd/cosmovisor/genesis/bin
mkdir -p ~/.symphonyd/cosmovisor/upgrades

# load node bin into cosmovisor foldar
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
# enable that service
sudo systemctl enable symphony.service

# start that servive u have enabled
sudo service symphony start

# check logs
sudo journalctl -fu symphony

# use ctrl+c to exit logs
```
## wait atleast 10 - 30 min for sync
watch logs  ... when until you can see the commits and txn in the logs then get the faucet
it might take around 10 - 30 min

## create wallet
```console
symphonyd keys add wallet
```
give local password and save the nmemonic seed

go to kepler wallet and import that seed

## get faucet from 

```console
https://testnet.ping.pub/symphony/faucet
# and connect you wallet on the explorer page and add the syphony-testnet2 chain
# after adding the chain, go to kepler wallet and go to manage chain visibility and add symphony-testnet chain
```

## create delegator
```console
symphonyd tx staking create-validator \
  --amount 1000000note \
  --commission-max-change-rate "0.05" \
  --commission-max-rate "0.10" \
  --commission-rate "0.05" \
  --min-self-delegation "1" \
  --pubkey=$(symphonyd tendermint show-validator) \
  --moniker 'symphony-node' \
  --chain-id symphony-testnet-2 \
  --node https://symphony-testnet-rpc.polkachu.com:443  \
  --from wallet
```

## save that txn hash and done your validator node is created

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

```console
sudo systemctl start symphony.service
```
# to check logs
```console
sudo journalctl -fu symphony
```
# use ctrl + c to exit logs
