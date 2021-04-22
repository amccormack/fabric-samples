# README

## About

This code is a very simple demonstration of custom smart contract code. Most of the files in the `asset-transfer-academic` folder are direct copies of `asset-transfer-basic` and can probably be ignored, and possibly deleted. 

Although normally excluded from the git repo, I included the `bin` and `config` folders in the `fabric-samples` folder. The binaries are for x64 linux installations.

## Tutorial

### Setting up the environment

We're basing this off the [getting started doc][1] that fabric documents provides. I assume you have git, docker, jq, and go installed. If not, check out the [prereqs page][2]. I'm running Ubuntu 20.04.2 LTS.

The fabric installation docs suggest cloning the fabric-sample docs into `$HOME/go/src/github.com/<your_github_userid>` - I'll be using a github username of `scrappythebird` as an example.

#### Commands only:

```bash
mkdir -p $HOME/go/src/github.com/scrappythebird
cd $HOME/go/src/github.com/scrappythebird
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.3.1 1.5.0
cd fabric-samples/
git remote add amccormack https://github.com/amccormack/fabric-samples.git
git fetch amccormack
git checkout academic-poc
```

#### Commands with results / explanation:

Create a directory to store the `fabric-samples` repo in. I'm pretending my github username is `scrappythebird` and that I am creating a fork of the `fabric-samples` repo. Use your own github username instead of `scrappythebird`.


```bash
almac@green:~$ mkdir -p $HOME/go/src/github.com/scrappythebird
almac@green:~$ cd $HOME/go/src/github.com/scrappythebird
```


Hyperledger provides a script which will pull down the fabric-samples repo, hyperledger fabric binaries and docker images. We specify the version of Fabric and Fabric CA to make this a bit more repeatable.

```bash
almac@green:~/go/src/github.com/scrappythebird$ curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.3.1 1.5.0
... redacted for length ...
hyperledger/fabric-baseos   2.3.1     fb85a21d6642   2 months ago   6.85MB             
hyperledger/fabric-ca       1.5       24a7c19a9fd8   6 weeks ago    70.8MB
```

The custom chaincode that I adopted for this demo is in my fork on github in a branch called `academic-poc`. The easiest way to get that code is to add my repo as a remote and then fetch the repo and checkout the branch. 
```bash
almac@green:~/go/src/github.com/scrappythebird$ cd fabric-samples/
almac@green:~/go/src/github.com/scrappythebird/fabric-samples$ git remote add amccormack https://github.com/amccormack/fabric-samples.git
almac@green:~/go/src/github.com/scrappythebird/fabric-samples$ git fetch amccormack
remote: Enumerating objects: 25, done.
remote: Counting objects: 100% (18/18), done.
remote: Compressing objects: 100% (16/16), done.
remote: Total 25 (delta 2), reused 14 (delta 2), pack-reused 7
Unpacking objects: 100% (25/25), 103.25 MiB | 11.89 MiB/s, done.
From https://github.com/amccormack/fabric-samples
 * [new branch]      academic-poc -> amccormack/academic-poc
 * [new branch]      main         -> amccormack/main
 * [new branch]      release      -> amccormack/release
...
almac@green:~/go/src/github.com/scrappythebird/fabric-samples$ git checkout academic-poc
Branch 'academic-poc' set up to track remote branch 'academic-poc' from 'amccormack'.
Switched to a new branch 'academic-poc'
```

### Running the tutorial

#### Commands only

 ```bash
cd ~/go/src/github.com/scrappythebird/fabric-samples
cd test-network/
./network.sh down # Ensure no previous runs are still around
./network.sh up   # Set up the network
./network.sh createChannel # Create the channel
# Compile and deploy the chaincode
./network.sh deployCC -ccn basic-academic -ccp ../asset-transfer-academic/chaincode-go -ccl go
# Set a bunch of environment variables to act as Org1MSP
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
# Init the ledger, notice that multiple certs are involved
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic-academic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'
# Validate that there are no assets yet
peer chaincode query -C mychannel -n basic-academic -c '{"Args":["GetAllAssets"]}'
# Create an initial asset with dummy data
peer chaincode invoke -o localhost:7050         --ordererTLSHostnameOverride orderer.example.com --tls  --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"      -C mychannel -n basic-academic --peerAddresses localhost:7051   --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"         --peerAddresses localhost:9051  --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt"    -c '{"function":"CreateAsset","Args":["rid01","owner0","http://example.com/data.json","deadbeef","sum","55"]}'
# Validate it exists
peer chaincode query -C mychannel -n basic-academic -c '{"Args":["GetAllAssets"]}'
# Add an endorser of "owner2"
peer chaincode invoke -o localhost:7050         --ordererTLSHostnameOverride orderer.example.com --tls  --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"      -C mychannel -n basic-academic --peerAddresses localhost:7051   --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"         --peerAddresses localhost:9051  --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt"    -c '{"function":"EndorseAsset","Args":["rid01","owner2"]}'
# See that owner2 was added as an endorser
peer chaincode query -C mychannel -n basic-academic -c '{"Args":["GetAllAssets"]}'
# Add owner3 as an edorser. Note that no owner3 key is involved.
peer chaincode invoke -o localhost:7050         --ordererTLSHostnameOverride orderer.example.com --tls  --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"      -C mychannel -n basic-academic --peerAddresses localhost:7051   --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"         --peerAddresses localhost:9051  --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt"    -c '{"function":"EndorseAsset","Args":["rid01","owner3"]}'
peer chaincode query -C mychannel -n basic-academic -c '{"Args":["GetAllAssets"]}'
 ```

The full output of what these commands looks like can be found below.






## Resources

 - Most of this was developed using this [tutorial](https://hyperledger-fabric.readthedocs.io/en/latest/test_network.html).

[1]: https://hyperledger-fabric.readthedocs.io/en/latest/install.html
[2]: https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html



# Interacting with the network with responses

We shutdown the network. Your output may look a little different depending on if you've run this test before.

```bash
almac@green:~/go/src/github.com/scrappythebird/fabric-samples$ cd test-network/
almac@green:~/go/src/github.com/scrappythebird/fabric-samples/test-network$ ./network.sh down
Stopping network
Removing network fabric_test
WARNING: Network fabric_test not found.
Removing volume docker_orderer.example.com
WARNING: Volume docker_orderer.example.com not found.
Removing volume docker_peer0.org1.example.com
WARNING: Volume docker_peer0.org1.example.com not found.
Removing volume docker_peer0.org2.example.com
WARNING: Volume docker_peer0.org2.example.com not found.
Removing network fabric_test
WARNING: Network fabric_test not found.
Removing volume docker_peer0.org3.example.com
WARNING: Volume docker_peer0.org3.example.com not found.
Removing remaining containers
Removing generated chaincode docker images
```
Start the network with `up`. 
```bash
almac@green:~/go/src/github.com/scrappythebird/fabric-samples/test-network$ ./network.sh up
Starting nodes with CLI timeout of '5' tries and CLI delay of '3' seconds and using database 'leveldb' with crypto from 'cryptogen'
LOCAL_VERSION=2.3.1
DOCKER_IMAGE_VERSION=2.3.1
/home/almac/go/src/github.com/scrappythebird/fabric-samples/test-network/../bin/cryptogen
Generating certificates using cryptogen tool
Creating Org1 Identities
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-org1.yaml --output=organizations
org1.example.com
+ res=0
Creating Org2 Identities
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-org2.yaml --output=organizations
org2.example.com
+ res=0
Creating Orderer Org Identities
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-orderer.yaml --output=organizations
+ res=0
Generating CCP files for Org1 and Org2
Creating network "fabric_test" with the default driver
Creating volume "docker_orderer.example.com" with default driver
Creating volume "docker_peer0.org1.example.com" with default driver
Creating volume "docker_peer0.org2.example.com" with default driver
Creating peer0.org1.example.com ... done
Creating orderer.example.com    ... done
Creating peer0.org2.example.com ... done
Creating cli                    ... done
CONTAINER ID   IMAGE                               COMMAND             CREATED         STATUS                  PORTS                                            NAMES
4e1aff8b19d3   hyperledger/fabric-tools:latest     "/bin/bash"         1 second ago    Up Less than a second                                                    cli
2a4f9d8252ba   hyperledger/fabric-peer:latest      "peer node start"   2 seconds ago   Up Less than a second   7051/tcp, 0.0.0.0:9051->9051/tcp                 peer0.org2.example.com
7fd70c96b6d3   hyperledger/fabric-orderer:latest   "orderer"           2 seconds ago   Up Less than a second   0.0.0.0:7050->7050/tcp, 0.0.0.0:7053->7053/tcp   orderer.example.com
47b2c8f897fd   hyperledger/fabric-peer:latest      "peer node start"   2 seconds ago   Up Less than a second   0.0.0.0:7051->7051/tcp                           peer0.org1.example.com
```
Create a channel on the network called `mychannel`.  The calls to configtxgen and similar binaries are being made by `network.sh`. We only execute one command in the codeblock. 
```bash
almac@green:~/go/src/github.com/scrappythebird/fabric-samples/test-network$ ./network.sh createChannel
Creating channel 'mychannel'.
If network is not up, starting nodes with CLI timeout of '5' tries and CLI delay of '3' seconds and using database 'leveldb
Generating channel genesis block 'mychannel.block'
/home/almac/go/src/github.com/scrappythebird/fabric-samples/test-network/../bin/configtxgen
+ configtxgen -profile TwoOrgsApplicationGenesis -outputBlock ./channel-artifacts/mychannel.block -channelID mychannel
2021-04-21 18:39:31.842 EDT [common.tools.configtxgen] main -> INFO 001 Loading configuration
2021-04-21 18:39:31.846 EDT [common.tools.configtxgen.localconfig] completeInitialization -> INFO 002 orderer type: etcdraft
2021-04-21 18:39:31.846 EDT [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 Orderer.EtcdRaft.Options unset, setting to tick_interval:"500ms" election_tick:10 heartbeat_tick:1 max_inflight_blocks:5 snapshot_interval_size:16777216
2021-04-21 18:39:31.846 EDT [common.tools.configtxgen.localconfig] Load -> INFO 004 Loaded configuration: /home/almac/go/src/github.com/scrappythebird/fabric-samples/test-network/configtx/configtx.yaml
2021-04-21 18:39:31.847 EDT [common.tools.configtxgen] doOutputBlock -> INFO 005 Generating genesis block
2021-04-21 18:39:31.847 EDT [common.tools.configtxgen] doOutputBlock -> INFO 006 Creating application channel genesis block
2021-04-21 18:39:31.847 EDT [common.tools.configtxgen] doOutputBlock -> INFO 007 Writing genesis block
+ res=0
Creating channel mychannel
Using organization 1
+ osnadmin channel join --channelID mychannel --config-block ./channel-artifacts/mychannel.block -o localhost:7053 --ca-file /home/almac/go/src/github.com/scrappythebird/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --client-cert /home/almac/go/src/github.com/scrappythebird/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt --client-key /home/almac/go/src/github.com/scrappythebird/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key
+ res=0
Status: 201
{
        "name": "mychannel",
        "url": "/participation/v1/channels/mychannel",
        "consensusRelation": "consenter",
        "status": "active",
        "height": 1
}

Channel 'mychannel' created
Joining org1 peer to the channel...
Using organization 1
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=0
2021-04-21 18:39:37.965 EDT [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2021-04-21 18:39:37.991 EDT [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
Joining org2 peer to the channel...
Using organization 2
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=0
2021-04-21 18:39:41.036 EDT [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2021-04-21 18:39:41.064 EDT [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
Setting anchor peer for org1...
Using organization 1
Fetching channel config for channel mychannel
Using organization 1
Fetching the most recent configuration block for the channel
+ peer channel fetch config config_block.pb -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
2021-04-21 22:39:41.172 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2021-04-21 22:39:41.174 UTC [cli.common] readBlock -> INFO 002 Received block: 0
2021-04-21 22:39:41.174 UTC [channelCmd] fetch -> INFO 003 Retrieving last config block: 0
2021-04-21 22:39:41.175 UTC [cli.common] readBlock -> INFO 004 Received block: 0
Decoding config block to JSON and isolating config to Org1MSPconfig.json
+ configtxlator proto_decode --input config_block.pb --type common.Block
+ jq '.data.data[0].payload.data.config'
Generating anchor peer update transaction for Org1 on channel mychannel
+ jq '.channel_group.groups.Application.groups.Org1MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org1.example.com","port": 7051}]},"version": "0"}}' Org1MSPconfig.json
+ configtxlator proto_encode --input Org1MSPconfig.json --type common.Config
+ configtxlator proto_encode --input Org1MSPmodified_config.json --type common.Config
+ configtxlator compute_update --channel_id mychannel --original original_config.pb --updated modified_config.pb
+ configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate
+ jq .
++ cat config_update.json
+ echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":{' '"channel_id":' '"mychannel",' '"isolated_data":' '{},' '"read_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org1MSP":' '{' '"groups":' '{},' '"mod_policy":' '"",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '},' '"write_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org1MSP":' '{' '"groups":' '{},' '"mod_policy":' '"Admins",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"AnchorPeers":' '{' '"mod_policy":' '"Admins",' '"value":' '{' '"anchor_peers":' '[' '{' '"host":' '"peer0.org1.example.com",' '"port":' 7051 '}' ']' '},' '"version":' '"0"' '},' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"1"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '}}}}'
+ configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope
2021-04-21 22:39:41.321 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2021-04-21 22:39:41.331 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update
Anchor peer set for org 'Org1MSP' on channel 'mychannel'
Setting anchor peer for org2...
Using organization 2
Fetching channel config for channel mychannel
Using organization 2
Fetching the most recent configuration block for the channel
+ peer channel fetch config config_block.pb -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
2021-04-21 22:39:41.438 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2021-04-21 22:39:41.440 UTC [cli.common] readBlock -> INFO 002 Received block: 1
2021-04-21 22:39:41.440 UTC [channelCmd] fetch -> INFO 003 Retrieving last config block: 1
2021-04-21 22:39:41.441 UTC [cli.common] readBlock -> INFO 004 Received block: 1
Decoding config block to JSON and isolating config to Org2MSPconfig.json
+ configtxlator proto_decode --input config_block.pb --type common.Block
+ jq '.data.data[0].payload.data.config'
Generating anchor peer update transaction for Org2 on channel mychannel
+ jq '.channel_group.groups.Application.groups.Org2MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org2.example.com","port": 9051}]},"version": "0"}}' Org2MSPconfig.json
+ configtxlator proto_encode --input Org2MSPconfig.json --type common.Config
+ configtxlator proto_encode --input Org2MSPmodified_config.json --type common.Config
+ configtxlator compute_update --channel_id mychannel --original original_config.pb --updated modified_config.pb
+ configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate
+ ++ jq .
cat config_update.json
+ echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":{' '"channel_id":' '"mychannel",' '"isolated_data":' '{},' '"read_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org2MSP":' '{' '"groups":' '{},' '"mod_policy":' '"",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '},' '"write_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org2MSP":' '{' '"groups":' '{},' '"mod_policy":' '"Admins",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"AnchorPeers":' '{' '"mod_policy":' '"Admins",' '"value":' '{' '"anchor_peers":' '[' '{' '"host":' '"peer0.org2.example.com",' '"port":' 9051 '}' ']' '},' '"version":' '"0"' '},' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"1"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '}}}}'
+ configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope
2021-04-21 22:39:41.578 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2021-04-21 22:39:41.588 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update
Anchor peer set for org 'Org2MSP' on channel 'mychannel'
Channel 'mychannel' joined
```
Now we compile and deploy the custom chaincode we created for tracking and endorsing assets. This code is located in the `asset-transfer-academic/chaincode-go` directory.
```bash
almac@green:~/go/src/github.com/scrappythebird/fabric-samples/test-network$ ./network.sh deployCC -ccn basic-academic -ccp ../asset-transfer-academic/chaincode-go -ccl go
deploying chaincode on channel 'mychannel'
executing with the following
- CHANNEL_NAME: mychannel
- CC_NAME: basic-academic
- CC_SRC_PATH: ../asset-transfer-academic/chaincode-go
- CC_SRC_LANGUAGE: go
- CC_VERSION: 1.0
- CC_SEQUENCE: 1
- CC_END_POLICY: NA
- CC_COLL_CONFIG: NA
- CC_INIT_FCN: NA
- DELAY: 3
- MAX_RETRY: 5
- VERBOSE: false
Vendoring Go dependencies at ../asset-transfer-academic/chaincode-go
~/go/src/github.com/scrappythebird/fabric-samples/asset-transfer-academic/chaincode-go ~/go/src/github.com/scrappythebird/fabric-samples/test-network
~/go/src/github.com/scrappythebird/fabric-samples/test-network
Finished vendoring Go dependencies
+ peer lifecycle chaincode package basic-academic.tar.gz --path ../asset-transfer-academic/chaincode-go --lang golang --label basic-academic_1.0
+ res=0
Chaincode is packaged
Installing chaincode on peer0.org1...
Using organization 1
+ peer lifecycle chaincode install basic-academic.tar.gz
+ res=0
2021-04-21 18:41:08.541 EDT [cli.lifecycle.chaincode] submitInstallProposal -> INFO 001 Installed remotely: response:<status:200 payload:"\nSbasic-academic_1.0:945abc5879edefedeb1d155a6627c40d25bb7bd73d6ad1d4d82a5ac2e6b71f8b\022\022basic-academic_1.0" >
2021-04-21 18:41:08.542 EDT [cli.lifecycle.chaincode] submitInstallProposal -> INFO 002 Chaincode code package identifier: basic-academic_1.0:945abc5879edefedeb1d155a6627c40d25bb7bd73d6ad1d4d82a5ac2e6b71f8b
Chaincode is installed on peer0.org1
Install chaincode on peer0.org2...
Using organization 2
+ peer lifecycle chaincode install basic-academic.tar.gz
+ res=0
2021-04-21 18:41:13.633 EDT [cli.lifecycle.chaincode] submitInstallProposal -> INFO 001 Installed remotely: response:<status:200 payload:"\nSbasic-academic_1.0:945abc5879edefedeb1d155a6627c40d25bb7bd73d6ad1d4d82a5ac2e6b71f8b\022\022basic-academic_1.0" >
2021-04-21 18:41:13.633 EDT [cli.lifecycle.chaincode] submitInstallProposal -> INFO 002 Chaincode code package identifier: basic-academic_1.0:945abc5879edefedeb1d155a6627c40d25bb7bd73d6ad1d4d82a5ac2e6b71f8b
Chaincode is installed on peer0.org2
Using organization 1
+ peer lifecycle chaincode queryinstalled
+ res=0
Installed chaincodes on peer:
Package ID: basic-academic_1.0:945abc5879edefedeb1d155a6627c40d25bb7bd73d6ad1d4d82a5ac2e6b71f8b, Label: basic-academic_1.0
Query installed successful on peer0.org1 on channel
Using organization 1
+ peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/almac/go/src/github.com/scrappythebird/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --channelID mychannel --name basic-academic --version 1.0 --package-id basic-academic_1.0:945abc5879edefedeb1d155a6627c40d25bb7bd73d6ad1d4d82a5ac2e6b71f8b --sequence 1
+ res=0
2021-04-21 18:41:15.731 EDT [chaincodeCmd] ClientWait -> INFO 001 txid [c6af5fa7bef68bf959fc93238dc5946e57348122ef243b91959c88258e4bd3c7] committed with status (VALID) at localhost:7051
Chaincode definition approved on peer0.org1 on channel 'mychannel'
Using organization 1
Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic-academic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": false
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Checking the commit readiness of the chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic-academic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": false
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 2
+ peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/almac/go/src/github.com/scrappythebird/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --channelID mychannel --name basic-academic --version 1.0 --package-id basic-academic_1.0:945abc5879edefedeb1d155a6627c40d25bb7bd73d6ad1d4d82a5ac2e6b71f8b --sequence 1
+ res=0
2021-04-21 18:41:23.883 EDT [chaincodeCmd] ClientWait -> INFO 001 txid [87da5b6719e20a6496ef2f88679f872cbd3fa302f20fff239f3e500052a7bfc1] committed with status (VALID) at localhost:9051
Chaincode definition approved on peer0.org2 on channel 'mychannel'
Using organization 1
Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic-academic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": true
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Checking the commit readiness of the chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic-academic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": true
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 1
Using organization 2
+ peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/almac/go/src/github.com/scrappythebird/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --channelID mychannel --name basic-academic --peerAddresses localhost:7051 --tlsRootCertFiles /home/almac/go/src/github.com/scrappythebird/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles /home/almac/go/src/github.com/scrappythebird/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt --version 1.0 --sequence 1
+ res=0
2021-04-21 18:41:32.143 EDT [chaincodeCmd] ClientWait -> INFO 001 txid [d0842a825295f575673d42edb24614b12380065044f8d7bc272f673134d4969e] committed with status (VALID) at localhost:7051
2021-04-21 18:41:32.143 EDT [chaincodeCmd] ClientWait -> INFO 002 txid [d0842a825295f575673d42edb24614b12380065044f8d7bc272f673134d4969e] committed with status (VALID) at localhost:9051
Chaincode definition committed on channel 'mychannel'
Using organization 1
Querying chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to Query committed status on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode querycommitted --channelID mychannel --name basic-academic
+ res=0
Committed chaincode definition for chaincode 'basic-academic' on channel 'mychannel':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2

peer chaincode invoke -o localhost:7050 \
Querying chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to Query committed status on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode querycommitted --channelID mychannel --name basic-academic
+ res=0
Committed chaincode definition for chaincode 'basic-academic' on channel 'mychannel':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org2 on channel 'mychannel'
Chaincode initialization is not required
```

Now that we have set up the initial `basic-academic` chaincode, we set up our environment variables so we can act as `Org1MSP`. This is most apparent when we do things like `GetAllAssets`, when we do not provide the credentials for the other organization`. 

The following code is executed in `~/go/src/github.com/scrappythebird/fabric-samples/test-network` but the path is removed for visibility.
```bash
$ export PATH=${PWD}/../bin:$PATH
$ export FABRIC_CFG_PATH=$PWD/../config/
$ export CORE_PEER_TLS_ENABLED=true
$ export CORE_PEER_LOCALMSPID="Org1MSP"
$ export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
$ export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
$ export CORE_PEER_ADDRESS=localhost:7051
```

Initialize the ledger. This function is essentially a noop as we don't create any default assets.

```bash
almac@green:~/go/src/github.com/scrappythebird/fabric-samples/test-network$ peer chaincode \ 
    invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls \
    --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" \
    -C mychannel -n basic-academic --peerAddresses localhost:7051 \
    --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" \
    -c '{"function":"InitLedger","Args":[]}'
2021-04-21 18:43:17.983 EDT [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:
```

Org1 will call GetAllAssets and no results are returned because we haven't created any assets yet.

```bash
almac@green:~/go/src/github.com/scrappythebird/fabric-samples/test-network$ peer chaincode query -C mychannel -n basic-academic -c '{"Args":["GetAllAssets"]}'

almac@green:~/go/src/github.com/scrappythebird/fabric-samples/test-network$ 
```

We invoke the CreateAsset function to add a new asset to the ledger. We provide 6 arguments:

 - rid01: This is the asset ID. For demo purposes, this is the only variable that matters.
 - owner0: The owner asset - this is arbitrary and unvalidated.
 - http://example.com/data.json: Data URL. In theory, this would point the data.
 - deadbeef: DataHashHex. In theory, this would be cryptographic hash of the data to ensure integrity.
 - sum: Operation. In theory, this would represent the operation to perform on the data.
 - 55: Result. In theory this would represent the result of the operation.

```bash
almac@green:~/go/src/github.com/scrappythebird/fabric-samples/test-network$ peer chaincode \
    invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com --tls \
    --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" \
    -C mychannel -n basic-academic --peerAddresses localhost:7051 \
    --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" \
    --peerAddresses localhost:9051 \
    --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" \
    -c '{"function":"CreateAsset","Args":["rid01","owner0","http://example.com/data.json","deadbeef","sum","55"]}'
2021-04-21 22:58:23.336 EDT [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200
```

Validate that the asset was created:

```bash
almac@green:~/go/src/github.com/scrappythebird/fabric-samples/test-network$ peer \
    chaincode query -C mychannel -n basic-academic -c '{"Args":["GetAllAssets"]}'
[{"ID":"rid01","owner":"owner0","endorsers":[],"data_url":"http://example.com/data.json","data_hash_hex":"deadbeef","operation":"sum","data_result":"55"}]
```

Assume `owner2` was able to validate the results, and decided to endorse the results. The actual validation/replication mechanism is left out of this implementation. We can add an endorsement for `rid01` by invoking `EndorseAsset`. But, note that org1 and org2 keys are involved.

```bash
almac@green:~/go/src/github.com/scrappythebird/fabric-samples/test-network$ peer chaincode \
    invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls \
    --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" \
    -C mychannel -n basic-academic --peerAddresses localhost:7051 \
    --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" \
    --peerAddresses localhost:9051 \
    --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" \
    -c '{"function":"EndorseAsset","Args":["rid01","owner2"]}'
2021-04-21 23:00:07.952 EDT [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200
```

Confirm that the endorsement was added. Pipe the data through `jq` for better visual. 

```bash
almac@green:~/go/src/github.com/scrappythebird/fabric-samples/test-network$ peer chaincode query -C mychannel -n basic-academic -c '{"Args":["GetAllAssets"]}' | jq '.'
[
  {
    "ID": "rid01",
    "owner": "owner0",
    "endorsers": [
      "owner2"
    ],
    "data_url": "http://example.com/data.json",
    "data_hash_hex": "deadbeef",
    "operation": "sum",
    "data_result": "55"
  }
]
almac@green:~/go/src/github.com/scrappythebird/fabric-samples/test-network$
```

To point out that this isn't really operating like we'd want, we add a new endorser, "owner3". Note that there is no owner3 cert involved. Even though org1 and org2 agree that `owner3` endorsed the asset a more preffered chaincode would reject this and require owner3 to submit the endorsement invocation.

```bash
almac@green:~/go/src/github.com/scrappythebird/fabric-samples/test-network$ peer chaincode invoke -o localhost:7050     --ordererTLSHostnameOverride orderer.example.com --tls  --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"      -C mychannel -n basic-academic --peerAddresses localhost:7051   --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"         --peerAddresses localhost:9051  --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt"         -c '{"function":"EndorseAsset","Args":["rid01","owner3"]}'
2021-04-21 23:00:16.940 EDT [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200
almac@green:~/go/src/github.com/scrappythebird/fabric-samples/test-network$ peer chaincode query -C mychannel -n basic-academic -c '{"Args":["GetAllAssets"]}' | jq '.'
[
  {
    "ID": "rid01",
    "owner": "owner0",
    "endorsers": [
      "owner2",
      "owner3"
    ],
    "data_url": "http://example.com/data.json",
    "data_hash_hex": "deadbeef",
    "operation": "sum",
    "data_result": "55"
  }
]
```