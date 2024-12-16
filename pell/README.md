
# Install dependencies, if needed

    sudo apt update && sudo apt upgrade -y
    sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y

# Install go, if needed

    cd $HOME
    VER="1.22.3"
    wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
    sudo rm -rf /usr/local/go
    sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
    rm "go$VER.linux-amd64.tar.gz"
    [ ! -f ~/.bash_profile ] && touch ~/.bash_profile
    echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
    source $HOME/.bash_profile
    [ ! -d ~/go/bin ] && mkdir -p ~/go/bin

    echo "export PELL_PORT="58"" >> $HOME/.bash_profile
    source $HOME/.bash_profile

    wget https://github.com/0xPellNetwork/network-config/releases/download/v1.1.1-ignite/pellcored-v1.1.1-linux-amd64

    mv pellcored-v1.1.1-linux-amd64 pellcored && mv pellcored /root/.local/bin

    pellcored init "test" --chain-id ignite_186-1

    wget -O $HOME/.pellcored/config/genesis.json https://raw.githubusercontent.com/0xblocksync/testnet/refs/heads/main/pell/addrbook.json
    wget -O $HOME/.pellcored/config/addrbook.json https://raw.githubusercontent.com/0xblocksync/testnet/refs/heads/main/pell/addrbook.json

# set seeds and peers

    SEEDS="5f10959cc96b5b7f9e08b9720d9a8530c3d08d19@pell-testnet-seed.itrocket.net:58656"
    PEERS="d003cb808ae91bad032bb94d19c922fe094d8556@pell-testnet-peer.itrocket.net:58656,28c0fcd184c31ac7f3e2b3a91ae60dedc086b0c3@94.130.204.227:26656,739d38ce19e4d2b22eb77016f91bf468e93c22af@37.252.186.230:26656,4efd5164f02c3af4247fc0292922af8d08a46ae6@51.89.1.16:26656,d52c32a6a8510bdf0d33909008041b96d95c8408@34.87.39.12:26656,f1049cc2be2902053bcf5ea1a553414d8a978ef6@[2a01:4f8:110:4265::11]:26656,d020ae784e920290fb8fe5cc58ca9139f2fcec57@161.35.27.208:26656,48c48532950e51fba80a1031d5a58c627737ed84@65.109.82.111:25656,a07fb3b45241b774db25f0704a65419f0e98be14@62.171.130.196:26656,80a0c0db6ce1f3f14cae20a09f1d874d987635ba@149.50.96.58:26656,407c7229a207a0c932a56ab3a979fc2312751390@162.19.240.7:28656"
    sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
           -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.pellcored/config/config.toml

# set custom ports in app.toml

    sed -i.bak -e "s%:1317%:${PELL_PORT}317%g;
    s%:8080%:${PELL_PORT}080%g;
    s%:9090%:${PELL_PORT}090%g;
    s%:9091%:${PELL_PORT}091%g;
    s%:8545%:${PELL_PORT}545%g;
    s%:8546%:${PELL_PORT}546%g;
    s%:6065%:${PELL_PORT}065%g" $HOME/.pellcored/config/app.toml

# set custom ports in config.toml file

    sed -i.bak -e "s%:26658%:${PELL_PORT}658%g;
    s%:26657%:${PELL_PORT}657%g;
    s%:6060%:${PELL_PORT}060%g;
    s%:26656%:${PELL_PORT}656%g;
    s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${PELL_PORT}656\"%;
    s%:26660%:${PELL_PORT}660%g" $HOME/.pellcored/config/config.toml

# config pruning

    sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.pellcored/config/app.toml 
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.pellcored/config/app.toml
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"19\"/" $HOME/.pellcored/config/app.toml

# set minimum gas price, enable prometheus and disable indexing

    sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0apell"|g' $HOME/.pellcored/config/app.toml
    sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.pellcored/config/config.toml
    sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.pellcored/config/config.toml

# create service file

    sudo tee /etc/systemd/system/pellcored.service > /dev/null <<EOF
    [Unit]
    Description=Pell node
    After=network-online.target
    [Service]
    User=$USER
    WorkingDirectory=$HOME/.pellcored
    ExecStart=$(which pellcored) start --home $HOME/.pellcored
    Environment=LD_LIBRARY_PATH=$HOME/.pellcored/lib/
    Restart=on-failure
    RestartSec=5
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF

# reset and download snapshot

    pellcored tendermint unsafe-reset-all --home $HOME/.pellcored
    if curl -s --head curl https://server-5.itrocket.net/testnet/pell/pell_2024-12-16_153601_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
      curl https://server-5.itrocket.net/testnet/pell/pell_2024-12-16_153601_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.pellcored
        else
      echo "no snapshot found"
    fi

# enable and start service

    sudo systemctl daemon-reload
    sudo systemctl enable pellcored
    sudo systemctl restart pellcored && sudo journalctl -fu pellcored


# to create a new wallet, use the following command. don’t forget to save the mnemonic

    pellcored keys add $WALLET

# to restore exexuting wallet, use the following command

    pellcored keys add $WALLET --recover

# save wallet and validator address

    WALLET_ADDRESS=$(pellcored keys show $WALLET -a)
    VALOPER_ADDRESS=$(pellcored keys show $WALLET --bech val -a)
    echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
    echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
    source $HOME/.bash_profile

# check sync status, once your node is fully synced, the output from above will print "false"

    pellcored status 2>&1 | jq 

# before creating a validator, you need to fund your wallet and check balance

    pellcored query bank balances $WALLET_ADDRESS


cd $HOME
# Create validator.json file

    echo "{\"pubkey\":{\"@type\":\"/cosmos.crypto.ed25519.PubKey\",\"key\":\"$(pellcored comet show-validator | grep -Po '\"key\":\s*\"\K[^"]*')\"},
        \"amount\": \"1000000pell\",
        \"moniker\": \"<your_moniker>\",
        \"identity\": \"<your_identity>\",
        \"website\": \"<your_website>\",
        \"security\": \"\",
        \"details\": \"Pell Validator\",
        \"commission-rate\": \"0.1\",
        \"commission-max-rate\": \"0.2\",
        \"commission-max-change-rate\": \"0.01\",
        \"min-self-delegation\": \"1\"
    }" > validator.json

# Create a validator using the JSON configuration

    pellcored tx staking create-validator validator.json \
        --from $WALLET \
        --chain-id ignite_186-1 \
    	--gas auto --gas-adjustment 1.5

**#Delete node**

    sudo systemctl stop pellcored
    sudo systemctl disable pellcored
    sudo rm -rf /etc/systemd/system/pellcored.service
    sudo rm $(which pellcored)
    sudo rm -rf $HOME/.pellcored
    sed -i "/PELL_/d" $HOME/.bash_profile
