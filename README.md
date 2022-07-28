#                                                                         Hermes_setup_manual
##                                                        Easy installation of IBC Relayer(Hermes) Stride & Gaia
![image](https://user-images.githubusercontent.com/93165931/181603865-ae62aa9b-bddb-4382-8ac6-280728cd6330.png)


## Create .hermes folder
```
cd $HOME
mkdir -p $HOME/.hermes/bin
```
## Install software
```
wget "https://github.com/informalsystems/ibc-rs/releases/download/v0.15.0/hermes-v0.15.0-x86_64-unknown-linux-gnu.tar.gz"
tar -C $HOME/.hermes/bin/ -vxzf hermes-v0.15.0-x86_64-unknown-linux-gnu.tar.gz
rm hermes-v0.15.0-x86_64-unknown-linux-gnu.tar.gz

echo "export PATH=$PATH:$HOME/.hermes/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Install dependencies

```
sudo apt install unzip -y
```
## Installing configurations


### STRIDE CHAIN

```
STRIDE_RPC="ENTER_YOUR_RPC"
STRIDE_RPC_PORT="26657"
STRIDE_GRPC_PORT="9090"
STRIDE_CHAIN_ID="STRIDE-TESTNET-2"
STRIDE_ACC_PREFIX="stride"
STRIDE_DENOM="ustrd"
STRIDE_REL_WALLET="ENTER_YOUR_WALLET_NAME"
```

### GAIA CHAIN
```
GAIA_RPC="ENTER_YOUR_RPC"
GAIA_RPC_PORT="46657"
GAIA_GRPC_PORT="9490"
GAIA_CHAIN_ID="GAIA"
GAIA_ACC_PREFIX="cosmos"
GAIA_DENOM="uatom"
GAIA_REL_WALLET="ENTER_YOUR_WALLET_NAME"
```

### Save result

```
echo "
export STRIDE_CHAIN_ID=${STRIDE_CHAIN_ID}
export STRIDE_DENOM=${STRIDE_DENOM}
export STRIDE_REL_WALLET=${STRIDE_REL_WALLET}
export GAIA_CHAIN_ID=${GAIA_CHAIN_ID}
export GAIA_DENOM=${GAIA_DENOM}
export GAIA_REL_WALLET=${GAIA_REL_WALLET}
" >> $HOME/.bash_profile

source $HOME/.bash_profile
```

## Create config  (Copy everything and paste)  

```
echo "[global]
log_level = 'info'
[mode]
[mode.clients]
enabled = true
refresh = true
misbehaviour = true
[mode.connections]
enabled = false
[mode.channels]
enabled = false
[mode.packets]
enabled = true
clear_interval = 100
clear_on_start = true
tx_confirmation = true
[rest]
enabled = true
host = '0.0.0.0'
port = 3000
[telemetry]
enabled = true
host = '0.0.0.0'
port = 3001
[[chains]]
id = '$STRIDE_CHAIN_ID'
rpc_addr = 'http://$STRIDE_RPC:$STRIDE_RPC_PORT'
grpc_addr = 'http://$STRIDE_RPC:$STRIDE_GRPC_PORT'
websocket_addr = 'ws://$STRIDE_RPC:$STRIDE_RPC_PORT/websocket'
rpc_timeout = '10s'
account_prefix = '$STRIDE_ACC_PREFIX'
key_name = '$STRIDE_REL_WALLET'
store_prefix = 'ibc'
max_tx_size = 100000
max_gas = 20000000
gas_price = { price = 0.001, denom = '$STRIDE_DENOM' }
gas_adjustment = 0.1
max_msg_num = 15
clock_drift = '5s'
trusting_period = '1days'
memo_prefix='YOUR_DISCORD'
trust_threshold = { numerator = '1', denominator = '3' }
[[chains]]
id = '$GAIA_CHAIN_ID'
rpc_addr = 'http://$GAIA_RPC:$GAIA_RPC_PORT'
grpc_addr = 'http://$GAIA_RPC:$GAIA_GRPC_PORT'
websocket_addr = 'ws://$GAIA_RPC:$GAIA_RPC_PORT/websocket'
rpc_timeout = '10s'
account_prefix = '$GAIA_ACC_PREFIX'
key_name = '$GAIA_REL_WALLET'
store_prefix = 'ibc'
max_tx_size = 100000
max_gas = 30000000
gas_price = { price = 0.001, denom = '$GAIA_DENOM' }
gas_adjustment = 0.1
max_msg_num = 15
clock_drift = '5s'
trusting_period = '1days'
memo_prefix= 'YOUR_DISCORD'
trust_threshold = { numerator = '1', denominator = '3' }" > $HOME/.hermes/config.toml

```
#Path config /root/.hermes/config.toml


## Output result

```
hermes config validate
```
![image](https://user-images.githubusercontent.com/93165931/181566948-191a8d67-399d-4cd7-9fe7-a11f049f6066.png)

## Create wallets(if there is no wallet):

```
STRIDE_REL_WALLET="YOUR_WALLET_NAME_STRIDE"
GAIA_REL_WALLET="YOUR_WALLET_NAME_GAIA"
STRIDE_CHAIN_ID="STRIDE-TESTNET-2"
GAIA_CHAIN_ID="GAIA"

```
### Stride Wallet

```
strided keys add $STRIDE_REL_WALLET

```
(please save mnemonic and wallet address)

### Gaia wallet

```
gaiad keys add $GAIA_REL_WALLET

```
(please save mnemonic and wallet address)

## Add wallet to Hermes

### Add wallet Stride

```
hermes keys restore $STRIDE_CHAIN_ID -n $STRIDE_REL_WALLET -m "your mnemonic phrase"
```
### Add wallet Gaia

```
hermes keys restore $GAIA_CHAIN_ID -n $GAIA_REL_WALLET -m "your mnemonic phrase"
```

### Check your balance 

```
strided q bank balances YOUR_ADDRESS_STRIDE
gaiad q bank balance YOUR_ADDRESS_GAIA
```

## Connecting to the channel

### Enter vars channel from output

```
HERMES_STRIDE_GAIA_CHANNEL_ID="channel-0"
HERMES_GAIA_STRIDE_CHANNEL_ID="channel-0"

echo "
export HERMES_STRIDE_GAIA_CHANNEL_ID=${HERMES_STRIDE_GAIA_CHANNEL_ID}
export HERMES_GAIA_STRIDE_CHANNEL_ID=${HERMES_GAIA_STRIDE_CHANNEL_ID}
" >> $HOME/.bash_profile

source $HOME/.bash_profile

```

### Check result 

```
hermes query channel end $STRIDE_CHAIN_ID transfer $HERMES_STRIDE_GAIA_CHANNEL_ID

hermes query channel end $GAIA_CHAIN_ID transfer $HERMES_GAIA_STRIDE_CHANNEL_ID
```

![image](https://user-images.githubusercontent.com/93165931/181574168-87611b18-8ec0-48ec-a535-7e622746c182.png)


### Run the service file

```
sudo tee /etc/systemd/system/hermesd.service > /dev/null <<EOF
[Unit]
Description=HERMES
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which hermes) start
Restart=on-failure
RestartSec=5
LimitNOFILE=6000
[Install]
WantedBy=multi-user.target
EOF

```
### Restart Hermes for start and run logs

```
sudo systemctl daemon-reload
sudo systemctl enable hermesd
sudo systemctl restart hermesd && journalctl -u hermesd -f

```
![image](https://user-images.githubusercontent.com/93165931/181575387-9d95e2e9-2f65-4a2a-91d1-e62c270db89c.png)

### Launch was successful

### You can send tokens by example

```
strided tx ibc-transfer transfer $HERMES_STRIDE_GAIA_CHANNEL_ID \ YOUR_WALLET_ADDRESS_STRIDE \ 777ustrd \  --from=STRIDE_REL_WALLET \
  --fees 3000ustrd
```

```
gaiad  tx ibc-transfer transfer $HERMES_GAIA_STRIDE_CHANNEL_ID \ YOUR_WALLET_ADDRESS_GAIA \ 777uatom \  --from=GAIA_REL_WALLET \
  --fees 3000uatom
```
  
  












