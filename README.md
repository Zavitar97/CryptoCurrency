# Cryptocurrency
A Cryptocurrency System, including Complete Blockchain Implementation, Nodes, Miners, Digital Wallets and Network API's.

## Features
* Complete Blockchain Implementation.
* Full Nodes
* Miners
* Digital Wallets.

## Model
The currency's system model is similar to other known currencies such as Bitcoin or Ethereum.
The blockchain is composed of blocks, referenced by their hash value. Each block contains a block header and block data. 

The block's data is a list of transactions. Each transaction is composed of (1) the transaction's general and authentication data (2) inputs- a list of unspent outputs signed by their owner's private key (3) outputs- a list of recipients' addresses and amount of currency to be spent. The transaction and its data are digitally signed.

Nodes are used to maintain a blockchain copy, track and serve its data to users, and verify new blocks. Miners are used to mine new blocks (using the proof work method) containing newly verified transactions.

## How it works
### Blockchain
Create a blockchain instance:
```Python
from blockchain import BlockChain
# creates new chain
chain = BlockChain()
```
Create initial transactions in order to use the blockchain. Use it only with a completely new blockchain, since other nodes will not approve this new block for their own copy as there are no existing unspent outputs to support new transactions. Otherwise, you will have to apply this action manually for all nodes.
```Python
from miner import mine_initial_txs
# list of tuples containing pairs of recipient address and amount
recipients = [('efdsf2', 1), ('das676df', 0.8), ('pmawq44', 0.2)]
prev_hash = chain.get_blocks(-1).hash_block()
mine_initial_txs(prev_hash, recipients)
```
#### API
While the system manages all the processes automatically, users can use the API as they wish.
```Python
# Generates a Genesis Block
chain.create_genesis()
# Adds new block (Block instance) to the chain
chain.add_block(block)
# Verifies the block chain. Also verifies with a new block if provided
chain.verify_chain(new_block=None)
# Returns block by index
chain.get_block(index)
# Returns last block
chain.get_last()
# Returns last index
chain.get_last_index()
# Returns last hash value
chain.get_last_hash()
# Returns all transactions from the blocks in a list
chain.fetch_all_tx()
# Returns a copy of the block chain
chain.copy()
# Returns blockchain in bytes
chain.serialize()
# Returns blockchain in JSON format
chain.json()
# Returns chain's iterator
iter(chain)
# Returns the size of the chain
len(chain)
```
### Nodes
Nodes holds a blockchian copy and responsible for: 
* Serve chain's blocks and it's data (transactions).
* Track and confirm unspent transaction outputs (UTXO).
* Verify new transactions.
* Verify and add new blocks to the chain.
* Broadcast new data to other nodes in the network.

In order to broadcast and distribute new data a node must be provided with a nodes peers network. If such a network isn't provided the node will not forward the data.
A peers network simply consists of an array of tuples of the size of the network, where each tuple contains a peer's ip address and port number.
```Python
from node import Node

peers = [('104.245.162.198', 62541), ('90.99.183.255', 62541), ('81.180.92.198', 61380)]
node_id = 'node123'
# creates a node
node = Node(node_id, chain, peers)
# use the node as a server
ip = '127.0.0.1'
port = 64123
node.run(ip, port)
```
#### API
```Python
# Returns blocks by their respective index
node.get_blocks(indexes)
# Returns unspent transaction outputs by their respective address
node.get_utxo_by_address(addresses)
# Validates given transactions
node.validate_transactions(txs)
# Validates block and its transactions
node.validate_block(block)
# Verifies and adds new block, then broadcasts it across the nodes network
node.new_block(block)
# Returns chain's length
node.get_chain_len()
# Serializes blockchain in bytes
node.serialize_chain()
# Returns chian in JSON format
node.json_chain()
```
### Miners
Miners receive and verify new transactions and mine new blocks using the PoW method.
```Python
from miner import Miner
# miner's address, to which the mining fee will be sent 
miner_address = 'ef16fdsfe'
# mode 0 - using node instance
# mode 1 - connecting to the node server
# node is either a Node instance or tuple of node server's ip and port
miner = Miner(miner_address, node, 0) # using node instance

# send new transaction. tx = new transaction
miner.handle_transaction(tx)
```
Miner as server:
```Python
miner.run_server('195.185.48.25', 65412) # ip, port

# connect to miner server as client
from miner import MinerClient
client = MinerClient()
client.connect('195.185.48.25', 65412)
# send transaction
client.send_tx(tx)
```
### Wallets
Digital wallets manage keys and addresses, send/receive new transactions and store transactions data such history, balance and available UTXO's. Wallets must be connected to a Node in order to extract relevant data from the blockchain. Wallet may also connect to a miner (or any server that connects to a miner), to which it will send new transactions to be verified and inserted into a block. This option is optional, and in such case new transaction will have to be sent manually to the miner.
```Python
from wallet import Wallet

# node and miner connection addresses samples
node_address = ('82.254.148.222', 64123)
miner_address = ('175.171.218.10', 52132)

wallet = Wallet(node_address, miner_address)
# creates new address
address = wallet.new_address()
# Returns utxo for given addresses. If no addresses are provided, the method returns for all the addresses in the wallet
unspent_txs = wallet.get_unspent()

# create transaction
# list of tuples containing pairs of recipient address and amount
recipients = [('efdsf2', 1), ('das676df', 0.8), ('pmawq44', 0.2)]
# parameters are sender address (optional- used for additoinal signature), recipients list, total amount and miners fee
# the method returns the new transaction in case no miner was provided
wallet.create_transaction('eewr', recipients, 2, 0.01)

# Fetch new transactions for given addresses from Node and adds it to history
wallet.receive_transaction()
```
Use wallets as a server and communicate it with a client, which provides the capability to use it from another machine/socket or a client written in any programming language other than python.
```Python
wallet_server = WalletServer(wallet)
wallet_server.start('127.0.0.1', 45174)
```
The protocol uses requests in JSON format and is composed of 'type' fields  and parameters fields (depends on the request).
```Python
# types
UTXO # parameter is addresses list or None
ADDRESSES # no parameters. returns key addresses
NEW_TX # parameters are utxo's, recipient addresses and their amount
RECEIVE # parameters are addresses or none to receive for all addresses
WALLET_DETAILS # no parameters. Returns all wallet's details
NEW_ADDRESS # no parameters. Returns new address from wallet
```
### Web client
