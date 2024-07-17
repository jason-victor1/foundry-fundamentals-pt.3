# Deploying a Smart Contract Locally Using Forge

## Deploying to a Local Blockchain

### Exploring Forge Capabilities
To explore Forge's capabilities, I typed:
```bash
forge --help
```

From the resulting list, I found the `create` command. To learn more about its options, I typed:
```bash
forge create --help
```

### Attempting Deployment
I tried running:
```bash
forge create SimpleStorage
```
It failed because I hadn't specified some required parameters:
1. Where to deploy?
2. Who's paying the gas fees/signing the transaction?

### Specifying Deployment Parameters
Each blockchain (private or public) has an RPC URL (RPC SERVER) that acts as an endpoint. By default, Forge tried to use `http://localhost:8545/`, which didn't host any blockchain.

### Deploying with Anvil
1. I ran:
   ```bash
   clear
   anvil
   ```
2. I created a new terminal by pressing the `+` button.
3. I copied one of the private keys from the Anvil terminal.
4. I ran:
   ```bash
   forge create SimpleStorage --interactive
   ```
   (No need to specify `--rpc-url` as Forge defaults to Anvil's RPC URL.)

### Checking Deployment Details
In the Anvil terminal, I saw details such as:
```plaintext
Transaction: 0x40d2ca8f0d680f098c7d5e3c127ef1ce1207ef439ba6e163c2042483e15998a6
Contract created: 0x5fbdb2315678afecb367f032d93f642f64180aa3
Gas used: 357076
Block Number: 1
Block Hash: 0x85a56c0b8f166e86d1cce65412615e0d9a72972e04b2488023275131ea27330a
Block Time: "Mon, 15 Apr 2024 11:50:55 +0000"
```

### Explicit Deployment Command
For a more explicit way to deploy using `forge create`, I used:
```bash
forge create SimpleStorage --rpc-url http://127.0.0.1:8545 --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
```
This included the `--rpc-url` to avoid relying on the default and the `--private-key` to avoid using the `--interactive` option.

### Key Points Learned
- How to deploy a smart contract on a local blockchain using Anvil.
- The importance of specifying deployment parameters.
- How to interact with the terminal efficiently for deploying smart contracts.

## Practicing Private Key Safety

### Importance of Private Key Security
Having a private key in plain text is extremely bad. The private keys used for local testing in the last lesson were well-known and could be kept in plain text. However, any other private key, especially those associated with accounts holding crypto or for production, should be kept hidden.

### Risks of Plain Text Keys
- **Bash History**: Having private keys in my bash history is a security risk. To delete my history, I typed:
  ```bash
  history -c
  ```

### Consequences of Poor Key Management
Hacking private keys is one of the primary reasons people and projects lose large amounts of money. Here are some notable examples:
- **The Ronin Hack**: Social engineering of private keys.
- **Early Crypto Investor Bo Shen**: Lost $42 million in a wallet hack.
- **FTX Hack**: $477 million lost due to unencrypted private keys, with over $150 million stolen from Alameda Research due to poor security.

I made sure not to be like that! Even if I wasn't holding millions, what I had was mine. I didn't let it become someone else's.

## Deploying a Smart Contract Locally Using Anvil via Scripts

### Why Deploy via Scripts?
Deploying a smart contract via scripting provided a consistent and repeatable way to deploy reliably. It enhanced the testing of both the deployment processes and the code itself. While the command-line approach was useful, scripting brought more functionality and ease of use. Foundry simplified this process since it was written in Solidity, meaning my deployment scripts would also be written in Solidity.

### Setting Up the Deployment Script
1. **Create Script File**:
   - In Foundry, scripts were kept in the `script` folder.
   - I created a new file called `DeploySimpleStorage.s.sol`. Using `.s.sol` as a suffix was a naming convention for Foundry scripts.

2. **Write the Script**:
   - I opened the newly created file and typed the following:
     ```solidity
     // SPDX-License-Identifier: MIT
     pragma solidity 0.8.19;

     import {Script} from "forge-std/Script.sol";
     import {SimpleStorage} from "../src/SimpleStorage.sol";

     contract DeploySimpleStorage is Script {
         function run() external returns (SimpleStorage) {
             vm.startBroadcast();
             SimpleStorage simpleStorage = new SimpleStorage();
             vm.stopBroadcast();
             return simpleStorage;
         }
     }
     ```
   - Explanation:
     - **Imports**: Imported `Script` from `forge-std/Script.sol` and `SimpleStorage` from `../src/SimpleStorage.sol`.
     - **Contract Declaration**: Declared the `DeploySimpleStorage` contract inheriting from `Script`.
     - **Run Function**: The `run` function created a new `SimpleStorage` contract. `vm.startBroadcast` and `vm.stopBroadcast` indicated the start and end points for transactions sent to the RPC URL.

### Running the Script
1. **Stop Anvil**:
   - I selected the Anvil terminal and pressed `CTRL(CMD) + C` to stop it.

2. **Run the Script**:
   - I ran the following command:
     ```bash
     forge script script/DeploySimpleStorage.s.sol
     ```
   - If I encountered errors related to incompatible Solidity versions, I ensured both `SimpleStorage.sol` and `DeploySimpleStorage.s.sol` used `pragma solidity 0.8.19;`.

### Script Output
I got the following output:
```plaintext
⠆ Compiling...
⠔ Compiling 2 files with 0.8.19
⠒ Solc 0.8.19 finished in 1.08s
Compiler run successful!
Script ran successfully.
Gas used: 338569

== Return ==
0: contract SimpleStorage 0x90193C961A926261B756D1E5bb255e67ff9498A1
```
If the RPC URL was not specified, Foundry automatically launched an Anvil instance, ran my script, and then terminated the Anvil instance.

### Simulating and Broadcasting Transactions
1. **Simulate On-Chain Transactions**:
   - I ran the following command:
     ```bash
     forge script script/DeploySimpleStorage.s.sol --rpc-url http://127.0.0.1:8545
     ```
   - Output:
     ```plaintext
     No files changed, compilation skipped
     EIP-3855 is not supported in one or more of the RPCs used.
     Unsupported Chain IDs: 31337. Contracts deployed with a Solidity version equal or higher than 0.8.20 might not work properly.
     For more information, please see https://eips.ethereum.org/EIPS/eip-3855

     Script ran successfully.

     == Return ==
     0: contract SimpleStorage 0x34A1D3fff3958843C43aD80F30b94c510645C316

     Setting up 1 EVM.
     ==========================
     Chain 31337
     Estimated gas price: 2 gwei
     Estimated total gas used for script: 464097
     Estimated amount required: 0.000928194 ETH
     ==========================
     SIMULATION COMPLETE.
     To broadcast these transactions, add --broadcast and wallet configuration(s) to the previous command. See forge script --help for more.
     ```

2. **Broadcast Transactions**:
   - I ran the following command:
     ```bash
     forge script script/DeploySimpleStorage.s.sol --rpc-url http://127.0.0.1:8545 --broadcast --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
     ```

### Verifying Deployment
Switching to the Anvil terminal, I saw:
```plaintext
Transaction: 0x73eb9fb4ef7b159e03c50d669c42e2ec4eeaa9358bea0a710cb07168e5192570
Contract created: 0x5fbdb2315678afecb367f032d93f642f64180aa3
Gas used: 357088
Block Number: 1
Block Hash: 0x8ea564f146e04bb36fc27f0b491223a023b5882d2fcfce3ff85e0dd152e611e4
Block Time: "Tue, 16 Apr 2024 13:39:51 +0000"
```
