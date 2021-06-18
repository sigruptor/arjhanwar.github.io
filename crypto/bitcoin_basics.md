# How Bitcoin Works

### Analogy with Money - How Money Works
Lets say Yudh wants to send/give money to Amli, he would give money (10$ or 10 Rupees) to her and that would be it. The money is Amli's and Yudh does not have any control
over it. However Bitcoin is another form of currency which is digital. When Yudh gives that digital coin to Amli, there are few questions:
1. Yudh needs to send it to Amli, he needs Amli's address of some sort
2. Also how does Amli know if the money is coming from Yudh, what if she was expecting similar exchanges from other people
3. How do we know Yudh had that digital coin in the first place
4. How do we know Yudh is not sending the same coin to others (_**Double Spending**_)
5. How is this digital currency generated

Problem 1 and 2 are solved with asymmetric cryptography (public-private key cryptography), i.e. Yudh while creating the transaction (sending money to Amli), can sign
it with his private key. Now Amli can verify that Money (Bitcoin, digital currency) is indeed coming from Yudh by decrypting it using the public key. 
Problem 3 and 4 are solved via another concept called **Ledger**. There is no centralized authority to verify and control, it is all managed in a decentralized manner
following the peer to peer model.

### Ledger (BlockChain)
Every transaction happening in digital world is added to a common ledger (one copy for the entire network). So everyone can verify that Yudh indeed had that much money
and sent it to just Amli and not others. When Yudh is sending money to Amli, he starts a new transaction, puts Amli's public Address, signs it with his private key
and hence in process creates a block (Tx block). This block is circulated in bitcoin network (remember its decentralized network), that Yudh would send it to its 
neighbouring nodes, which would then propogate it forward (like gossip (protocol)). The user's client software will formulate the transaction and send it to a 
nearby node in the bitcoin network. The first node to hear about the transaction shares it with others until it's widely distributed throughout the network.
<p>
  Some of the nodes are miners that participate in process of actually updating the blockchain, it takes a list of transactions that are not in blockchain, validates them
  (valid signatures, sum of outputs < sum of inputs), resulting list of new, valid transactions is called a block._ The miner also adds a special transaction granting
  itself a reward (currently 12.5$) - for creating a block. This is how currency is added to the network_. But its not like all the nodes could add this block, they
  have to win the right to add the block. This involves solving a puzzle, which is time and compute intensive. e.g. they create a nonce and generate a hash from it,
  and the contraint might bee to find the lowest hash (hash which begins with 72 zeroes). Whoever finds the first block announces it to rest of the network, others verifies
  and add that block to their copy of blockchain. This way entire network converges on a single copy of blockchain (ledger) and removes the dependency on centralized
  authority or few of the powerful nodes.
</p>

#### Incentives for Miners
  Miners are awarded 12.5 bitcoins for solving the puzzle and helping validate and addition of the block. Earlier the award was 50 bitcoins, but there is a limit
  to which number of coins can be generated (22 million). Going forward reward is going to go down further and miners would just get the Tx fee, this is a big concern.

### Challanges
  
#### Scaling debate
  
- Requires lot of energy and is deemed to slow by design i.e. 1 block in 10 minutes or so. 
  Each blocks size is 1 Mb, and each Tx size is 500 bytes. So 1 block could have 2000 Txs and 3.33 Tx per second. This is way slower that what our Visa card provides.
 - Some ideates mention that we should increase the size of block, but increasing the size would require more storage on each node, which might eliminate general 
  users of the network and the power might just go in few powerful hands.
 - The idea is that we need to scale based on this small block size.
 - Consumes a lot of evergy to create/add one block, brings huge environmental concerns.

 #### Forks
  - Each of the node in network should follow the same rules, if part of network starts following other protocol, the network would diverge and it would practically 
  be impossible to converge them. Bitcoin cash is one example of one of the forks.
  
#### There to stay ?
  - First Digital currency, and fully decentralized
  - Irreversible Transaction ? Another issue ?
  - Applications on top of bitcoin network to implement anti-fraud protection etc. like big block chain exchanges.
 

















#### References
1. https://medium.com/free-code-camp/explain-bitcoin-like-im-five-73b4257ac833
2. https://arstechnica.com/tech-policy/2020/12/how-bitcoin-works/
3. https://www.lopp.net/bitcoin-information/getting-started.html
4. 
