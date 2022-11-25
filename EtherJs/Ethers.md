# EtherJs

## JAVASCRIPT (abd using asynchronous functions)

Solidity is synchronous, exception: when we work with oracle

Javascript can be asynchronous (by defalut), meaning we can have the code running at the same time
In sync, we wait for each step to finish first and then the next step is executed. Eg, In a treassure hunt you cannot move to next stage without completing the current stage
In async programming, we can implement other steps while waiting for a task to complete. Eg, You can talk with your friends while you are waiting for your girlfriend to finish her class ;) do stuff without waiting around for prev stuff to finish

In sync, function that comes with waiting period, returns a promise
A promise can be pending, fullfilled or rejected
In order to make the other functions wait to finish the execution of the current function we use the keyword - await tells any promise based function to wait for that promise to be fullfiled or rejected

!!! await can only be used in promise based functions

If we don't use await keyword then we get promise in it's pending state beacuse our code gets finished before actually deploying the contract. Await resoles promise to a contract.

When we deploy a contract, we want to "wait" for it to deploy
if we dont use sync programming, and use our function without async keyword then we won't wait for contract.deploy operation to finish

## Basic setup

```javascript
async function main() {}

main()
    .then(() => process.exit(0))
    .catch((error) => {
        console.error(error);
        process.exit(1);
    });
```

## Compiling our Solidity

Consider the SimpleStorage.sol file.
We will use yarn instead of npm to install libraries. It can be used to intall dependencies and run scripts
To install yarn:

```console
corepack enable
```

In order for us to compile our .sol file we gonna need a tool called solc-js
Installing solc-js using yarn

```console
yarn add solc
```

Adding the compiler version in solc, since we were using solidity version 0.8.7 in our SimpleStorage.sol, we will install the same version using (-fixed was added because eariler there was some issue with this compiler version):

```console
yarn add solc@0.8.7-fixed
```

We can compile our code in two ways:

1. compile them in our code (not using this method)
2. compile them seperatly using yarn

Compile using :

```console
yarn solcjs --bin --abi --include-path node_modules/ --base-path . -o . SimpleStorage.sol
```

Compiling will give us a binary file and an abi file

Shortning our yarn script : Go to package.json and add:

```json
"scripts": {
        "compile": "yarn solcjs --bin --abi --include-path node_modules/ --base-path . -o . SimpleStorage.sol"
    }
```

This basically compiles our SimpleStorage file every time we write yarn compile

# DEPLOYING

## Deploying to JVM or fake blockchain using GANACHE

(In future we are gonna do this using only hardhat)
Ganache : Similar to VM in Remix, fake Blockchain to run locally in order to test, deploy and run code.
All networks have different RPC's (remote procedure call) -> helps to make api calls to connect to the blockchain

!!! If you face a issue with opening starting Ganche new Workstation like I did when I opened it for the second time, go to C -> user -> owner -> appdata -> roaming and delete the ganache file.

Take the RPC server link from ganache

# EtherJs

Ether.js library aims to be a complete and compact library for interacting with the Ethereum Blockchain and its ecosystem. Alternate - Web3.js (used more in future)
Ehter is the main tool that powers hardhat environment

Adding Ethers:

```console
yarn add ethers
```

Importing ethers in our deploy.js file:

```javascript
const ethers = require("ethers");
```

To connect to our local blockchain (we our entering our RPC link) :

```javascript
async function main() {
    const provider = new ethers.providers.JsonRpcProvider(
        "http://127.0.0.1:7545"
    );
}
```

Connecting our wallet:
Never save your private key directly in your code!!!

```javascript
const wallet = new ethers.Wallet("(enter private key here)", provider);
```

Now, in order to compile our contract, we need to read our bin and abi file, for this we use fs

```javascript
const fs = require("fs-extra");
// inside main
const abi = fs.readFileSync("./SimpleStorage_sol_SimpleStorage.abi", "utf8"); //reading file synchronously
const binary = fs.readFileSync("./SimpleStorage_sol_SimpleStorage.bin", "utf8"); //reading file synchronously
```

Now that we have both abi and binary file, we can create a contract factory and deploy our contract using ethers :

```javascript
const contractFactory = new ethers.ContractFactory(abi, binary, wallet);
console.log("Deploying, please wait...");
const contract = await contractFactory.deploy(); // deploying with ehters
```

Await tells our code to stop here and wait for our contract to get deployed
Finally running our file by using node to deploy the contract:

```console
node deploy.js
```

You might encounter an error :

```console
Error: could not detect network (event="noNetwork", code=NETWORK_ERROR, version=providers/5.7.2)
```

This is because:
You are working in the WSL ubuntu terminal and installed the Ganache locally and must be located in the WSL server. So, Ganache is not connected to different environment. Go to the Ganache and click on Settings, then go the server and choose WSL as server. After this save and restart the Ganache. In your code in the connection code replace this

```javascript
const provider = new ethers.providers.JsonRpcProvider("new_RPC_url");
```

or with your shown JSON-RPC. It will work for you. Make sure to change the prikat key of the wallet also.

## Adding gas price and other parameters

inside deploy() we can use gasPrice, gasLimit

```javascript
const contract = await contractFactory.deploy({ gasPrice: 1000000000000 });
```

## Transaction Receipts

If we wanna wait 1 block to make sure that it actually got attached to our chain, here we are waiting for 1 block confirmation

```javascript
const transactionReceipt = await contract.deployTransaction.wait(1);
```

transaction receipt is what we get when we wait for a block confirmation
deployment transaction or the transaction response is what we get just when we create our transaction

## Sending a raw transaction in ethers -Deploying or sending transactions purely with transaction data!!!

nonce are used in wallets to send transactions

Old way:

```javascript
console.log("Deploying with only transaction data: ");
const nonce = await wallet.getTransactionCount();
const tx = {
    nonce: nonce,
    gasPrice: 20000000000,
    gasLimit: 1000000,
    to: null,
    value: 0,
    data: "data inside our .bin file, make sure to add 0x in starting",
    chainId: 5777,
};
// const signedTxResponse = await wallet.signTransaction(tx); //signing a transaction
const sentTxResponse = await wallet.sendTransaction(tx);
await sentTxResponse.wait(1);
console.log(sentTxResponse);
```

In purely Ethers method:

## Interacting with Contracts in EthersJs

contract variable that was created using contractFactory.deploy() method helps us to interact with our contract. It's retreive() function reads information from our abi file and returns us the required value.
For reading a particular value, for eg. favourite number :

```javascript
const currentFavoriteNumber = await contract.retrieve();
```

Doing this will not cost us any gas since this variable was declared as view and we are only retrieving it's value.
Now if we compile, we got something called BigNumber (from bignumber library) because solidity does not have floating/decimal numbers.

```console
Deploying, please wait...
BigNumber { _hex: '0x00', _isBigNumber: true }
```

To fix this we use .toString() method:

```javascript
console.log(`Current favorite number: ${currentFavoriteNumber.toString()}`);
```

Saving our favorite number and getting it:

```javascript
const transactionRespose = await contract.store("7"); // stores the favNum by calling a function contract
const transactionReceipt = await transactionRespose.wait(1); // transactionRec waits 1 block for confiriming
const updatedFavoriteNumber = await contract.retrieve();
console.log(`Updated favorite number: ${updatedFavoriteNumber.toString()}`);
```

## Environment Variables

We don't actually store our private key and RPC link direactly onto our code i.e. we never write any sensetive data in our code. We store it in a environment file instead. This file is hidden in the background and is not accessible to outside users.
To create a env file, just type .env in new file.
// Sometimes a compiler may ask to put 0x at start of the pvt key, not necessary in hardhat/ethers

```env
PRIVATE_KEY=904cfc7d9f08b184c507fa7c581a0fba6c55674843534598e17b87c76cf30b946f0698
RPC_URL=http://172.25.96.1:7545
```

Now to add this pvt key to our deploy.js file, we need a tool called dotenv.

```javascript
require("dotenv").config();
```

Entering our pvt key in our file

```javascript
const provider = new ethers.providers.JsonRpcProvider("process.env.RPC_URL");
const wallet = new ethers.Wallet("process.env.PRIVATE_KEY", provider);
```

In order to make sure that .env file is not added to the github repo, we use a .gitignore file. This also includes node_modules.

## Better private key management

Providing the private key inside a .env file can sometimes be risky as we can forget to put the .env file in .gitignore file. In order to sequre our private key more efficienty we use the Encryption method.
We can encrypt our private key using a password. Follow along:

1. Create a file called encryptKey.js
2. Add a password in .env file

```env
PASSWORD=password
```

3. use .encrypt() method to encrypt the pvt key

```javascript
const encryptedJsonKey = await wallet.encrypt(
    process.env.PRIVATE_KEY_PASSWORD,
    process.env.PRIVATE_KEY
);
```

4. Save the encrypted pvt key in new file .encryptedKey.json

```javascript
fs.writeFileSync("./.encryptedKey.json", encryptedJsonKey);
```

5. Add this file to .gitignore
6. Now we can deleted the private key and password from .env file
7. Each time we deploy our javascript file using node we provide the password along with the file

```console
node PASSWORD=password deploy.js
```

8. Note that if someone hacks into our system, they can access our history to get the password therefore, we should clear our history using history -c

NOTE : For rest of the course, we use pvt key directly in env file just for easy access
Finall encryptedKey.js file:

```javascript
const ethers = require("ethers");
const fs = require("fs-extra");
require("dotenv").config();

async function main() {
    const wallet = new ethers.Wallet(process.env.PRIVATE_KEY);
    const encryptedJsonKey = await wallet.encrypt(
        process.env.PRIVATE_KEY_PASSWORD,
        process.env.PRIVATE_KEY
    );
    console.log(encryptedJsonKey);
    // saving our encrypted key
    fs.writeFileSync("./.encryptedKey.json", encryptedJsonKey);
}

main()
    .then(() => process.exit(0))
    .catch((error) => {
        console.error(error);
        process.exit(1);
    });
```

## Making prettier plugin available to every user

Adding prettier to our project using yarn

```console
yarn add prettier prettier-plugin-solidity
```

Create a new file called .prettierrc and add the required arguments init

```json
{
    "tabWidth": 4,
    "semi": false,
    "singleQuote": false
}
```

## Deploying to a testnet or mainnet

We can get the RPC url of our testnet from Alchemy
https://dashboard.alchemy.com/
Once we are in out dashboard, we can get the RPC url by: View key -> Copy http -> Replace it with .env RPC_URL
For getting the private key we use our Metamask wallet.
Replace the Ganache Private key with the Metamask wallet key and we are all set to deploy our contract on a testnet!

## what goes behind alchemy (not required)

## CONCLUSION

Final boiler plate for creating and deploying a contract and interacting with it :

```javascript
const ethers = require("ethers");
const fs = require("fs-extra");
require("dotenv").config();

async function main() {
    // conecting to blockchain
    let provider = new ethers.providers.JsonRpcProvider(process.env.RPC_URL); // direct RPC connection
    // wallet to sign in blockchain
    let wallet = new ethers.Wallet(process.env.PRIVATE_KEY, provider); // like this if private key is stored in .env file
    // if we have encrypted our private key then we can use the code below to read it from the encryptedKey.json file
    // const encryptedJson = fs.readFileSync("./.encryptedKey.json", "utf8");
    // let wallet = new ethers.Wallet.fromEncryptedJsonSync(
    //     encryptedJson,
    //     process.env.PRIVATE_KEY_PASSWORD
    // );
    // wallet = await wallet.connect(provider);
    // reading our abi file data
    const abi = fs.readFileSync(
        "./SimpleStorage_sol_SimpleStorage.abi",
        "utf8"
    ); //reading file synchronously
    // reading our binary file data
    const binary = fs.readFileSync(
        "./SimpleStorage_sol_SimpleStorage.bin",
        "utf8"
    ); //reading file synchronously
    // creating our contract
    const contractFactory = new ethers.ContractFactory(abi, binary, wallet);
    console.log("Deploying, please wait...");
    const contract = await contractFactory.deploy(); // deploying with ehters
    //cosnole.log to know what address we are at for mainnet/testnet
    console.log(`Contract Address: ${contract.address}`);
    await contract.deployTransaction.wait(1); //waiting 1 block for deploy confirmation
    // below code to interact with the blockchain
    const currentFavoriteNumber = await contract.retrieve();
    console.log(`Current favorite number: ${currentFavoriteNumber.toString()}`);
    const transactionRespose = await contract.store("7");
    const transactionReceipt = await transactionRespose.wait(1);
    const updatedFavoriteNumber = await contract.retrieve();
    console.log(`Updated favorite number: ${updatedFavoriteNumber.toString()}`);
}

main()
    .then(() => process.exit(0))
    .catch((error) => {
        console.error(error);
        process.exit(1);
    });
```
