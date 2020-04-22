# poa_developmentchain
Developing a testnet blockchain for an organization using an Power of Authority. 

# Environment set up for Mac OS

## Create ETH environment
conda create -n zbank python=3.7
source activate zbank
brew tap ethereum/ethereum
brew install ethereum
pip install web3
pip install -U python-dotenv

# Creating the testnet blockchain

## Create a Geth Folder

- mkdir poa_developmentchain

## Create two new accounts using your neumonic phrase in myCrypto and choose the top two accounts. 

## Create Genesis Block

- Type puppeth to start puppeth terminal
- Name Network = zBank
- 2 to configure new genesis
- 1 to create new genesis
- 2 for proof of authority consensus algorithm
- 5 seconds
- Get your wallet account address
- Enter until you get to chain/network ID and create new network ID ex. random
- Genesis Block is created now 

## Manage existing Genesis Block

- 2 manage existing genesis config
- 2 export genesis config
- Enter and it will save folders
- Exit puppeth using ctrl C


## Create two nodes

### Create node1

- 1st node
- geth account new --datadir node1
- Enter password: 12345
- Node 1
- Enter
- Enter

## Create node 2 (in the same shell)

- 2nd Node
- geth account new --datadir node2
- Enter password: 12345
- Node 2
- Enter
- Enter

## Initialize nodes
- We need to initialize
- geth init zBank.json --datadir node1
- geth init zBank.json --datadir node2
- (The nodes are not sequential)


## Starting the blockchain

- Node1
- geth --datadir node1 --mine --minerthreads 1 (minerthread tells how many (processors) threads we are using)
- Get the enode address from the minethread above. (hint: it starts with enode://)
- Node2
- Open a new terminal and enter command below with the enode above
- geth --datadir node2 --port 30304 --rpc --bootnodes "enode address"

### Once this complete let the nodes continue to mine in the background. 


# Creating testnet Ethereum network in myCrypto

## Create a custom testnet network and node in myCrypto

- Open myCrypto

- Go to "Create New Wallet"

- "Generate New Wallet"

- "Generate Keystore File" and save to Geth Folder

- Create a password and click "Generate New Wallet." Make sure you save your password!

- Go to "Change Network"

- Go to "Add Custom Node"

- Name the network in the "Node Name" section zbank and enter the URL: http://127.0.0.1:8555

# Sending a transaction

## Download these libraries

import os
from web3 import Web3
from dotenv import load_dotenv
from web3.middleware import geth_poa_middleware
from eth_account import Account
from pathlib import Path
from getpass import getpass

## Create .env file to store private key

## Create the raw text function using account one (Account one is the account the currency is leaving)

load_dotenv()
w3 = Web3(Web3.HTTPProvider("http://127.0.0.1:8545"))
w3.middleware_onion.inject(geth_poa_middleware, layer=0)
account_one = Account.from_key(os.getenv("PRIVATE_KEY"))
with open(
    Path(
        "UTC--2020-04-18T16-28-43.263Z--6fb811681e86c54486ccd862c8db67e0e6dc5b2d"
    )
) as keyfile:
    encrypted_key = keyfile.read()
    private_key = w3.eth.account.decrypt(
        encrypted_key, getpass("Enter keystore password: ")
    )
    account_two = Account.from_key(private_key)
def create_raw_tx(account, recipient, amount):
    gasEstimate = w3.eth.estimateGas(
        {"from": account.address, "to": recipient, "value": amount}
    )
    return {
        "from": account.address,
        "to": recipient,
        "value": amount,
        "gasPrice": w3.eth.gasPrice,
        "gas": gasEstimate,
        "nonce": w3.eth.getTransactionCount(account.address),
    }
print(create_raw_tx(account_one, account_one.address, 10))

## Create the send function using account two (Account two is the account the currency is being sent)

def send_tx(account, recipient, amount):
    tx = create_raw_tx(account, recipient, amount)
    signed_tx = account.sign_transaction(tx)
    result = w3.eth.sendRawTransaction(signed_tx.rawTransaction)
    print(result.hex())
    return result.hex()

send_tx(account_one, '0x6d2DE9cbB155a9D949cd51C7865dc7D48d050B21', 10)