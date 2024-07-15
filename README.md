# Side Protocol

# Side Protocol
Side Protocol Node Installation Instructions </br>
### [Official site](https://side.one)

### System requirements: </br>

Minimum Requirements
CPU: 4 cores
RAM: 8 GB
Storage: 500 GB
Network: 1 Gbps

Recommended Specifications
CPU: 8 cores
RAM: 16 GB
Storage: 800 GB
Network: 1 Gbps

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
git checkout v0.9.0
make install
```

# Initialize the node
```
sided init NODENAME --chain-id=grimoria-testnet-1
sided config chain-id grimoria-testnet-1
```

# Download genesis files
```
wget https://raw.githubusercontent.com/sideprotocol/testnet/main/grimoria-testnet-1/genesis.json -O $HOME/.side/config/genesis.json
```

# Install snapshot
```
cd $HOME
apt install lz4
sudo systemctl stop sided
cp $HOME/.side/data/priv_validator_state.json $HOME/.side/priv_validator_state.json.backup
rm -rf $HOME/.side/data
curl -o - -L https://side-t.snapshot.stavr.tech/side-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.side --strip-components 2
curl -o - -L https://side-t.wasm.stavr.tech/wasm-side.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.side --strip-components 2
mv $HOME/.side/priv_validator_state.json.backup $HOME/.side/data/priv_validator_state.json
wget -O $HOME/.side/config/addrbook.json "https://raw.githubusercontent.com/111STAVR111/props/main/Side/addrbook.json"
sudo systemctl restart sided && journalctl -u sided -f -o cat
```

# Set seeds and peears
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.005uside\"/;" ~/.side/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.side/config/config.toml
peers="6bef0693d7a31fed473b95123ad19b544b414093@202.182.119.24:26656,44f8009ed91fddd7ee51117482ede20fb4f33e78@149.28.156.79:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.side/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.side/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.side/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.side/config/config.toml
wget -O $HOME/.side/config/addrbook.json "https://snap1.konsortech.xyz/side-testnet/addrbook.json"
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
--chain-id grimoria-testnet-1 \
--identity="" \
--website="" \
--details=""
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.005uside \
-y
```

### Upgrade
```
There have been no updates at the moment, as soon as they come out, we will immediately add them to this section.

Current network:grimoria-testnet-1
Current version:v0.9.0
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
