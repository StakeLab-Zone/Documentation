# Interact with a Cosmos RPC node
Through this guide you gonna learn how to interact with a Cosmos node, from basic query to some advanced features using RPC.  
Remote procedure calls (RPCs) are frequently used in distributed computing and allow for inter-process communication (IPC). In most crypto ecosystem, there is public node (endpoint) existing, giving read/write access, that let you try for low amount of queries on it for free.  
On Cosmos, here are the few endpoints to remember:  
- **gRPC:** ```PORT:``` 9090
- **RPC:** ```PORT:``` 26657
- **Rest:** ```PORT:``` 1317

## Difference between Ethereum and Cosmos RPC's
There is no major differenciation between both of them.  
We could only see some delta if you are looking for an historic of all transaction of a specific account, and even on a specific module. In this case, in Ethereum side it would require an indexer, when it's way easier to get it from basic query using Cosmos RPC.  

## Summary
### Optionnal - Only if you wanna use your own endpoint in case of lot of queries
1. Prepare your machine
  - [Install the prerequies](https://github.com/StakeLab-Hub/Documentation/blob/main/CLI/Interact%20with%20a%20node/README.md#Install-the-prerequies)
2. Run your full node
  - [Install the binary](https://github.com/StakeLab-Hub/Documentation/blob/main/CLI/Interact%20with%20a%20node/README.md#Install-the-binary)
  - [Configure your network files](https://github.com/StakeLab-Hub/Documentation/blob/main/CLI/Interact%20with%20a%20node/README.md#Configure-your-network-files)
  - [Launch the chain](https://github.com/StakeLab-Hub/Documentation/blob/main/CLI/Interact%20with%20a%20node/README.md#Launch-the-chain)
  
### You can start here if you don't wanna run your own node
3. Basic query using RPC (CLI)
  - [Query a balance](https://github.com/StakeLab-Hub/Documentation/blob/main/CLI/Interact%20with%20a%20node/README.md#Query-a-balance)
  - []()
4. Advanced query using RPC (CLI)
  - []() 
5. Basic query using RPC (JavaScript)
  - []()
6. Advanced query using RPC (Javascript)
  - []()

## Prepare your machine
### Install the prerequies
*  Linux Machine 20.4+  
*  Install few packages:  
```shell
apt update && apt upgrade -y 
```  

```shell
apt install build-essential git ufw curl jq unzip lz4 aria2 -y
```

```shell
wget -c https://go.dev/dl/go1.19.3.linux-amd64.tar.gz && rm -rf /usr/local/go && tar -C /usr/local -xzf go1.19.3.linux-amd64.tar.gz && rm -rf go1.19.3.linux-amd64.tar.gz
```  

```shell
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
```  
You can verify if it's well installed by using the following command:  
```shell
go version
#Should return: go version go1.19.3 linux/amd64
```  

## Run your full node
### Install the binary
Clone the repository on your ```$HOME``` to only have to ```git pull``` for future upgrade of binary and ```git checkout``` the last version:
```shell
git clone https://github.com/cosmos/gaia && cd gaia && git checkout v7.1.0 && make install
```  
You can verify if it's well installed by using the follow command:  
```shell
gaiad version
#Should return: v7.1.0
```  
You can now init the chain to create the ```.gaia```folder that will have everything related to your Blockchain (data, config, keys...):  
```shell
gaiad init <nameofyournode> --chain-id cosmoshub-4
```  
Replace ```<nameofyournode>```by the name you want to have your node appears on the network (public).  

### Configure your network files  
Depending on your usage, you will either choose to run one of the three below nodes. We will use a ```snapshot``` to synch faster with the chain:   
- **Archive node:** all historic states will be saved, nothing will be deleted (High storage requirement - 4TB size)  
```shell
URL=`curl -L https://quicksync.io/cosmos.json|jq -r '.[] |select(.file=="cosmoshub-4-archive")|.url'` && cd $HOME/.gaia && aria2c -x5 $URL
```  
- **Full node:** the last 100 states are kept in addition to every 500th state; pruning at 10 block intervals (Average storage requirement - 2TB size)  
```shell
URL=`curl -L https://quicksync.io/cosmos.json|jq -r '.[] |select(.file=="cosmoshub-4-default")|.url'` && cd $HOME/.gaia && aria2c -x5 $URL
```  
- **Pruned node:** determine with your own custom values (Low storage requirement - 50GB size)  
```shell
URL=`curl -L https://quicksync.io/cosmos.json|jq -r '.[] |select(.file=="cosmoshub-4-pruned")|.url'` && cd $HOME/.gaia && aria2c -x5 $URL
```  
Once you have your data folder loaded, you can change some files value in the config folder:  
- **app.toml:** 
- **config.toml:** 

### Launch the chain
Once everything is setup on the configuration side, you can run using different way the Blockchain:  
- **CLI command (prefered to launch on a different screen):**  
```shell
gaiad start
```  
To stop the Blockchain, you just have to press ```CTRL```+```C```  

- **Systemctl service:**  
Create the service file by copy/paste the below command:  
```shell
tee <<EOF >/dev/null /etc/systemd/system/gaiad.service
[Unit]
Description=Teritori Cosmos daemon
After=network-online.target

[Service]
User=$USER
ExecStart=/home/$USER/go/bin/gaiad start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```  
Once done, you just have to copy/past the three below commands to enable, load and launch your node:  
```shell
systemctl enable gaiad
systemctl daemon-reload
systemctl restart gaiad
```  
As it's running the back, you can check the log using:  
```shell
journalctl -u gaiad.service -f -n 100
```  
To stop the Blockchain, you can run:  
```shell
systemctl stop gaiad
```  

- **Cosmovisor (Recommended):**  
Install Cosmovisor on your machine:  
```shell
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@latest
```  
Source your profile with the daemon configuration:  
```shell
export DAEMON_NAME=gaiad
export DAEMON_HOME=$HOME/.gaia
source ~/.profile
```  
Create the folders to store future binaries for future upgrades:  
```shell
mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
mkdir -p $DAEMON_HOME/cosmovisor/upgrades
```  
Copy the actual binary into Cosmovisor folder:  
```shell
cp $HOME/go/bin/gaiad $DAEMON_HOME/cosmovisor/genesis/bin
```  
Create the service file by copy/paste the below command:  
```shell
tee <<EOF >/dev/null /etc/systemd/system/gaiad.service
[Unit]
Description=Cosmos Daemon (cosmovisor)

After=network-online.target

[Service]
User=$USER
ExecStart=/home/$USER/go/bin/cosmovisor run start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=gaiad"
Environment="DAEMON_HOME=/home/$USER/.gaia"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"

[Install]
WantedBy=multi-user.target
EOF
```  
Once done, you just have to copy/past the three below commands to enable, load and launch your node:  
```shell
systemctl enable gaiad
systemctl daemon-reload
systemctl restart gaiad
```  
As it's running the back, you can check the log using:  
```shell
journalctl -u gaiad.service -f -n 100
```  
To stop the Blockchain, you can run:  
```shell
cosmovisor run stop
```  

You can verify if your chain is synched with the last block by using the below command:  
```shell
gaiad status
#Should return the following result if synched: "catching_up":false
```  

## Basic query using RPC (CLI)
### Query a balance

## Advanced query using RPC (CLI)
### 

## Basic query using RPC (JavaScript)
### Query a balance
To first start using JavaScript for your RPC queries, you will need to install CosmJS:  
```shell
npm install @cosmjs/proto-signing @cosmjs/stargate
# or
yarn add @cosmjs/proto-signing @cosmjs/stargate
```  

Once done, you should be able to use a basic ```getBalance``` query to retrieve the balance of a determinated account:  
```shell
const balance = getBalance(
    "uatom",
    "the cosmos1 address you wanna query",
    "public endpoint or the one you created that should be on http://localhost:26657 by default"
 );
 ```  
 
See the below example format:  

```shell
const balance = getBalance(
    "uatom",
    "cosmos1gpx52r9h3zeul45amvcy2pysgvcwddxrdjwdll",
    "https://cosmos-mainnet-rpc.allthatnode.com:26657/"
 );
 ```  
 
## Advanced query using RPC (JavaScript)
### 
