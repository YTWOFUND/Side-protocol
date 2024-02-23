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
cd $HOME
rm -rf sidechain
git clone -b dev https://github.com/sideprotocol/sidechain.git
cd sidechain
git checkout v0.6.0
make install
sided version
```

# Initialize the node
```
sided init YOUR MONIKER --chain-id=side-testnet-2
```

# Download genesis and addrbook files
```
curl -Ls https://ss-t.side.nodestake.org/genesis.json > $HOME/.side/config/genesis.json 
curl -Ls https://ss-t.side.nodestake.org/addrbook.json > $HOME/.side/config/addrbook.json
```

# Download latest chain data snapshot
```
SNAP_NAME=$(curl -s https://ss-t.side.nodestake.org/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
curl -o - -L https://ss-t.side.nodestake.top/${SNAP_NAME}  | lz4 -c -d - | tar -x -C $HOME/.side
```

# Create a service
```
sudo tee /etc/systemd/system/sided.service > /dev/null <<EOF
[Unit]
Description=sided Daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which sided) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable sided

# Start the service and check the logs
sudo systemctl restart sided
journalctl -u sided -f
```

### Becoming a Validator

# Create wallet key new
```
sided keys add wallet
```

(OPTIONAL) RECOVER EXISTING KEY
```
sided keys add wallet --recover
```

### We receive tokens from the tap in the discord(https://discord.com/invite/BfEHpm6uFc)

Go to the #testnet-faucet branch and specify your side wallet $request side-testnet-2 (side....)

### Create the Validator

Before creating a validator, enter the command and check that you have false. This means that the Node has synchronized and you can create a validator:
```
sided status 2>&1 | jq .SyncInfo.catching_up
```

### Check the balance before creating for the presence of tokens
```
sided q bank balances $(sided keys show wallet -a)
```

Please enter your details below in quotation marks where required

```
sided tx staking create-validator \
--amount=1000000ibc/8A138BC76D0FB2665F8937EC2BF01B9F6A714F6127221A0E155106A45E09BCC5 \
--pubkey=$(sided tendermint show-validator) \
--moniker="Moniker" \
--identity=FFB0AA51A2DF5955 \
--details="I love YTWO" \
--chain-id=side-testnet-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.005ibc/8A138BC76D0FB2665F8937EC2BF01B9F6A714F6127221A0E155106A45E09BCC5 \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Update
```
There have been no updates at the moment, as soon as they come out, we will immediately add them to this section.

Current network:side-testnet-2 
Current version:v0.6.0
```

### Useful commands

Check balance
```
sided q bank balances $(sided keys show wallet -a)
```

CHECK SERVICE LOGS
```
journalctl -u sided -f
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
sided tx staking delegate $(sided keys show wallet --bech val -a) 1000000ibc/8A138BC76D0FB2665F8937EC2BF01B9F6A714F6127221A0E155106A45E09BCC5 --from wallet --chain-id side-testnet-1 --gas-prices 0.005ibc/8A138BC76D0FB2665F8937EC2BF01B9F6A714F6127221A0E155106A45E09BCC5 --gas-adjustment 1.5 --gas auto -y
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop sided && sudo systemctl disable sided && sudo rm /etc/systemd/system/sided.service && sudo systemctl daemon-reload && rm -rf $HOME/.sidechain && rm -rf sidechain && sudo rm -rf $(which sided) 
```
