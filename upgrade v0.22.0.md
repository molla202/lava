```
cd $HOME
wget -O lavad https://github.com/lavanet/lava/releases/download/v0.22.0/lavad-v0.22.0-linux-amd64
chmod +x $HOME/lavad
sudo mv $HOME/lavad $(which lavad)
sudo systemctl restart lavad && sudo journalctl -u lavad -f
```
