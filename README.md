# Node setup on nibiru



<img width="241" alt="zazaza" src="https://user-images.githubusercontent.com/108979536/199266689-ffd11985-73f4-404c-b1f7-fb48052d79f6.png">



### Nibiru Chain
Nibiru is a sovereign proof-of-stake blockchain, open-source platform, and member of a family of interconnected blockchains that comprise the Cosmos Ecosystem.

Nibiru unifies leveraged derivatives trading, spot trading, staking, and bonded liquidity provision into a seamless user experience, enabling users of over 40 blockchains to trade with leverage using a suite of composable decentralized applications.



# Hardware Requirements

 ### Minimum Hardware Requirements
 
  + 4x CPUs; the faster clock speed the better
   
  + 8GB RAM
  
  + 100GB of storage (SSD or NVME)


 ### Recommended Hardware Requirements
 
  + 8x CPUs; the faster clock speed the better
  
  + 64GB RAM
  
  + 1TB of storage (SSD or NVME)



### 1-ɪɴᴛᴀʟʟᴀᴛɪᴏɴ ᴏɴᴇ ᴛɪᴍᴇ (ꜱᴄʀɪᴘᴛ ᴀᴜᴛᴏᴍᴀᴛɪᴄ ɪɴꜱᴛᴀʟʟᴀᴛɪᴏɴ)

     wget -O nibiru.sh https://raw.githubusercontent.com/appieasahbie/nibiru/main/nibiru.sh && chmod +x nibiru.sh && ./nibiru.sh
     
     
 
### Post installation

        source $HOME/.bash_profile
    
 + (Check the status of your node if is it true type Cltr+c)

        nibid status 2>&1 | jq .SyncInfo
      
 +   Check logs ( to stop checking logs Cltr+c)
 
          journalctl -fu nibid -o cat   
       
       
### Open ports and active the firewall
 
    sudo ufw default allow outgoing
    sudo ufw default deny incoming
    sudo ufw allow ssh/tcp
    sudo ufw limit ssh/tcp
    sudo ufw allow ${NIBIRU_PORT}656,${NIBIRU_PORT}660/tcp
    sudo ufw enable
 
 
 ### (OPTIONAL) State-Sync provided by Nodejumber

      sudo systemctl stop nibid

     cp $HOME/.nibid/data/priv_validator_state.json $HOME/.nibid/priv_validator_state.json.backup
     nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book

     rm -rf $HOME/.nibid/data 

     SNAP_NAME=$(curl -s https://snapshots3-testnet.nodejumper.io/nibiru-testnet/ | egrep -o ">nibiru-testnet-1.*\.tar.lz4" | tr -d ">")
     curl https://snapshots3-testnet.nodejumper.io/nibiru-testnet/${SNAP_NAME} | lz4 -dc - | tar -xf - -C $HOME/.nibid

     mv $HOME/.nibid/priv_validator_state.json.backup $HOME/.nibid/data/priv_validator_state.json

     sudo systemctl restart nibid
     sudo journalctl -u nibid -f --no-hostname -o cat
    
    

### Update block time parameters

    CONFIG_TOML="$HOME/.nibid/config/config.toml"
    sed -i 's/timeout_propose =.*/timeout_propose = "100ms"/g' $CONFIG_TOML
    sed -i 's/timeout_propose_delta =.*/timeout_propose_delta = "500ms"/g' $CONFIG_TOML
    sed -i 's/timeout_prevote =.*/timeout_prevote = "100ms"/g' $CONFIG_TOML
    sed -i 's/timeout_prevote_delta =.*/timeout_prevote_delta = "500ms"/g' $CONFIG_TOML
    sed -i 's/timeout_precommit =.*/timeout_precommit = "100ms"/g' $CONFIG_TOML
    sed -i 's/timeout_precommit_delta =.*/timeout_precommit_delta = "500ms"/g' $CONFIG_TOML
    sed -i 's/timeout_commit =.*/timeout_commit = "1s"/g' $CONFIG_TOML
    sed -i 's/skip_timeout_commit =.*/skip_timeout_commit = false/g' $CONFIG_TOML


### Create wallet

 + (Please save all keys on your notepad)

     nibid keys add $WALLET
     
     
### To recover your old wallet use this command

     nibid keys add $WALLET --recover
 
 
### To get current list of wallets

    nibid keys list

### Add wallet and valoper address and load variables into the system

    NIBIRU_WALLET_ADDRESS=$(nibid keys show $WALLET -a)
    NIBIRU_VALOPER_ADDRESS=$(nibid keys show $WALLET --bech val -a)
    echo 'export NIBIRU_WALLET_ADDRESS='${NIBIRU_WALLET_ADDRESS} >> $HOME/.bash_profile
    echo 'export NIBIRU_VALOPER_ADDRESS='${NIBIRU_VALOPER_ADDRESS} >> $HOME/.bash_profile
    source $HOME/.bash_profile
 
 
 ### Fund your wallet

     curl -X POST -d '{"address": "'"$NIBIRU_WALLET_ADDRESS"'", "coins": ["10000000unibi","100000000000unusd"]}' https://faucet.testnet-1.nibiru.fi/
 
 
 ### Create validator
   +(first check your bank balance )
 
      nibid query bank balances $NIBIRU_WALLET_ADDRESS
      
      
 And
 
 
      nibid tx staking create-validator \
      --amount 10000000unibi \
      --commission-max-change-rate "0.1" \
      --commission-max-rate "0.20" \
      --commission-rate "0.1" \
      --min-self-delegation "1" \
      --details "put your validator description there" \
      --pubkey=$(nibid tendermint show-validator) \
      --moniker <your_moniker> \
      --chain-id nibiru-testnet-1 \
      --gas-prices 0.025unibi \
      --from <key-name>

 
 

### Service management
  + Check logs

        journalctl -fu nibid -o cat
     
     
### Start service

     sudo systemctl start nibid
     
### Stop service

     sudo systemctl stop nibid
     
     
### Restart service

     sudo systemctl restart nibid
     

### Synchronization info

     nibid status 2>&1 | jq .SyncInfo
     
### Validator info

     nibid status 2>&1 | jq .ValidatorInfo
     
### Node info

     nibid status 2>&1 | jq .NodeInfo
     
### Show node id

     nibid tendermint show-node-id
     

### Voting

     nibid tx gov vote 1 yes --from $WALLET --chain-id=$NIBIRU_CHAIN_ID

### Delegate stake

     nibid tx staking delegate $NIBIRU_VALOPER_ADDRESS 10000000unibi --from=$WALLET --chain-id=$NIBIRU_CHAIN_ID --gas=auto
     
### Redelegate stake from validator to another validator

     nibid tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000unibi --from=$WALLET --chain-id=$NIBIRU_CHAIN_ID --gas=auto
     
     
### Unjail validator

      nibid tx slashing unjail \
      --broadcast-mode=block \
      --from=$WALLET \
      --chain-id=$NIBIRU_CHAIN_ID \
      --gas=auto     
 
 
### Delete node

     sudo systemctl stop nibid
     sudo systemctl disable nibid
     sudo rm /etc/systemd/system/nibi* -rf
     sudo rm $(which nibid) -rf
     sudo rm $HOME/.nibid* -rf
     sudo rm $HOME/nibiru -rf
     sed -i '/NIBIRU_/d' ~/.bash_profile
 
 
 [buy me cup of coffee ](https://www.paypal.com/paypalme/AbdelAkridi?country.x=NL&locale.x=en_US)
