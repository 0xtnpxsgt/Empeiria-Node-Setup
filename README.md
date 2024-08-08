# Empeiria-Node-Setup

```bash
🖥️System Requirements
- OS: Linux
- CPU: 6 Core(s)
- Memory: 32GB
- Storage: 300GB
```

- This guide is created under the assumption you are using Ubuntu 22.04 LTS
- You can buy from : Contabo
- You should buy VPS which is fulfilling all these requirements : 

## Install dependencies
```bash
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential snapd unzip nginx
sudo apt -qy upgrade
```

## Set environment
```bash
export INSTALLATION_DIR=${HOME}/empe-chain
export DAEMON_NAME=emped
export DAEMON_HOME=${HOME}/.empe-chain
export SERVICE_NAME=emped
export MONIKER="YOUR_NODE_NAME_HERE"
export WALLET="YOUR_WALLET_NAME_HERE"
```

## Write env to .profile
```bash
echo 'export DAEMON_NAME=${DAEMON_NAME}' >> ~/.profile
echo 'export DAEMON_HOME=${DAEMON_HOME}' >> ~/.profile
echo 'export DAEMON_ALLOW_DOWNLOAD_BINARIES=true' >> ~/.profile
echo 'export DAEMON_RESTART_AFTER_UPGRADE=true' >> ~/.profile
echo 'export DAEMON_LOG_BUFFER_SIZE=512' >> ~/.profile
echo 'export WALLET=${WALLET}' >> ~/.profile
source ~/.profile
```

## Prepare installation
```bash
mkdir -p ${INSTALLATION_DIR}/bin
mkdir -p ${DAEMON_HOME}/cosmovisor/genesis/bin
mkdir -p ${DAEMON_HOME}/cosmovisor/upgrades
```

## Install and Setup Empeiria Daemon
```bash
cd ${INSTALLATION_DIR}
```
```bash
#Download Empe Daemon and basic setup
wget -qO - https://github.com/empe-io/empe-chain-releases/raw/master/v0.1.0/emped_linux_amd64.tar.gz | tar -xzf -
mv ${DAEMON_NAME} ${INSTALLATION_DIR}/bin
```
```bash
#Download and install cosmovisor
wget https://github.com/cosmos/cosmos-sdk/releases/download/cosmovisor%2Fv1.5.0/cosmovisor-v1.5.0-linux-amd64.tar.gz
tar -xvzf cosmovisor-v1.5.0-linux-amd64.tar.gz
```
```bash
#Copy Binaries
cp cosmovisor ${INSTALLATION_DIR}/bin/cosmovisor
cp ${INSTALLATION_DIR}/bin/${DAEMON_NAME} ${DAEMON_HOME}/cosmovisor/genesis/bin
sudo ln -s ${INSTALLATION_DIR}/bin/cosmovisor /usr/local/bin/cosmovisor -f
sudo ln -s ${DAEMON_HOME}/cosmovisor/genesis ${DAEMON_HOME}/cosmovisor/current -f
sudo ln -s ${DAEMON_HOME}/cosmovisor/current/bin/${DAEMON_NAME} /usr/local/bin/${DAEMON_NAME} -f
```

## Check emped version
```bash
emped --home ${DAEMON_HOME} version
```
## Download Genesis 
```bash
wget -O $HOME/.empe-chain/config/genesis.json "https://raw.githubusercontent.com/empe-io/empe-chains/master/testnet-2/genesis.json"
wget -O $HOME/.empe-chain/config/addrbook.json "https://raw.githubusercontent.com/MictoNode/empe-chain/main/addrbook.json"
```

## Peers
```bash
SEEDS=""
PEERS="edfc10bbf28b5052658b3b8b901d7d0fc25812a0@193.70.45.145:26656,4bd60dee1cb81cb544f545589b8dd286a7b3fd65@149.202.73.140:26656,149383fab60d8845c408dce7bb93c05aa1fd115e@54.37.80.141:26656,83f9769416445c3c0b0b3a9f79c2b4e19b45441b@94.16.115.147:43656,99ff20ec0c3623d2d725924b8fd0f1c4e1b22e15@195.201.59.173:26656,eed34161ff47076ad1ff83e68942bdd667106536@159.69.179.43:43656,4ff7d588d4c5d59a7208d4c0457cc3c26e6713cd@78.46.19.116:29056,a753ff99004f7860b546fd4b101f9e03e9ab0295@95.216.40.250:26656,692099b20acde5520084ecea12ddd9a36d4ca54d@178.18.251.146:12656,098f3c412e5a32948c526f8b9b7066d3f9f3786a@194.233.69.9:26656,829207ca2cf7debb16787a79c9fc1aa94e9b55ea@116.203.238.65:43656,5406f64d38f433cca31c2f6e96d5619fa92be5b5@168.119.179.250:26656,a9cf0ffdef421d1f4f4a3e1573800f4ee6529773@136.243.13.36:29056,0c0a348442fa5881ce15aba1030214cc7fad23ba@37.27.193.4:43656,94529b5e044f208d1869980f456a53fcef8fb321@14.167.155.13:43656,2354e634c9e8f630e2f93c6d3a7b845c681d00bb@167.235.102.45:11756,d8dedcd1b8c541141e9c57a23db35bea44a05129@37.27.129.24:23656,37f0a23a5eafa80e0ffb8961251343afc6efdea0@37.60.226.37:43656,5dfcb1c82cc041b18ff86dd520ce9185a2f0220c@116.203.133.101:43656,3373d3b6f215cc6dfb8e8b172053b72387575cb0@207.180.198.20:43656,ef2ad74a4f2e5e2a106f8b3df468984b267e2c02@5.9.73.170:29056,45bdc8628385d34afc271206ac629b07675cd614@65.21.202.124:25656,1c72d5acca5b75eec4aa9cfce4a8c298f7fecf46@20.78.12.218:26656,0a123ef98ea1d1176cdf7bec2b416b93010422bd@148.113.170.13:16610"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.empe-chain/config/config.toml
```

## Create or Restore Wallet
```bash
#If you want to create new wallet
emped --home ${DAEMON_HOME} keys add ${WALLET}

#If you already have wallet and want to use same phrase
emped --home ${DAEMON_HOME} keys add ${WALLET} --recover
```

## Check your wallet
```bash
emped --home ${DAEMON_HOME} keys list
```

## Setup pruning config
```bash
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
  ${DAEMON_HOME}/config/app.toml
```

## Setting minimum gas fee
```bash
sed -i 's/minimum-gas-prices *=.*/minimum-gas-prices = "0.0001uempe"/' ${DAEMON_HOME}/config/app.toml
```

## Setting up Cosmovisor
```bash
sudo tee /etc/systemd/system/emped.service > /dev/null <<EOF  
[Unit]
Description=Empeiria Testnet Daemon (cosmovisor)
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --home ${DAEMON_HOME}
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=${DAEMON_NAME}"
Environment="DAEMON_HOME=${DAEMON_HOME}"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"

[Install]
WantedBy=multi-user.target
EOF
```

## Enable the service
```bash
sudo systemctl daemon-reload
sudo systemctl enable emped.service
sudo systemctl start emped.service
```


## Download Snapshot
```bash
#Stop the service
sudo systemctl stop emped.service
```
```bash
cp $HOME/.empe-chain/data/priv_validator_state.json $HOME/.empe-chain/priv_validator_state.json.backup
```

- Check the latest Version here by Date - https://server-5.itrocket.net/testnet/empeiria/
```bash
apt install aria2c
```
```bash
if curl -s --head https://server-5.itrocket.net/testnet/empeiria/empeiria_2024-08-07_865547_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  echo "Snapshot found, downloading..."
  aria2c -x5 -s4 https://server-5.itrocket.net/testnet/empeiria/empeiria_2024-08-07_865547_snap.tar.lz4 -o empe_snapshot.lz4
  if [ $? -eq 0 ]; then
    echo "Download complete, extracting..."
    lz4 -dc empe_snapshot.lz4 | tar -xf - -C $HOME/.empe-chain
    if [ $? -eq 0 ]; then
      echo "Snapshot downloaded and extracted successfully."
    else
      echo "Failed to extract snapshot."
    fi
  else
    echo "Failed to download snapshot."
  fi
else
  echo "No snapshot found."
fi
```
```bash
mv $HOME/.empe-chain/priv_validator_state.json.backup $HOME/.empe-chain/data/priv_validator_state.json
```


## Check logs
```bash
sudo systemctl restart emped && sudo journalctl -u emped -f --no-hostname -o cat
```

## Check Sync Info
#### False = Fully Sync
```bash
emped status 2>&1 | jq .SyncInfo
```

#### If already Fully Sync - Please Proceed 

## Validator Setup - Fill this with your validator info
```bash
export MONIKER="YOUR_NODE_NAME_HERE"
export WALLET="YOUR_WALLET_NAME_HERE"
export DETAILS="YOUR_NODE_INFORMATION_HERE"
export WEBSITE="YOUR_WEBSITE_HERE"
export DISPLAY_PIC="YOUR_KEYBASE_IMAGE_PGP"
export CONTACT="YOUR_CONTACT"
```
- Display Picture - Create Account here: https://keybase.io/
<img width="854" alt="Screenshot 1403-05-17 at 8 59 46 PM" src="https://github.com/user-attachments/assets/e9455690-1a3f-4442-a04f-f42623d48e0e">


## Write env to .profile
```bash
echo 'export MONIKER=${MONIKER}' >> ~/.profile
echo 'export WALLET=${WALLET}' >> ~/.profile
echo 'export DETAILS=${DETAILS}' >> ~/.profile
echo 'export WEBSITE=${WEBSITE}' >> ~/.profile
echo 'export DISPLAY_PIC=${DISPLAY_PIC}' >> ~/.profile
echo 'export CONTACT=${CONTACT}' >> ~/.profile
source ~/.profile
```
## Get Testnet EMPE - 
#### You can obtain testnet tokens to fund your address from official EMPE faucet: https://faucet.cnd.biz.id/
or Join here https://t.me/empevalidators and request token from the admin
## You can verify your balance with this command:
```bash
emped query bank balances $(emped keys show $WALLET -a)
```

## Create Validator
```bash
emped tendermint show-validator
```
#### The output will be similar to this (with a different key):
```bash
{"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="}
```
## You can create your validator by using command below :
```bash
emped tx staking create-validator \
  --amount=10000000uempe \
  --pubkey=$(emped tendermint show-validator) \
  --moniker=${MONIKER} \
  --identity=${DISPLAY_PIC} \
  --website=${WEBSITE} \
  --security-contact=${CONTACT} \
  --details=${DETAILS} \
  --chain-id=empe-testnet-2 \
  --commission-rate="0.05" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --gas="auto" \
  --min-self-delegation="1000000" \
  --fees=25uempe \
  --from=${WALLET}
```

## Self Delegate
```bash
emped tx staking delegate $(emped keys show $WALLET --bech val -a) 1000000uempe \
--from $WALLET \
--chain-id empe-testnet-2 \
--gas auto \
--gas-adjustment 1.5 \
--fees 25uempe -y
```

- Check your validator here:
 https://explorer-testnet.empe.io/validators/your_validator_address

#### Done Congratulation!

#### View Validator Info 
```bash
emped q staking validator $(emped keys show "YOUR_WALLET_NAME" --bech val -a)
```

## Other Commands
```bash
#Start 
sudo systemctl start emped
```
```bash
#Stop 
sudo systemctl stop emped
```
```bash
#Status
sudo systemctl status emped
```














