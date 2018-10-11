# Answers

**Try not to look at the answers unless you are stuck!**

## Introduction

Q1. How many ways are there to create a new cryptocurrency other than creating one from scratch?

* Fork a current coin and tweak the parameters (eg Bitcoin Cash, Bitcoin Diamond, Bitcoin Gold)
* Create a Token Smart Contract on a platform that supports it. Eg, Ethereum, NEO, WAVES, EOS, NEM..etc  

## Chapter 1

Q1. How is cost, transaction speed and decentralisation related?

Choose 2 out of 3. A fully centralised system is the cheapest and fastest. On the other hand, a fully decentralised system is either very costly or very slow.

Q2. Would you use decentralised or centralised systems for the following use cases and why?

* Decentralisation is slow and expensive. Creation of a national identity system for its citizens.

Ans: Centralised. No need for transparency unless countries need to share data.

* A parcel tracking system for company xyz

Ans: Centralised. No need for transparency.

* Tracking of rice production between Vietnam and Australia.

Ans: Leaning towards decentralised but still debatable,dDepending on the number of nodes.

* Voting for the next goverment.

Ans: Decentralised

* Record the transaction history of lotus miles (points) of vietnam airlines.

Ans: Centralised.

Q3. Is Bitcoin fully decentralised? How do you know?

Reasonably (for pow) at the time of writing - https://www.blockchain.com/en/pools

Also consider decentralisation from the human point of view. Is there a godly figure that decides on the future of the currency? The disappearance of Satoshi means there is no central figure in Bitcoin. What about the Bitcoin core developers?

## Chapter 2

Q1. What is the unique identifier in the Block? How do we ensure that its unique?

Ans: The block hash. It will always be unique because we will always have something random (nonce) in the hash calculation.

Q2. Can you see a problem with this class?

Ans: No hash.

Q3. Why is there a need for pendingTransactions?

Ans: Because we can only pack that much of tx in each block so we need to do it batches. All pending transactions need to go into the mempool.

Q4. What is the problem with the getAddressBalance function?

* balance can be negative. 
* what if trans.value is negative? 
* What if there is an integer overflow? It doesn't mean we should implement the fix in this function, but its something we have to consider.
* The sender and receiver can be the same address.

Q5. Is it possible that 2 miners find a block at the same time? What happens then?

We have a fork and a possibility of double spending if you accept 1 block confirmation only. The first miner that finds the next block will use its history as the source of truth.

Q6. Did you see any problems with alice sending 40 tokens to bob? How do we fix it?

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

Why are we hardcoding a timestamp? Because we want to provide the same conditions for calculateHash() to find the next hash.

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

## Chapter 5

Q1. In a transaction, why is the sum of output always lesser than the sum of input?

Ans: The output needs to include the tx fees.

Q2. Write the code for the getAddressBalance function in Blockchain.js

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

Q3. Refer to the createTransaction endpoint in main.js. What problem can you see with the code here?

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

Q4. Refer to the createTransaction endpoint in main.js. What problem can you see with the code here?

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

## Chapter 6

Q1. In the mineBlock function, is it ok to run `block = Blockchain.runSmartContract(block)` after `block.mineBlock(this.difficulty)` ?

Ans: Before a block is mined, we can check that the smart contract is doing the right things and stop it if it doesn't. After a block is mined, it should be immutable and be propagated straight away, so even if the smart contract not doing anything illegal, there might be a chance that your node and other nodes will not be in synced.

Q2. Let's just say Mycoin Script is a new upgrade to Mycoin. Can you see any problems with this upgrade?

Ans: No matter how fantastic this upgrade is, in a fully decentralised network, all machines have the rights not to upgrade. In that case, you are left with a fork, ie a split chain which is highly undesirable. Look at what happened to Ethereum Classic and Ethereum due to the DAO Fork. This is a much more serious issue to consider in a public rather than a private chain.