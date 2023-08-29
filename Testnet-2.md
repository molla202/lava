# LAVA
![image](https://github.com/molla202/lava/assets/91562185/f7fcc08f-b4c5-471f-bf4c-0cf404037e85)


## Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	4|
| RAM	| 8+ GB |
| Storage	| 500 GB SSD |

## Gerekli yazılımları kuralım
```
# install dependencies, if needed
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```
## Go kurulumu
```
# install go, if needed
cd $HOME
! [ -x "$(command -v go)" ] && {
VER="1.20.5"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
}
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```
## Kurulum
Not: cüzdan adı ve moniker kısmını değiştiriniz.
```
# set vars
echo "export WALLET="cüzdan-adı"" >> $HOME/.bash_profile
echo "export MONIKER="adınızı-yazınız"" >> $HOME/.bash_profile
echo "export LAVA_CHAIN_ID="lava-testnet-2"" >> $HOME/.bash_profile
echo "export LAVA_PORT="20"" >> $HOME/.bash_profile
source $HOME/.bash_profile

# download binary
cd $HOME
rm -rf $HOME/lava
git clone https://github.com/lavanet/lava.git
cd lava
git checkout v0.21.1.2
make install

# config and init app
lavad config node tcp://localhost:${LAVA_PORT}657
lavad config keyring-backend os
lavad config chain-id lava-testnet-2
cd $HOME 
curl -s https://raw.githubusercontent.com/lavanet/lava-config/main/testnet-2/genesis_json/genesis.json > $HOME/.lava/config/genesis.json 
curl -Ls https://lava-binary-upgrades.s3.amazonaws.com/testnet-2/cosmovisor-upgrades/cosmovisor-upgrades.zip > cosmovisor-upgrades.zip 
unzip cosmovisor-upgrades.zip 
rm cosmovisor-upgrades.zip 
sed -i \   
-e 's/timeout_commit = ".*"/timeout_commit = "30s"/g' \   
-e 's/timeout_propose = ".*"/timeout_propose = "1s"/g' \  
-e 's/timeout_precommit = ".*"/timeout_precommit = "1s"/g' \   -e 's/timeout_precommit_delta = ".*"/timeout_precommit_delta = "500ms"/g' \   
-e 's/timeout_prevote = ".*"/timeout_prevote = "1s"/g' \   
-e 's/timeout_prevote_delta = ".*"/timeout_prevote_delta = "500ms"/g' \   
-e 's/timeout_propose_delta = ".*"/timeout_propose_delta = "500ms"/g' \   
-e 's/skip_timeout_commit = ".*"/skip_timeout_commit = false/g' \   
-e 's/seeds = ".*"/seeds = "3a445bfdbe2d0c8ee82461633aa3af31bc2b4dc0@testnet2-seed-node.lavanet.xyz:26656,e593c7a9ca61f5616119d6beb5bd8ef5dd28d62d@testnet2-seed-node2.lavanet.xyz:26656"/g' \   $HOME/.lava/config/config.toml 
sed -i -e 's/broadcast-mode = ".*"/broadcast-mode = "sync"/g' $HOME/.lava/config/config.toml 
sudo mv $HOME/cosmovisor-upgrades/genesis/bin/lavad $HOME/go/bin/lavad
lavad init "test" --chain-id lava-testnet-2

# download genesis and addrbook
wget -O $HOME/.lava/config/genesis.json https://testnet-files.itrocket.net/lava/genesis.json
wget -O $HOME/.lava/config/addrbook.json https://testnet-files.itrocket.net/lava/addrbook.json

# set seeds and peers
SEEDS="eb7832932626c1c636d16e0beb49e0e4498fbd5e@lava-testnet-seed.itrocket.net:20656"
PEERS="3693ea5a8a9c0590440a7d6c9a98a022ce3b2455@lava-testnet-peer.itrocket.net:20656,ef1b3374ca00c338de50d51fc41ca317488156eb@207.244.245.41:26656,5c107bb2b72c930a5ab3406a1f7c7345b7229b49@148.251.11.99:11656,0516c4d11552b334a683bdb4410fa22ef7e3f8ba@65.21.239.60:11656,276c73534246fb9ec48d5c72ebd62c42e2f96462@157.90.17.150:26656,1fd86f6ba06ef4b189276f97f70fea04161019db@144.76.176.154:11656,60a144ffed618f4e99466da8fbe7b799b009b9e7@65.108.70.119:38656,57d64cbf5a16820aa9a0582335705f37dde4c18b@190.15.217.229:26656,0064c6e0d31cd4059a49b185aa6048d79db391e1@109.123.241.244:26656,0d08a1b452e6d7ccdfbc9b54658b5f9ed24eff7b@135.181.138.160:29956,5c2a752c9b1952dbed075c56c600c3a79b58c395@185.16.39.172:27066"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.lava/config/config.toml

# set custom ports in app.toml
sed -i.bak -e "s%:1317%:${LAVA_PORT}317%g;
s%:8080%:${LAVA_PORT}080%g;
s%:9090%:${LAVA_PORT}090%g;
s%:9091%:${LAVA_PORT}091%g;
s%:8545%:${LAVA_PORT}545%g;
s%:8546%:${LAVA_PORT}546%g;
s%:6065%:${LAVA_PORT}065%g" $HOME/.lava/config/app.toml

# set custom ports in config.toml file
sed -i.bak -e "s%:26658%:${LAVA_PORT}658%g;
s%:26657%:${LAVA_PORT}657%g;
s%:6060%:${LAVA_PORT}060%g;
s%:26656%:${LAVA_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${LAVA_PORT}656\"%;
s%:26660%:${LAVA_PORT}660%g" $HOME/.lava/config/config.toml

# config pruning
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.lava/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.lava/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.lava/config/app.toml

# set minimum gas price, enable prometheus and disable indexing
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.0ulava"/g' $HOME/.lava/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.lava/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.lava/config/config.toml

# create service file
sudo tee /etc/systemd/system/lavad.service > /dev/null <<EOF
[Unit]
Description=Lava node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.lava
ExecStart=$(which lavad) start --home $HOME/.lava
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF


# enable and start service
sudo systemctl daemon-reload
sudo systemctl enable lavad
sudo systemctl restart lavad && sudo journalctl -u lavad -f
```
