# Answers

**Try not to look at the answers unless you are stuck!**

## Introduction

Short Quiz 1. How many ways are there to create a new cryptocurrency other than creating one from scratch?

* Copy the code of a cryptocurrency and tweak the parameters (eg Bitcoin Cash, Bitcoin Diamond, Bitcoin Gold)
* Create a Token Smart Contract on a platform that supports it. Eg, Ethereum, NEO, WAVES, EOS, NEM..etc  

Short Quiz 2. Which of the following is/are true for all cryptocurrencies currently?

Ans: It is faster to use cryptocurrency for cross border funds transfer as compared to using traditional banking system.

## Chapter 1

Short Quiz  Q1. A Distributed Ledger can be very decentralised, cheap to use and fast.

The [project management triangle](https://en.wikipedia.org/wiki/Project_management_triangle) applies. Choose 2 out of 3. A fully centralised system is the cheapest and fastest. On the other hand, a fully decentralised system is either very costly or very slow.

In distributed system, the [CAP theorem](https://cryptographics.info/cryptographics/blockchain/cap-theorem/) applies.

Short Quiz Q2. Would you use decentralised or centralised systems for the following use cases and why?

* Creation of a national identity system for its citizens.

Ans: Centralised. No need for transparency unless countries need to share data. Decentralisation is slow and expensive. We should not use it unless necessary. 

* A parcel tracking system for company xyz

Ans: Centralised. No need for transparency.

* Tracking of rice production between Vietnam and Australia.

Ans: Leaning towards decentralised but still debatable, depending on the number of nodes and governance.

* Voting for the next goverment.

Ans: Decentralised

* Record the transaction history of lotus miles (points) of vietnam airlines.

Ans: Centralised. Unless there is a point swapping system with other airlines.

Short Quiz Q3. Bitcoin is centralised because the core developers have full control over the source code.

False. It's worth noting that there are different levels of decentralisation. From the infrastructure point of view, it is Reasonably decentralised (pow) at the time of writing - https://www.blockchain.com/en/pools

Even if core developers decided to inject a bug into the system, the nodes don't have to upgrade.

## Chapter 2

Q1. What is the unique identifier in the Block? How do we ensure that its unique?

Ans: The block hash. It will always be unique because we will always have something random (nonce) in the hash calculation.

Q2. Do you see a problem with the Transaction class?

Ans: No hash.

Q3. Why is there a need for pendingTransactions?

Ans: Because we can only pack that much of tx in each block so we need to do it batches. All pending transactions need to go into the mempool.

Q4. What is the problem with the getAddressBalance function in the Blockchain class?

* balance can be negative. 
* what if trans.value is negative? 
* What if there is an integer overflow? It doesn't mean we should implement the fix in this function, but its something we have to consider.
* The sender and receiver can be the same address.

Q5. Did you see any problems with alice sending 40 tokens to bob? How do we fix it?

Ans: Add a check before we create the transaction.

```
...
app.post('/createTransaction', (req, res) => {

    // check balance
    if (mycoin.getAddressBalance(req.body.fromAddress) - req.body.value < 0) {
        res.send('Not enough funds!\n')
        return false
    }
    ...
```

Short Quiz Q1. What happens when 2 miners find a block at the same time?

Nothing happens, the network continues to have 2 split chains. We have a fork and a possibility of double spending if you accept 1 block confirmation only. The first miner that finds the next block will use its history as the source of truth.

Short Quiz 02. How can one person double spend in Bitcoin?

The attacker sends the coin to an address and receives a tx id. It then builds a fork that does not contain the tx id and hope that this fork will become the longest chain in the network.

## Chapter 3

Q1. Write a mineBlock function that accepts a difficulty parameter in the Block class. this.hash must also be guaranteed unique.

```
# src/chapter_03/Block.js

const CryptoJs= require("crypto-js");

class Block {
    constructor(timestamp, transactions, previousHash = '') {
        this.previousHash = previousHash;
        this.timestamp = timestamp;
        this.transactions = transactions;
        this.hash = this.calculateHash();
        this.nonce = 0;
    }

    calculateHash() {
        return CryptoJs.SHA256(this.previousHash + this.timestamp + JSON.stringify(this.transactions) + this.nonce).toString();
    }

    mineBlock(difficulty) {
        while (this.hash.substring(0, difficulty) !== Array(difficulty + 1).join("0")) {
            this.nonce++;
            this.hash = this.calculateHash();
        }
    }
}

module.exports = Block;
```

Q2. In Blockchain.js, add a mineBlock function that accepts a minerAddress argument. Add a new transaction before all other pending transactions. This is the coinbase transaction to reward the miner only.

```
    ...
    mineBlock(minerAddress) {
        // coinbase transaction
        this.pendingTransactions.unshift(new Transaction(null, minerAddress, this.miningReward))

        // create new block based on hardcoded timestamp, all pending tx and previous blockhash
        let block = new Block(1535766955, this.pendingTransactions, this.getBlock(this.getBlockHeight()).hash)
        block.mineBlock(this.difficulty)

        this.chain.push(block)
        // reset pending Transactions
        this.pendingTransactions = []
        return block
    }
    ...
```

Short Quiz 1. Refering to the code below, why are we hardcoding a 1535766955 as timestamp when creating mining the block? 

```
let block = new Block(1535766955, this.pendingTransactions, this.getBlock(this.getBlockHeight()).hash)
block.mineBlock(this.difficulty)
```

Ans: C) Because we hope to have a predictable block hash (calculateHash()).

Short Quiz 2. In Proof-of-Work, which of the following is false?

Ans: Any new node must be approved by the network

Short Quiz 3. Not true. Mining a full block of transactions is not significantly harder than mining an empty Block. Think of adding extra parameters into a function, it doesn't make finding the nonce any harder. The only benefit of a spy miner is milliseconds of head start in finding the next block.

Short Quiz 4. True. The biggest damage that a spy miner can do is to delay the transaction speed.

Short Quiz 5. True. Because bitcoin reward will keep halving until it reaches 21 million total circulation supply sometime in 2140. At that time, the only motivation of mining empty blocks is to disrupt the chain. There are no more mining reward, ie the only reward is from transaction fees. At that time, mining empty blocks will be expensive and serves has no monetary incentives.

## Chapter 4

Q1. Why is Bitcoin address(eg 367f4YWz1VCFaqBqwbTrzwi2b1h2U3w1AF) much shorter than the public key we generated?

Because there are several hashing operations done on the public key, ie sha256 and ripemd160.

See https://en.bitcoin.it/wiki/Technical_background_of_version_1_Bitcoin_addresses

Q2. Proof that this signature is a signed "hello" message from alice

```
const wallet = require('./Wallet')

let sig = '{"r":"67a63bac0a761021b38c8d5ce602bc2de3210bd236f6b59539b3526395ad2ec8","s":"ed2edaeb2e1893c78a460a4055ca1f6cae41e1eca6da7ae5ffc44981c33c3c97","recoveryParam":1}'

let res = wallet.verifySignature('hello', JSON.parse(sig), '04c7facf88f8746f4388bcd1654a43afff83e5552a4b723352b5547cd5ba021e55ea4014c5cdec3133652f93a6d032b394387c487ed881cee5ac232bbc754cddec')

console.log(res)
```

Q3. Where would you make changes in the code to pre-mine some mycoin to alice? Pre-mine 30 coins to alice.

```
# src/chapter_04/Blockchain.js

class Blockchain {

    ...
    createGenesisBlock() {
        let tx = new Transaction(null,'04c7facf88f8746f4388bcd1654a43afff83e5552a4b723352b5547cd5ba021e55ea4014c5cdec3133652f93a6d032b394387c487ed881cee5ac232bbc754cddec', 30)
        // why do we need to hardcode a time?
        return new Block(1535766956, [tx], "0")
    }
    ...
}
```

Q4. Add logic to createTransaction API endpoint in main.js to check that sender is allowed to spend the funds before adding the transaction to the pending transactions queue.

```
# src/chapter_04/main.js

...
const wallet = require('./Wallet')
...
    app.post('/createTransaction', (req, res) => {

        // check balance
        if (mycoin.getAddressBalance(req.body.fromAddress) - req.body.value < 0) {
            res.send('Not enough funds!\n')
            return false
        }

        let tx = new Transaction(req.body.fromAddress, req.body.toAddress, req.body.value)

        // Let us sign the tx with the private key
        let sig = wallet.sign(tx.hash, req.body.privKey)
        // if the signer is an owner of the address, we insert it in the pending tx queue
        if (wallet.verifySignature(tx.hash, sig, req.body.fromAddress)) {
            mycoin.createTransaction(tx);
            let pendingTx = JSON.stringify(mycoin.getPendingTransactions())
            broadcast(pendingTx)
            res.send('Current Pending Txs: '+pendingTx+'\n')
        }
        else {
            res.send('You are not the owner of the funds!')
            return
        }
    });
...
```

Can you see a problem with injecting the private key into the payload? How can we avoid this?

Tip: Remember to update createGenesisBlock in Blockchain.js as the Transaction constructor now accepts 5 parameters instead of 3.

```
# src/chapter04/Blockchain.js

...
    createGenesisBlock() {
        // why do we need to hardcode a time?
        let tx = new Transaction(null,'04c7facf88f8746f4388bcd1654a43afff83e5552a4b723352b5547cd5ba021e55ea4014c5cdec3133652f93a6d032b394387c487ed881cee5ac232bbc754cddec', 30,'',1535766956)
        return new Block(1535766956, [tx], "0")
    }
...
```

Short Quiz 1. Which of the following option is the safest way to prevent your private keys being stolen?

Ans: Write your key down on a piece of paper yourself and hid it somewhere.

Other options include key sharding, ie split your key into chunks and give them to different people for safe keeping.

Short Quiz 2. Revealing your wallet's public key provides a small chance for other people to guess your private key.

Ans: false

Short Quiz 3. Which of the following is the exact HD wallet path for the first child address of the first Ethereum Classic account?

Ans: C) m/44/61/0/1/1 has the closes answer. 

A possible answer could also be m/44/61/0/0/1

## Chapter 5

Q1. Write the code for the getAddressBalance function in Blockchain.js

```
getAddressBalance(address){
    let balance = 0;
    let utxos = this.getUTXO(address)
    for (const utxo of utxos) {
        balance += utxo.value
    }
    return balance;
}
```

Q2. Refer to the createTransaction endpoint in main.js. What problem can you see with the code here?

```
// Get the UTXO and decide how many utxo to sign.
let utxos = mycoin.getUTXO(req.body.fromAddress)
let accumulatedUTXOVal = 0
let requiredUTXOs = []
for(const utxo of utxos) {
    requiredUTXOs.push(utxo)
    accumulatedUTXOVal += utxo.value
    if (accumulatedUTXOVal - req.body.value > 0) {
        break
    }
}
```

There is no problem with the code. Its worth noting that the sequence of utxo is important. Imagine you are sending 21 mycoin to someone, would you spend 1 output of 30 mycoin or 2 outputs of 10 and 20 mycoin? There is a big science on it - https://medium.com/@lopp/the-challenges-of-optimizing-unspent-output-selection-a3e5d05d13ef

It could be really expensive to send large amount of small outputs. Think of the hassle of having large amount of 1 cent in your pocket. Bitcoin nodes will reject transactions that are larger than 100KB in size. The size of Bitcoin block is limited to 1 MB and you have to pay higher tx fees if there are lots of inputs and outputs.

It is recommended to consolidate multiple utxo addresses with small value into 1 address during low peak period.

Q3. Refer to the createTransaction endpoint in main.js. What problem can you see with the code here?

```
// lets spend the the UTXO by signing them. What is wrong with this?
for(let requireUTXO of requiredUTXOs) {
    // why is this a bad practice?
    let sig = wallet.sign(requireUTXO.txOutHash, req.body.privKey)
    if (wallet.verifySignature(requireUTXO.txOutHash, sig, req.body.fromAddress)) {
        let txIn = new TxIn(requireUTXO.txOutHash, requireUTXO.txOutIndex, sig)
        requiredUTXOValue += requireUTXO.value
        txIns.push(txIn)
    }
    else {
        res.send('You are not the owner of the funds!')
        return
    }
}
```

You are exposing your private key in via curl. You lose your private key, you lose the world.

Short Quiz Q1. In a Bitcoin transaction, why is the sum of output always lesser than the sum of input?

Ans: The output needs to include the tx fees.

Short Quiz Q2. What happens if Alice uses 1 UTXO (30 mycoins) to send 1 mycoin to Bob and doesn't provide change address?

Ans: The miner gets Alice's 29 mycoins.

## Chapter 6

Q1. In the mineBlock function, is it ok to run `block = Blockchain.runSmartContract(block)` after `block.mineBlock(this.difficulty)` ?

Ans: Before a block is mined, we can check that the smart contract is doing the right things and stop it if it doesn't. After a block is mined, it should be immutable and be propagated straight away, so even if the smart contract not doing anything illegal, there might be a chance that your node and other nodes will not be in synced.

Q2. Let's just say Mycoin Script is a new upgrade to Mycoin. Can you see any problems with this upgrade?

Ans: No matter how fantastic this upgrade is, in a fully decentralised network, all machines have the rights not to upgrade. In that case, you are left with a fork, ie a split chain which is highly undesirable. Look at what happened to Ethereum Classic and Ethereum due to the [DAO Fork](https://www.coindesk.com/ethereum-executes-blockchain-hard-fork-return-dao-investor-funds/). This is a much more serious issue to consider in a public rather than a private chain.

There are 2 types of fork.

“Soft" (Soft Fork) changes tighten up the rules - old software will accept all the blocks and transactions created by new software, but the opposite may not be true. "Soft" changes do not require the entire network of miners and merchants and users to upgrade or be left behind. 

"Hard" (Hard Fork) changes modify the rules in a way that old, un-upgraded software consider illegal. At this point it is much, much more difficult (some might say impossible) to roll out "hard" changes, because they require every miner and merchant and user to upgrade.

For fork, we have also be careful about [replay attack](https://blockonomi.com/replay-attacks/).

Short Quiz 1. In stack programming, What is the following `2 2 2 2` returns?

Ans: Based on LIFO, it returns 2

Short Quiz 2. What are the benefits of Non-Turing Complete languages

* More predictable resource usage
* lesser loopholes for hacks to occur
* Easier to debug

## Chapter 7

Q1. In the POW system, given each transaction as 249 bytes and if it takes an average of 10 mins to mine a block, find the transaction speed of Bitcoin per second?

Ans: Assuming each block is 1 MB, and 10 mins is 600 secs, ans is 1000000 / 249 / 600 = 6.7 tx/s

Short Q1. How does a crypto exchange provide fast Bitcoin to Litecoin trading since we know that Bitcoin cannot scale at the moment?

A) The trading doesn't happen on the chain, its just fake manipulation of database values.

Short Q2. You created a transaction in the Bitcoin network but realised your fee was too low. Due to its low scalability, what is the best way to ensure that your transaction is mined as soon as possible.

D) Try to spend one of the unconfirmed output from the previous transaction with a higher tx fee.