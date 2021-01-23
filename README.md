# Hydraweb3-js - Web3.js for HYDRA

Hydraweb3 is a library for dApps to interract with the HYDRA blockchain. Hydraweb3 communicates to a HYDRA node via the provider provided.

https://www.npmjs.com/package/hydraweb3

## Get Started
Run the following in your project folder:

	npm install hydraweb3 --save

## Web Dapp Usage
This is example is meant for web dapps who would like to use Hydraweb3's convenience methods with HydraWallet's RPC provider. HydraWallet is a Hydra wallet [Chrome extension](#). More details about HydraWallet [here](https://github.com/hydrachain/hydrawallet).

### 1. Construct Hydraweb3 instance
If you have HydraWallet installed, you will have a `window.hydrawallet` object injected in your browser tab. Pass that into Hydraweb3 as a parameter to set the provider.
```
const hydraweb3 = new Hydraweb3(window.hydrawallet.rpcProvider);
```

### 2. Construct Contract instance
The Contract class is meant for executing `sendtocontract` or `callcontract` at a specific contract address with a given ABI.
```
const contractAddress = 'f7b958eac2bdaca0f225b86d162f263441d23c19';
const contractAbi = [{"constant":false,"inputs":[{"name":"_eventAddress","type":"address"},{"name":"_eventName","type":"bytes32[10]"},{"name":"_eventResultNames","type":"bytes32[10]"},{"name":"_numOfResults","type":"uint8"},{"name":"_lastResultIndex","type":"uint8"},{"name":"_arbitrationEndBlock","type":"uint256"},{"name":"_consensusThreshold","type":"uint256"}],"name":"createDecentralizedOracle","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"_eventAddress","type":"address"},{"name":"_eventName","type":"bytes32[10]"},{"name":"_eventResultNames","type":"bytes32[10]"},{"name":"_numOfResults","type":"uint8"},{"name":"_lastResultIndex","type":"uint8"},{"name":"_arbitrationEndBlock","type":"uint256"},{"name":"_consensusThreshold","type":"uint256"}],"name":"doesDecentralizedOracleExist","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_oracle","type":"address"},{"name":"_eventAddress","type":"address"},{"name":"_eventName","type":"bytes32[10]"},{"name":"_eventResultNames","type":"bytes32[10]"},{"name":"_numOfResults","type":"uint8"},{"name":"_bettingEndBlock","type":"uint256"},{"name":"_resultSettingEndBlock","type":"uint256"},{"name":"_consensusThreshold","type":"uint256"}],"name":"createCentralizedOracle","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"_oracle","type":"address"},{"name":"_eventAddress","type":"address"},{"name":"_eventName","type":"bytes32[10]"},{"name":"_eventResultNames","type":"bytes32[10]"},{"name":"_numOfResults","type":"uint8"},{"name":"_bettingEndBlock","type":"uint256"},{"name":"_resultSettingEndBlock","type":"uint256"},{"name":"_consensusThreshold","type":"uint256"}],"name":"doesCentralizedOracleExist","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"name":"","type":"bytes32"}],"name":"oracles","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"_addressManager","type":"address"}],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":true,"name":"_contractAddress","type":"address"},{"indexed":true,"name":"_oracle","type":"address"},{"indexed":true,"name":"_eventAddress","type":"address"},{"indexed":false,"name":"_name","type":"bytes32[10]"},{"indexed":false,"name":"_resultNames","type":"bytes32[10]"},{"indexed":false,"name":"_numOfResults","type":"uint8"},{"indexed":false,"name":"_bettingEndBlock","type":"uint256"},{"indexed":false,"name":"_resultSettingEndBlock","type":"uint256"},{"indexed":false,"name":"_consensusThreshold","type":"uint256"}],"name":"CentralizedOracleCreated","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"_contractAddress","type":"address"},{"indexed":true,"name":"_eventAddress","type":"address"},{"indexed":false,"name":"_name","type":"bytes32[10]"},{"indexed":false,"name":"_resultNames","type":"bytes32[10]"},{"indexed":false,"name":"_numOfResults","type":"uint8"},{"indexed":false,"name":"_lastResultIndex","type":"uint8"},{"indexed":false,"name":"_arbitrationEndBlock","type":"uint256"},{"indexed":false,"name":"_consensusThreshold","type":"uint256"}],"name":"DecentralizedOracleCreated","type":"event"}];

// Create a new Contract instance and use the same provider as hydraweb3
const contract = hydraweb3.Contract(contractAddress, contractAbi);
```

### 3. Get Logged In Hydrawallet Account
To get the current logged in account in Hydrawallet, you will have to add an [Event Listener](https://www.w3schools.com/jsref/met_element_addeventlistener.asp) to listen to messages sent from Hydrawallet.
```
let account;

function onHydrawalletAcctChange(event) {
  if (event.data.message && event.data.message.type == "ACCOUNT_CHANGED") {
    account = event.data.message.payload;

    // You now have the logged in account.
    // account = InpageAccount {
    //   loggedIn: true,
    //   name: "My Wallet", 
    //   network: "Mainnet",
    //   address: "HJHp6dUSmDShpEEMmwxqHPo7sFSdydSkPM",
    //   balance: 49.10998413
    // } 

    // You may also get the account from `window.hydrawallet.account`.
    // account = window.hydrawallet.account
  }
}
window.addEventListener('message', onHydrawalletAcctChange, false);
```

### 4. Execute sendtocontract
The last piece is to execute a `sendtocontract` on your Contract instance. This will automatically show a Hydrawallet popup to confirm that you would like to send the transaction.
```
// Does a sendtocontract call on a function called setResult(uint8)
const tx = await contract.send('setResult', {
  methodArgs: [1],    // Sets the function params
  gasLimit: 1000000,  // Sets the gas limit to 1 million
  senderAddress: account.address,
});
// tx = txid of the transaction
```

## Hydraweb3Provider
The provider is the link between Hydraweb3 and the blockchain. A compatible Hydraweb3 Provider adheres to the following interface:
```
interface Hydraweb3Provider: {
  rawCall: (method: string, args: any[]) => Promise; // returns the result of the request
}
```

## Hydraweb3
Instantiate a new instance of `Hydraweb3`: 
```
const { Hydraweb3 } = require('hydraweb3');

// Instantiate Hydraweb3 with HttpProvider
// Pass in the URL of your HYDRA node RPC port with auth credentials.
// Default Hydraweb3 RPC ports: testnet=13389 mainnet=3389
const hydra = new Hydraweb3('http://user:password@localhost:3389');

// Instantiate Hydraweb3 with HydrawalletRPCProvider
// HydrawalletRPCProvider is a provider for the Hydrawallet  Chrome Extension.
// Please note HydrawalletRPCProvider only allows the rawCall() method to be used.
// It is specifically used for `sendtocontract` and `callcontract` only.
const hydraweb3 = new Hydraweb3(window.hydrawallet.rpcProvider);
```

### isConnected()
Checks if you are connected properly to the local hydra node.
```
async function isConnected() {
  return await hydraweb3.isConnected();
}
```

### getHexAddress(address)
Converts a HYDRA address to hex format.
```
async function getHexAddress() {
  return await hydraweb3.getHexAddress('HKjn4fStBaAtwGiwueJf9qFxgpbAvf1xAy');
}
```

### fromHexAddress(hexAddress)
Converts a hex address to HYDRA format.
```
async function fromHexAddress() {
  return await hydraweb3.fromHexAddress('29a4ebddfae8a7b6608a095f89b46d9930fdbe14');
}
```

### getBlockCount()
Gets the current block height of your local HYDRA node.
```
async function getBlockCount() {
  return await hydraweb3.getBlockCount();
}
```

### getTransaction(txid)
Gets the transaction details of the transaction id.
```
async function getTransaction(args) {
  const {
    transactionId, // string
  } = args;

  return await hydraweb3.getTransactionReceipt(transactionId);
}
```

### getTransactionReceipt(txid)
Gets the transaction receipt of the transaction id.
```
async function getTransactionReceipt(args) {
  const {
    transactionId, // string
  } = args;

  return await hydraweb3.getTransactionReceipt(transactionId);
}
```

### listUnspent()
Gets the unspent outputs that can be used.
```
async function listUnspent() {
  return await hydraweb3.listUnspent();
}
```

### searchLogs(fromBlock, toBlock, addresses, topics, contractMetadata, removeHexPrefix)
Gets the logs given the params on the blockchain.

The `contractMetadata` param contains the contract names and ABI that you would like to parse. An example of one is:
```
# contract_metadata.js
module.exports = {
  EventFactory: {
    address: 'd53927df927be7fc51ce8bf8b998cb6611c266b0',
    abi: [{ constant: true, inputs: [{ name: '', type: 'bytes32' }], name: 'topics', outputs: [{ name: '', type: 'address' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: false, inputs: [{ name: '_oracle', type: 'address' }, { name: '_name', type: 'bytes32[10]' }, { name: '_resultNames', type: 'bytes32[10]' }, { name: '_bettingEndBlock', type: 'uint256' }, { name: '_resultSettingEndBlock', type: 'uint256' }], name: 'createTopic', outputs: [{ name: 'topicEvent', type: 'address' }], payable: false, stateMutability: 'nonpayable', type: 'function' }, { constant: true, inputs: [{ name: '_name', type: 'bytes32[10]' }, { name: '_resultNames', type: 'bytes32[10]' }, { name: '_bettingEndBlock', type: 'uint256' }, { name: '_resultSettingEndBlock', type: 'uint256' }], name: 'doesTopicExist', outputs: [{ name: '', type: 'bool' }], payable: false, stateMutability: 'view', type: 'function' }, { inputs: [{ name: '_addressManager', type: 'address' }], payable: false, stateMutability: 'nonpayable', type: 'constructor' }, { anonymous: false, inputs: [{ indexed: true, name: '_topicAddress', type: 'address' }, { indexed: true, name: '_creator', type: 'address' }, { indexed: true, name: '_oracle', type: 'address' }, { indexed: false, name: '_name', type: 'bytes32[10]' }, { indexed: false, name: '_resultNames', type: 'bytes32[10]' }, { indexed: false, name: '_bettingEndBlock', type: 'uint256' }, { indexed: false, name: '_resultSettingEndBlock', type: 'uint256' }], name: 'TopicCreated', type: 'event' }],
  },

  TopicEvent: {
    abi: [{ constant: false, inputs: [{ name: '_resultIndex', type: 'uint8' }, { name: '_sender', type: 'address' }, { name: '_amount', type: 'uint256' }], name: 'voteFromOracle', outputs: [{ name: '', type: 'bool' }], payable: false, stateMutability: 'nonpayable', type: 'function' }, { constant: true, inputs: [], name: 'totalBotValue', outputs: [{ name: '', type: 'uint256' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: true, inputs: [{ name: '_oracleIndex', type: 'uint8' }], name: 'getOracle', outputs: [{ name: '', type: 'address' }, { name: '', type: 'bool' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: true, inputs: [{ name: '', type: 'address' }], name: 'didWithdraw', outputs: [{ name: '', type: 'bool' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: true, inputs: [], name: 'resultSet', outputs: [{ name: '', type: 'bool' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: true, inputs: [], name: 'status', outputs: [{ name: '', type: 'uint8' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: true, inputs: [], name: 'getFinalResult', outputs: [{ name: '', type: 'uint8' }, { name: '', type: 'string' }, { name: '', type: 'bool' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: true, inputs: [{ name: '', type: 'uint256' }], name: 'resultNames', outputs: [{ name: '', type: 'bytes32' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: true, inputs: [{ name: '', type: 'uint256' }], name: 'oracles', outputs: [{ name: 'didSetResult', type: 'bool' }, { name: 'oracleAddress', type: 'address' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: false, inputs: [], name: 'finalizeResult', outputs: [{ name: '', type: 'bool' }], payable: false, stateMutability: 'nonpayable', type: 'function' }, { constant: false, inputs: [{ name: '_oracle', type: 'address' }, { name: '_resultIndex', type: 'uint8' }, { name: '_consensusThreshold', type: 'uint256' }], name: 'centralizedOracleSetResult', outputs: [], payable: false, stateMutability: 'nonpayable', type: 'function' }, { constant: true, inputs: [], name: 'totalHydraValue', outputs: [{ name: '', type: 'uint256' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: false, inputs: [{ name: '_consensusThreshold', type: 'uint256' }], name: 'invalidateOracle', outputs: [], payable: false, stateMutability: 'nonpayable', type: 'function' }, { constant: true, inputs: [], name: 'getBetBalances', outputs: [{ name: '', type: 'uint256[10]' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: true, inputs: [], name: 'owner', outputs: [{ name: '', type: 'address' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: true, inputs: [], name: 'calculateHydraContributorWinnings', outputs: [{ name: '', type: 'uint256' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: true, inputs: [], name: 'getVoteBalances', outputs: [{ name: '', type: 'uint256[10]' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: true, inputs: [], name: 'getTotalVotes', outputs: [{ name: '', type: 'uint256[10]' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: false, inputs: [{ name: '_better', type: 'address' }, { name: '_resultIndex', type: 'uint8' }], name: 'bet', outputs: [], payable: true, stateMutability: 'payable', type: 'function' }, { constant: false, inputs: [{ name: '_resultIndex', type: 'uint8' }, { name: '_currentConsensusThreshold', type: 'uint256' }], name: 'votingOracleSetResult', outputs: [{ name: '', type: 'bool' }], payable: false, stateMutability: 'nonpayable', type: 'function' }, { constant: true, inputs: [], name: 'getTotalBets', outputs: [{ name: '', type: 'uint256[10]' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: true, inputs: [], name: 'getEventName', outputs: [{ name: '', type: 'string' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: true, inputs: [], name: 'invalidResultIndex', outputs: [{ name: '', type: 'uint8' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: true, inputs: [], name: 'numOfResults', outputs: [{ name: '', type: 'uint8' }], payable: false, stateMutability: 'view', type: 'function' }, { constant: false, inputs: [], name: 'withdrawWinnings', outputs: [], payable: false, stateMutability: 'nonpayable', type: 'function' }, { constant: false, inputs: [{ name: '_newOwner', type: 'address' }], name: 'transferOwnership', outputs: [], payable: false, stateMutability: 'nonpayable', type: 'function' }, { constant: true, inputs: [], name: 'calculateBotContributorWinnings', outputs: [{ name: '', type: 'uint256' }], payable: false, stateMutability: 'view', type: 'function' }, { inputs: [{ name: '_owner', type: 'address' }, { name: '_centralizedOracle', type: 'address' }, { name: '_name', type: 'bytes32[10]' }, { name: '_resultNames', type: 'bytes32[10]' }, { name: '_bettingEndBlock', type: 'uint256' }, { name: '_resultSettingEndBlock', type: 'uint256' }, { name: '_addressManager', type: 'address' }], payable: false, stateMutability: 'nonpayable', type: 'constructor' }, { payable: true, stateMutability: 'payable', type: 'fallback' }, { anonymous: false, inputs: [{ indexed: true, name: '_eventAddress', type: 'address' }, { indexed: false, name: '_finalResultIndex', type: 'uint8' }], name: 'FinalResultSet', type: 'event' }, { anonymous: false, inputs: [{ indexed: true, name: '_winner', type: 'address' }, { indexed: false, name: '_hydraTokenWon', type: 'uint256' }, { indexed: false, name: '_botTokenWon', type: 'uint256' }], name: 'WinningsWithdrawn', type: 'event' }, { anonymous: false, inputs: [{ indexed: true, name: '_previousOwner', type: 'address' }, { indexed: true, name: '_newOwner', type: 'address' }], name: 'OwnershipTransferred', type: 'event' }],
  },
};
```

Usage:
```
const ContractMetadata = require('./contract_metadata');

async function(args) {
  let {
    fromBlock, // number
    toBlock, // number
    addresses, // string array
    topics // string array
  } = args;

  if (addresses === undefined) {
    addresses = [];
  }
  if (topics === undefined) {
    topics = [];
  }

  // removeHexPrefix = true removes the '0x' hex prefix from all hex values
  return await hydraweb3.searchLogs(fromBlock, toBlock, addresses, topics, contractMetadata, true);
}
```

## Contract.js
Instantiate a new instance of `Contract`: 
```
const { Hydraweb3 } = require('hydraweb3');

const hydraweb3 = new Hydraweb3('http://user:password@localhost:3389');

// contractAddress = The address of your contract deployed on the blockchain
const contractAddress = 'f7b958eac2bdaca0f225b86d162f263441d23c19';

// contractAbi = The ABI of the contract
const contractAbi = [{"constant":false,"inputs":[{"name":"_eventAddress","type":"address"},{"name":"_eventName","type":"bytes32[10]"},{"name":"_eventResultNames","type":"bytes32[10]"},{"name":"_numOfResults","type":"uint8"},{"name":"_lastResultIndex","type":"uint8"},{"name":"_arbitrationEndBlock","type":"uint256"},{"name":"_consensusThreshold","type":"uint256"}],"name":"createDecentralizedOracle","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"_eventAddress","type":"address"},{"name":"_eventName","type":"bytes32[10]"},{"name":"_eventResultNames","type":"bytes32[10]"},{"name":"_numOfResults","type":"uint8"},{"name":"_lastResultIndex","type":"uint8"},{"name":"_arbitrationEndBlock","type":"uint256"},{"name":"_consensusThreshold","type":"uint256"}],"name":"doesDecentralizedOracleExist","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_oracle","type":"address"},{"name":"_eventAddress","type":"address"},{"name":"_eventName","type":"bytes32[10]"},{"name":"_eventResultNames","type":"bytes32[10]"},{"name":"_numOfResults","type":"uint8"},{"name":"_bettingEndBlock","type":"uint256"},{"name":"_resultSettingEndBlock","type":"uint256"},{"name":"_consensusThreshold","type":"uint256"}],"name":"createCentralizedOracle","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"_oracle","type":"address"},{"name":"_eventAddress","type":"address"},{"name":"_eventName","type":"bytes32[10]"},{"name":"_eventResultNames","type":"bytes32[10]"},{"name":"_numOfResults","type":"uint8"},{"name":"_bettingEndBlock","type":"uint256"},{"name":"_resultSettingEndBlock","type":"uint256"},{"name":"_consensusThreshold","type":"uint256"}],"name":"doesCentralizedOracleExist","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"name":"","type":"bytes32"}],"name":"oracles","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"_addressManager","type":"address"}],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":true,"name":"_contractAddress","type":"address"},{"indexed":true,"name":"_oracle","type":"address"},{"indexed":true,"name":"_eventAddress","type":"address"},{"indexed":false,"name":"_name","type":"bytes32[10]"},{"indexed":false,"name":"_resultNames","type":"bytes32[10]"},{"indexed":false,"name":"_numOfResults","type":"uint8"},{"indexed":false,"name":"_bettingEndBlock","type":"uint256"},{"indexed":false,"name":"_resultSettingEndBlock","type":"uint256"},{"indexed":false,"name":"_consensusThreshold","type":"uint256"}],"name":"CentralizedOracleCreated","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"_contractAddress","type":"address"},{"indexed":true,"name":"_eventAddress","type":"address"},{"indexed":false,"name":"_name","type":"bytes32[10]"},{"indexed":false,"name":"_resultNames","type":"bytes32[10]"},{"indexed":false,"name":"_numOfResults","type":"uint8"},{"indexed":false,"name":"_lastResultIndex","type":"uint8"},{"indexed":false,"name":"_arbitrationEndBlock","type":"uint256"},{"indexed":false,"name":"_consensusThreshold","type":"uint256"}],"name":"DecentralizedOracleCreated","type":"event"}];

// Create a Contract instance from the Hydraweb3 instance
const contract = hydraweb3.Contract(contractAddress, contractAbi);
```

### call(methodName, params)
Executes a `callcontract`
```
// callcontract on a method named 'bettingEndBlock'
async function exampleCall(args) {
  const {
    senderAddress, // address
  } = args;

  return await contract.call('bettingEndBlock', {
    methodArgs: [],
    senderAddress: senderAddress,
  });
}
```

### send(methodName, params)
Executes a `sendtocontract`
```
// sendtocontract on a method named 'setResult'
async function exampleSend(args) {
  const {
    resultIndex, // number
    senderAddress, // address
  } = args;

  return await contract.send('setResult', {
    methodArgs: [resultIndex],
    gasLimit: 1000000, // setting the gas limit to 1 million
    senderAddress: senderAddress,
  });
}
```

## Encoder
`Encoder` static functions are exposed in Hydraweb3 instances.
```
const { Hydraweb3 } = require('hydraweb3');

const hydraweb3 = new Hydraweb3('http://user:password@localhost:3389');
hydraweb3.encoder.addressToHex(address);
hydraweb3.encoder.boolToHex(value);
hydraweb3.encoder.intToHex(num);
hydraweb3.encoder.uintToHex(num);
hydraweb3.encoder.stringToHex(string, maxCharLen);
hydraweb3.encoder.stringArrayToHex(strArray, numOfItems);
hydraweb3.encoder.padHexString(hexStr);
hydraweb3.encoder.constructData(abi, methodName, args);
```

## Decoder
`Decoder` static functions are exposed in Hydraweb3 instances.
```
const { Hydraweb3 } = require('hydraweb3');

const Hydraweb3 = new Hydraweb3('http://user:password@localhost:3389');
hydraweb3.decoder.toHydraAddress(hexAddress, isMainnet);
hydraweb3.decoder.removeHexPrefix(value);
hydraweb3.decoder.decodeSearchLog(rawOutput, contractMetadata, removeHexPrefix);
hydraweb3.decoder.decodeCall(rawOutput, contractABI, methodName, removeHexPrefix);
```

## Utils
`Utils` static functions are exposed in Hydraweb3 instances.
```
const { Hydraweb3 } = require('Hydraweb3');

const Hydraweb3 = new Hydraweb3('http://user:password@localhost:3389');
hydraweb3.utils.paramsCheck(methodName, params, required, validators);
hydraweb3.utils.appendHexPrefix(value);
hydraweb3.utils.trimHexPrefix(str);
hydraweb3.utils.chunkString(str, length);
hydraweb3.utils.toUtf8(hex);
hydraweb3.utils.fromUtf8(str);
hydraweb3.utils.isJson(str);
hydraweb3.utils.isHydraAddress(address);
```
