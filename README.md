# LotusChain
Vietnam Blockchain Layer 1
// File: src/blockchain.js
const crypto = require('crypto');

class Block {
  constructor(index, previousHash, timestamp, data, hash, validator) {
    this.index = index;
    this.previousHash = previousHash;
    this.timestamp = timestamp;
    this.data = data;
    this.hash = hash;
    this.validator = validator;
  }
}

class Blockchain {
  constructor() {
    this.chain = [this.createGenesisBlock()];
    this.validators = new Map(); // Map of validator addresses to reputation scores
    this.shards = new Map(); // Map of shard IDs to blockchain instances
    this.storage = { hot: [], cold: [] }; // Layered storage
  }

  createGenesisBlock() {
    return new Block(0, '0', Date.now(), 'Genesis Block', this.calculateHash(0, '0', Date.now(), 'Genesis Block'), 'genesis');
  }

  calculateHash(index, previousHash, timestamp, data) {
    return crypto.createHash('sha256').update(index + previousHash + timestamp + JSON.stringify(data)).digest('hex');
  }

  addValidator(address, reputation = 100) {
    this.validators.set(address, reputation);
  }

  selectValidators() {
    // Simulate Vietnam-centric validator selection (e.g., based on reputation and compliance)
    return Array.from(this.validators.entries())
      .sort((a, b) => b[1] - a[1])
      .slice(0, 3)
      .map(entry => entry[0]);
  }

  addBlock(data, shardId = 'main') {
    if (!this.shards.has(shardId)) {
      this.shards.set(shardId, [this.createGenesisBlock()]);
    }
    const shardChain = this.shards.get(shardId);
    const previousBlock = shardChain[shardChain.length - 1];
    const validator = this.selectValidators()[0]; // Simplified: pick top validator
    const newBlock = new Block(
      previousBlock.index + 1,
      previousBlock.hash,
      Date.now(),
      data,
      this.calculateHash(previousBlock.index + 1, previousBlock.hash, Date.now(), data),
      validator
    );
    shardChain.push(newBlock);
    this.storage.hot.push(newBlock); // Store in hot storage
    if (this.storage.hot.length > 10) { // Pruning simulation
      this.storage.cold.push(this.storage.hot.shift());
    }
    return newBlock;
  }

  takeSnapshot(shardId = 'main') {
    return JSON.stringify(this.shards.get(shardId));
  }
}

module.exports = Blockchain;

// File: src/api.js
const express = require('express');
const Blockchain = require('./blockchain');

const app = express();
app.use(express.json());
const blockchain = new Blockchain();

app.post('/block', (req, res) => {
  const { data, shardId } = req.body;
  const block = blockchain.addBlock(data, shardId);
  res.json(block);
});

app.get('/snapshot/:shardId', (req, res) => {
  const snapshot = blockchain.takeSnapshot(req.params.shardId);
  res.json({ snapshot });
});

app.listen(3000, () => console.log('API running on port 3000'));

// File: README.md
# Lotus Chain - Vietnam Layer 1 Blockchain

## Overview
Lotus Chain is an open-source Layer 1 blockchain designed for Vietnam’s digital economy, society, and government. It features a Hybrid DPoS consensus, layered storage, sharding, and open APIs.

## Installation
1. **Prerequisites**:
   - Node.js (v16 or higher)
   - npm (v7 or higher)
2. **Clone Repository**:
   ```bash
   git clone https://github.com/lotuschain/lotuschain.git
   cd lotuschain
   ```
3. **Install Dependencies**:
   ```bash
   npm install express crypto
   ```
4. **Run the Blockchain**:
   ```bash
   node src/api.js
   ```
5. **Test API**:
   - POST `/block`: Add a block (e.g., `{"data": {"tx": "example"}, "shardId": "main"}`)
   - GET `/snapshot/:shardId`: Retrieve snapshot for a shard

## License
MIT License (see LICENSE file)

## Repository
https://github.com/lotuschain/lotuschain (placeholder URL)
