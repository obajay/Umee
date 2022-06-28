# Umee mainnet guide

    sudo apt update
    
    sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils htop net-tools lsof -y < "/dev/null"

#Go and binaries 

    wget https://golang.org/dl/go1.18.1.linux-amd64.tar.gz; \
    rm -rv /usr/local/go; \
    tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && \
    rm -v go1.18.1.linux-amd64.tar.gz && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
    source ~/.bash_profile && \
    go version

    cd $HOME 
    git clone https://github.com/umee-network/umee.git \
    cd umee \
    git pull \
    git checkout tags/v1.0.3 \
    make build \
    sudo cp $HOME/umee/build/umeed /usr/local/bin \
    umeed version

    umeed init moniker --chain-id umee-1

    wget -O $HOME/.umee/config/genesis.json https://github.com/umee-network/mainnet/raw/main/genesis.json \
    
    jq -S -c -M '' $HOME/.umee/config/genesis.json | shasum -a 256
#### d4264b31472a922ec05620793dd8832a3a78032b23cdaa065f38ff3803f13800

    umeed unsafe-reset-all 
    
### Peers
    peers="06169345b4ae8a59f5132bae78a63733767dc952@51.195.60.123:26656,8b6baf477cd6c5fde18573a57767e0bb0083a8ce@116.202.36.138:26656,f00230b900b2e03a0ebfb0cec024bc0229f4043f@135.181.223.194:26656,31c2b4851604cb0f88909116bc2029b2af392767@194.163.166.56:26656,e324ca5fad08769325921ed042b76bdb1df41e12@162.55.131.220:26656,4720fe172f90026e72723c38d75f4f20611bc792@88.198.70.2:26656,7d2b275cea5dc30a90c9657220b2ef9cf02dfe87@157.90.179.182:26656,d9c0fc2da0bf7b22b92f3cd89b4e98ff089fe446@65.21.132.226:56656,ae41472c094737bef61450c11f1b4978c0a3550d@18.144.151.186:26656,f6b22c8d26370afd0b3e5e78697e19f7a2fb8c73@144.217.74.27:26656,d0659fc256c3e6f99def7a7b16500097065a67e9@195.201.170.172:26656,5ec673b49eea3198f7c0df0782d62e0b7a7d5b9f@51.195.60.117:26656,cce3ded2638edcaf804e4fa18a4a988cd19e9ee1@148.251.152.54:26656,66377bf9c7d2106f8fb2814d105b934e2cf9bde8@78.46.66.6:26656,6dfab3a8a1d692c6270758757cb2026005a10622@65.108.106.252:26656,b7c7e560f13988dc00c6892c813ff6c459521917@44.231.119.182:26656,60349afbb66bfa51d466a1807b6034c8a8446b41@34.215.214.32:26656,96391162797cbdf10982cda8866913be471fbdd4@44.230.43.94:26656,9f86f8acfa46ac5380796328fe0d7daff5038f56@3.37.216.115:26656,629ce04f882462999de6791b0c4010dba5dafaaf@142.132.201.53:26656,77F54319D6F62C17036CA71B3F88365F652BF79F@169.197.142.149:26656,912b7279934187f8c94eacdc21a2e0bdee245eef@54.241.232.181:26656,94a928e1f5ebbc5fae12400c7d8bbdad8b197ad2@52.79.49.253:26656,870c0a786dc941f8ebecd2772c41c014b6cf8899@51.210.118.65:26656,47dd32dc5aa926ff76d8e53a4bc1fcf596cb254c@38.242.205.238:26656,efbcd2de6981fa7f692771e1b845c780c310e2fe@176.9.17.230:26656"
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.umee/config/config.toml 

    echo "[Unit] \
    Description=Umee Node \
    After=network.target 
    
    [Service] \
    User=$USER \
    Type=simple \
    ExecStart=$(which umeed) start \
    Restart=always \
    RestartSec=3 \
    LimitNOFILE=65535 
    
    [Install] \
    WantedBy=multi-user.target" > $HOME/umeed.service \
    sudo mv $HOME/umeed.service /etc/systemd/system \
    sudo tee <<EOF >/dev/null /etc/systemd/journald.conf \
    Storage=persistent \
    EOF 
  
  
    sudo systemctl restart systemd-journald \
    sudo systemctl daemon-reload \
    sudo systemctl enable umeed \
    sudo systemctl restart umeed \
    journalctl -u umeed -f -o cat
  
  
