# lava

## Go kurulumu
```
cd $HOME
VER="1.19.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm -rf  "go$VER.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```
## dosyaları çekiyoruz
```
git clone https://github.com/K433QLtr6RA9ExEq/GHFkqmTzpdNLDd6T.git
cd GHFkqmTzpdNLDd6T/testnet-1
source setup_config/setup_config.sh
```
## Config yapılandırıyoruz
```
echo "Lava config file path: $lava_config_folder"
mkdir -p $lavad_home_folder
mkdir -p $lava_config_folder
cp default_lavad_config_files/* $lava_config_folder
```
## Genesis
```
cp genesis_json/genesis.json $lava_config_folder/genesis.json
```
## Genesis binary
```
lavad_binary_path="$HOME/go/bin/"
mkdir -p $lavad_binary_path
wget https://lava-binary-upgrades.s3.amazonaws.com/testnet/v0.3.0/lavad
chmod +x lavad
```
## Servis dosyası yapılandırma
```
echo "[Unit]
Description=Lava Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which lavad) start --home=$lavad_home_folder --p2p.seeds $seed_node
Restart=always
RestartSec=180
LimitNOFILE=infinity
LimitNPROC=infinity
[Install]
WantedBy=multi-user.target" >lavad.service
sudo mv lavad.service /lib/systemd/system/lavad.service
```
## Servis başlatıyoruz
```
sudo systemctl daemon-reload
sudo systemctl enable lavad.service
sudo systemctl restart systemd-journald
sudo systemctl start lavad
```
## Logları kontrol ediyoruz.
```
sudo journalctl -u lavad -f -o cat
```
