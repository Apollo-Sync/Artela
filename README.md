# Artela

Website: artela.network

X      : https://x.com/Artela_Network

Discord: https://discord.com/invite/artela

**Clone project repository**
```
cd && rm -rf artela
git clone https://github.com/artela-network/artela
cd artela
git checkout v0.4.7-rc7-fix-execution
```

**Build binary**
```
make install
```

**Set node CLI configuration**
```
artelad config chain-id artela_11820-1
artelad config keyring-backend test
artelad config node tcp://localhost:27857
```

**Initialize the node**
```
artelad init "Your Node Name" --chain-id artela_11820-1
```
**Download genesis and addrbook files**
```
curl -L https://raw.githubusercontent.com/Apollo-Sync/Artela/main/genesis.json > $HOME/.artelad/config/genesis.json
curl -L https://raw.githubusercontent.com/Apollo-Sync/Artela/main/addrbook.json > $HOME/.artelad/config/addrbook.json
```

**Set seeds**
```
sed -i -e 's|^seeds *=.*|seeds = "211536ab1414b5b9a2a759694902ea619b29c8b1@47.251.14.47:26656,d89e10d917f6f7472125aa4c060c05afa78a9d65@47.251.32.165:26656,bec6934fcddbac139bdecce19f81510cb5e02949@47.254.24.106:26656,32d0e4aec8d8a8e33273337e1821f2fe2309539a@47.88.58.36:26656,1bf5b73f1771ea84f9974b9f0015186f1daa4266@47.251.14.47:26656"|' $HOME/.artelad/config/config.toml
```

# Set minimum gas price
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "20000000000uart"|' $HOME/.artelad/config/app.toml

# Set pruning
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.artelad/config/app.toml

# Change ports
sed -i -e "s%:1317%:27817%; s%:8080%:27880%; s%:9090%:27890%; s%:9091%:27891%; s%:8545%:27845%; s%:8546%:27846%; s%:6065%:27865%" $HOME/.artelad/config/app.toml
sed -i -e "s%:26658%:27858%; s%:26657%:27857%; s%:6060%:27860%; s%:26656%:27856%; s%:26660%:27861%" $HOME/.artelad/config/config.toml

# Download latest chain data snapshot
curl "https://snapshots-testnet.nodejumper.io/artela-testnet/artela-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.artelad"

# Create a service
sudo tee /etc/systemd/system/artelad.service > /dev/null << EOF
[Unit]
Description=Artela node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which artelad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable artelad.service

# Start the service and check the logs
sudo systemctl start artelad.service
sudo journalctl -u artelad.service -f --no-hostname -o cat
