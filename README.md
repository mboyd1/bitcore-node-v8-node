Ubuntu 18.04 OS

Step1  
//Install node.js,the node .js version is v8.15.0 and  npm version 6.4.1  
```curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -```  
```sudo apt-get install -y nodejs```  

Step2  
//Install mongoDB and run it
```apt-get install mongodb```
```service mongodb start```

Step3  
//Install bitcoin-sv's bitcoind (version is v0.1.0.0)   
```sudo apt-get install git build-essential libtool autotools-dev automake pkg-config libssl-dev libevent-dev bsdmainutils libboost-chrono-dev libboost-program-options-dev libboost-test-dev libboost-thread-dev libzmq3-dev libdb-dev libdb++-dev```  
```git clone https://github.com/bitcoin-sv/bitcoin-sv.git```  
```cd bitcoin-sv```  
```./autogen.sh```  
```./configure```  
```make```  
```sudo make install```  

Step4 
//run bitcoind   
```mkdir ~/.bitcoin```  
```touch ~/.bitcoin/bitcoin.conf```  
```vi ~/.bitcoin/bitcoin.conf```  
```  
datadir=[where you want to store the node data]
server=1
whitebind=127.0.0.1:8333
whitelist=127.0.0.1
txindex=1
addressindex=1
timestampindex=1
spentindex=1
zmqpubrawtx=tcp://127.0.0.1:28332
zmqpubrawtxlock=tcp://127.0.0.1:28332
zmqpubhashblock=tcp://127.0.0.1:28332
rpcallowip=127.0.0.1
rpcport=8332
rpcuser=bsv
rpcpassword=local321
```
```bitcoind -conf=/home/[user]/bitcoin.conf```  

Step5  
//Install bitcore-node  
```git clone -b v8.0.0 https://github.com/bitpay/bitcore-node.git```  
```cd bitcore-node```  
```npm i ajv --save```  
```npm i hawk --save```  
```npm i --save```  

Step5  
//start at line 66,modify RPC part to connect the local p2p node   
```cd ~/bitcore-node/src```  
```vi config.ts```  
```
import * as os from "os";
import parseArgv from "./utils/parseArgv";
import ConfigType from "./types/Config";
let program = parseArgv([], ["config"]);

function findConfig(): ConfigType | undefined {
  let foundConfig;
  const envConfigPath = process.env.BITCORE_CONFIG_PATH;
  const argConfigPath = program.config;
  const configFileName = "bitcore.config.json";
  let bitcoreConfigPaths = [
    `${os.homedir()}/${configFileName}`,
    `../../../${configFileName}`,
    `../${configFileName}`
  ];
  const overrideConfig = argConfigPath || envConfigPath;
  if (overrideConfig) {
    bitcoreConfigPaths.unshift(overrideConfig);
  }
  // No config specified. Search home, bitcore and cur directory
  for (let path of bitcoreConfigPaths) {
    if (!foundConfig) {
      try {
        const bitcoreConfig = require(path) as { bitcoreNode: ConfigType };
        foundConfig = bitcoreConfig.bitcoreNode;
      } catch (e) {
        foundConfig = undefined;
      }
    }
  }
  return foundConfig;
}

function setTrustedPeers(config: ConfigType): ConfigType {
  for (let [chain, chainObj] of Object.entries(config)) {
    for (let network of Object.keys(chainObj)) {
      let env = process.env;
      const envString = `TRUSTED_${chain.toUpperCase()}_${network.toUpperCase()}_PEER`;
      if (env[envString]) {
        let peers = config.chains[chain][network].trustedPeers || [];
        peers.push({
          host: env[envString],
          port: env[`${envString}_PORT`]
        });
        config.chains[chain][network].trustedPeers = peers;
      }
    }
  }
  return config;
}
const Config = function(): ConfigType {
  let config: ConfigType = {
    maxPoolSize: 20,
    pruneSpentScripts: true,
    port: 3000,
    dbHost: process.env.DB_HOST || "127.0.0.1",
    dbName: process.env.DB_NAME || "bitcore",
    numWorkers: os.cpus().length,
    chains: {}
  };

  let foundConfig = findConfig();
  Object.assign(config, foundConfig, {});
  if (!Object.keys(config.chains).length) {
    Object.assign(config.chains, {
      BCH: {
        mainnet: {
          chainSource: "p2p",
          trustedPeers: [{ host: "127.0.0.1", port: 8333 }],
          rpc: {
            host: "127.0.0.1",
            port: 8332,
            username: "bsv",
            password: "local321"
          }
        }
      }
    });
  }
  config = setTrustedPeers(config);
  return config;
};

export default Config();

```


Step7(options)  
//Install pm2  
```cd ~```  
```sudo npm install pm2 -g```  
//If the server reboot,pm2 will auto restart.  
```pm2 startup```  
```sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu```  
```pm2 save```  

//if you install pm2 please choose step7.1 to run the bitcoin-cash server.  
Step 8.1  
```cd ~/bicore-node```   
```pm2 start "/usr/bin/npm" --name "BSVAPISERVER" -- start```  

//if you didin't install pm2,please choose step7.2 to run the bitcoin-cash server.   
Step8.2  
```cd ~/bitcore-node```  
```npm start```

//////Done!!!!///////

