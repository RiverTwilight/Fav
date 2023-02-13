> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [helpthisbook.com](https://helpthisbook.com/sunny/the-newbies-guide-to-crypto/89be5a1e-a223-4c46-ac56-718f56baa112)

> The public ledger that is the secret sauce behind web3. What it is and why it’s so important.

The public ledger that is the secret sauce behind web3. What it is and why it’s so important.

![](http://dbsrh6r8wal74.cloudfront.net/b9ca8f92-a4b1-45be-800a-1db1b0805f20.png)﻿  
﻿  

Since a Blockchain is a series of blocks, organised in a chain, that contain information (bet you're glad you started reading this revolutionary stuff), this article is going to be organised in the same way.

Let's start with our first block...

Block 1: The Basics

![](http://dbsrh6r8wal74.cloudfront.net/a7cd45d3-c6a9-467b-bc8f-316099b40711.png)﻿  
﻿  

We'll start with the example of signing up to Facebook. When you sign up with your name, email and any other information you need to provide, this data is stored in a database. That database is privately owned by Facebook. They decide what to do with your data. When you sign up, you are having to trust Facebook with your data. If Facebook is hacked, the hackers may be able to alter your data.

In contrast to this, a blockchain is a decentralised, distributed ledger. It's completely public. All entries to the ledger can be seen, and no one owns the information on the blockchain.

"Distributed" means that the current state of the ledger is replicated across many different machines. Together, those machines agree by consensus on what the current state of the ledger is and validate any changes being made to the ledger. The idea is that the blockchain records information in such a way that it is very difficult to cheat or hack. Once some data has been recorded inside a blockchain, it becomes very difficult to change.

Block 2: Inside a Block

![](http://dbsrh6r8wal74.cloudfront.net/605132de-94fb-4e1b-afbb-21968e41ca79.png)﻿  
﻿  

Each block contains some data, the hash of the block, and the hash of the previous block. A hash is like the fingerprint of a block. It identifies that block's contents and it is always unique. Changing the data inside the block will cause the hash to change. If the hash changes, that block is no longer the same block it was before.

![](http://dbsrh6r8wal74.cloudfront.net/6b6c582d-697e-43c1-81b2-14f09935f3f6.png)﻿  
﻿  

Block 3: Creating a Chain

![](http://dbsrh6r8wal74.cloudfront.net/c1d348e1-3b28-4481-83c9-2b7cb340902d.png)﻿  
﻿  

The third element we mentioned that goes into each block is the hash of the previous block. This is what is used to create a chain. Every block except the very first one points to a previous block. So what happens if you try to tamper with one block in the chain? The data inside it changes, so the hash changes. All of the following blocks in the chain are now invalid because the next block is pointing back to a hash that doesn't exist anymore.

What about the first block?

The first block is a little bit special, and the name for it is the Genesis Block or Block 0. It can't point back to a previous block because it is the first one in the chain. Every subsequent block in the chain can be traced back to this genesis block.

Block 4: Consensus Mechanisms

![](http://dbsrh6r8wal74.cloudfront.net/869dfd98-9427-4107-ac2d-6da7b960259b.png)﻿  
﻿  

But hashes are not enough. Modern computers are very fast. They have the processing power to tamper with a block and then recalculate all of the blocks further down in the chain. To protect against this, blockchains have consensus mechanisms. These are processes that slow down the creation and check the validity of a new block, before adding it to a chain. Before a block is added, over 50% of the validator nodes in the network have to agree that this block is valid. This makes it very hard to tamper with a blockchain.﻿

Side Note: Nodes

These blocks of data we are discussing are stored on nodes. On a blockchain network, all nodes are connected, and they use these connections to continuously exchange the newest information with each other. Nodes can be any kind of device, but they are usually computers, laptops and servers.

Block 5: Adding a Block

![](http://dbsrh6r8wal74.cloudfront.net/52eff1fa-d604-4e5f-8b6b-e4e6d22c6cd5.png)﻿  
﻿  

When a new block is created, it is sent to everyone in the network. The validator nodes then verify the block, to make sure it hasn't been tampered with. The nodes agree on which blocks are valid and which aren't. Invalid blocks are rejected by all nodes in the network. If everything is okay, each node adds this block to its blockchain.

Block 6: Tamper Proof?

![](http://dbsrh6r8wal74.cloudfront.net/088c6379-c242-446b-880a-97247ff27564.png)﻿  
﻿  

So to successfully hack a blockchain - these are the steps. Tamper with all the blocks on the blockchain, redo the hash calculation and the consensus mechanism process for each block, and then you need to somehow take control of over 50% of the nodes in the globally distributed network - so that the network accepts your tampered blocks. If you think about a large, global network like Bitcoin or Ethereum, this is a very, very difficult task.

Block 7: Smart Contracts

![](http://dbsrh6r8wal74.cloudfront.net/4f5ed2bf-3731-4b38-b786-c0bd51a78a63.png)﻿  
﻿  

Smart contracts are programs stored on the blockchain. They can be used to automatically exchange data if certain conditions are met. The terms of the agreement between the parties involved are directly written into the code.

One example of a smart contract could be the use case of life insurance. In the event of someone passing, a death certificate could be the input needed to trigger the smart contract. Once the smart contract is triggered, it would automatically pay out funds to the beneficiaries named in the life insurance agreement.

Block 8: Uses

![](http://dbsrh6r8wal74.cloudfront.net/90699102-9d2f-4814-9e4a-4dfc0981c53e.png)﻿  
﻿  

Digital currency may be where all the hype is. But those in the web3 world have much wider-ranging plans for the uses of blockchain technology. Blockchain technology and the highly secure, public chain of record it produces can be used for:

1.  Fairer voting systems
    
2.  Secure sharing of medical data
    
3.  Personal identity security
    
4.  Recording real estate transactions
    
5.  Gambling
    

Block 9: Creating your own

![](http://dbsrh6r8wal74.cloudfront.net/70f9a50a-5773-4885-a1d6-dc63ad32669b.png)﻿  
﻿  

This is kind of going from 0 to 100, but if you have basic programming skills, you may be interested in this video on how to create your own blockchain in Javascript. Even non-technical people can use this video to understand more because the explanations are very clear.

[https://www.youtube.com/watch?v=zVqczFZr124](https://www.youtube.com/watch?v=zVqczFZr124)﻿

So that was our blockchain to explain blockchains (how meta). Hopefully, you now understand more about why the words "decentralised" and "trustless" are being thrown around so much these days. The main principles of blockchain networks are that they are public, distributed, time-stamped and persistent (meaning that blockchain records can't be misplaced or damaged in any way). The most important lesson here is that blockchain technology is not some obscure thing related to Bitcoin. It's likely to affect many aspects of your life in future.