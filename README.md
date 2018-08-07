# Hyberledger-1.2aberic
Hyberledger Fabric V1.2 Run Examples

1. 区块链节点为 peer(2) order(1) kafka(1) zokkeeper(1) cli(1)
2. 拷贝用例到**Hyberledger Fabric** 源码的根目录下
3. 环境的准备
    ```linux
    cd gopath/src/github.com/hyperledger/fabric
    git clone https://github.com/Maijh97/Hyberledger-1.2aberic.git
    ```
    
    * 运行e2e 进行下载镜像
    
    ```linux
    cd examples/e2e_cli/
    ./network_setup.sh up
    ```
    
    * 拷贝用作生成二进制文件的bin到Hyberledger-1.2aberic下
    
    ```linux
    cd release
    cp -r linux-amd64/bin/ ../Hyberledger-1.2aberic/
    ```
    
    * 生成组织证书创建通道和创世块
    
    ```
    cd Hyberledger-1.2aberic
    # 生成证书
    ./bin/cryptogen generate --config=./crypto-config.yaml
    
    # 创建创世块
    ./bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
    
    # 创建通道
    ./bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/mychannel.tx -channelID mychannel
    
    ```
    * 运行docker-compose
    
    ```linux
    docker-compose -f docker-compose-cli.yaml up -d
    ```

4. 测试**chaincode**  
    * 进入客户端 cli

    ```linux
    docker exec -it cli bash
    ``` 
    
    *  设置tls 证书路径
    
    ```linux
    export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    ```   
    *   创建通道
    
    ```linux
    peer channel create -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/mychannel.tx --tls --cafile $ORDERER_CA
    ```  
    *   加入通道
    
    ```
    peer channel join -b mychannel.block
    ```  
    *  安装链码
    
    ```linux
    peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/aberic/chaincode/go/example02/cmd
    ```
    * 实例化证书 
            
    ```linux
    peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C mychannel -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
    ```
    * 实现查询合约数据
        
    ```linux
    peer chaincode query -C mychannel -n mychannel -c '{"Args":["query","a"]}'
    ```  
    * 实现合约转移金额 a转b
    
    ```linux
    peer chaincode invoke -o orderer.example.com:7050  --tls --cafile $ORDERER_CA -C mychannel -n mychannel peer0.org1.example.com -c '{"Args":["invoke","a","b","10"]}' 
    ```  
    
   



