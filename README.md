# Side Protocol

# Side Protocol
Side Protocol Node Installation Instructions </br>
### [Official site](https://side.one)

### System requirements: </br>
CPU: 4 Core </br>
RAM: 8 Gb </br>
SSD: 200 Gb </br>
OS: Ubuntu 20.04 LTS </br>

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>
    
# Manual configuration of the Lava node with Cosmovisor
Required packages installation </br>
```
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl git jq lz4 build-essential
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```

Go installation.
```
cd $HOME
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

### Download and build binaries
```
cd $HOME && mkdir -p go/bin/
git clone https://github.com/sideprotocol/side.git
cd side
git checkout v0.8.1
make install

```

# Initialize the node
```
sided init NODENAME --chain-id=S2-testnet-2
sided config chain-id S2-testnet-2
```

# Download genesis files
```
wget https://github.com/sideprotocol/testnet/raw/main/S2-testnet-2/genesis.json -O ~/.side/config/genesis.json
```

# Set seeds and peears
```
peers="15f2a9dd8f903ef2894744f9fb3385deceb48c10@84.247.129.32:28656,9c14080752bdfa33f4624f83cd155e2d3976e303@65.108.231.124:45656,c13aa9de83b85825e2a773ab8ab1c0d4f749f38c@94.72.116.155:26656,65a8c4d4ec822cc95f4c3d1d017e9bf4721b8c6c@65.109.82.230:22656,d5519e378247dfb61dfe90652d1fe3e2b3005a5b@65.109.68.190:17456,ed87aea1ec989f81e5a14d629d343408d511a1a4@52.6.127.41:26656,5c2a752c9b1952dbed075c56c600c3a79b58c395@195.3.220.21:27516,5adabb1ff9c94807e0e1d986705bfb9dd608c3b9@138.201.51.154:53004"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$peers\"|" $HOME/.side/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.005uside\"/;" ~/.side/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.side/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.side/config/config.toml
seeds="582dedd866dd77f25ac0575118cf32df1ee50f98@202.182.119.24:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.side/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.side/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.side/config/config.toml
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.side/config/app.toml
indexer="null" &&
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.side/config/config.toml
```


# Create a service
```
sudo tee /etc/systemd/system/sided.service > /dev/null <<EOF
[Unit]
Description=sided
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sided) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

# Start the service and check the logs
```
sudo systemctl daemon-reload
sudo systemctl enable sided
sudo systemctl restart sided && sudo journalctl -fu sided -o cat
```

### Becoming a Validator

# Create wallet key new
```
sided keys add <walletname> --key-type="segwit"
```

(OPTIONAL) RECOVER EXISTING KEY
```
sided keys add <walletname> --key-type="segwit" --recover
```

### We receive tokens from the tap in the [faucet](https://testnet.side.one/faucet)

### Create the Validator

Before creating a validator, enter the command and check that you have false. This means that the Node has synchronized and you can create a validator:
```
sided status 2>&1 | jq .SyncInfo
```

### Check the balance before creating for the presence of tokens
```
sided q bank balances $(sided keys show wallet -a)
```

Please enter your details below in quotation marks where required

```
sided tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.1 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount 1000000uside \
--pubkey $(sided tendermint show-validator) \
--from <adresswallet> \
--moniker="YOUR NICK" \
--chain-id S2-testnet-2 \
--identity="" \
--website="" \
--details=""
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.005uside \
-y
```

### Update
```
There have been no updates at the moment, as soon as they come out, we will immediately add them to this section.

Current network:S2-testnet-2
Current version:v0.8.1
```

### Useful commands

Check balance
```
sided q bank balances $(sided keys show wallet -a)
```

CHECK SERVICE LOGS
```
sudo journalctl -fu sided -o cat
```

RESTART SERVICE
```
sudo systemctl restart sided
```

GET VALIDATOR INFO
```
sided status 2>&1 | jq .ValidatorInfo
```

DELEGATE TOKENS TO YOURSELF
```
sided tx staking delegate Your_valpoer........ "100000000"uside --from Wallet_Name --gas 350000 --chain-id=S2-testnet-1 -y
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop sided
sudo systemctl disable sided
rm /etc/systemd/system/sided.service
sudo systemctl daemon-reload
cd $HOME
rm -rf side
rm -rf .side
rm -rf $(which sided)
```
