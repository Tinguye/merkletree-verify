# Silver Sixpence Merkle Tree Generator

A Merkle Tree is a privacy-preserving data structure that uses hash proofs to store and manage large datasets. 
It uses these hash functions to construct layers of nodes that build a tree-like structure with several layers of decreasing size, until only a single node remains. This node is called the Merkle Root.
There is no industry standard on how to generate Merkle Trees, so many different customized versions may exist. 
The Silver Sixpence Merkle Tree Generator uses its own approach, but can easily be modified on several parameters based on customer demand. 

Our standard methodology is as follows:
The Silver Sixpence Merkle Tree Generator generates the leaf nodes by concatenating a client ID, a random salt, an Audit ID and asset balances. It then calculates a SHA256 hash on the result:

Client_ID: Unique client identifier. Can be an ID number or hash of the username

Example: ```Client_ID = "287e29b8-de5b-4924-8906-b216f2d48cd6"```

Client_Balances = (Asset1=Balance | Asset2=Balance | Asset3=Balance ...)

Example: ```Client_Balances = "BTC=76.83|ETH=19.26|XRP=68.95|USDC=6.32"```

Salt is a random string of arbitrary length to ensure the uniqueness of all leaves.

Example: ```Salt = "2A496ECE"```

Audit_ID is a unique identifier for the audit that took place.

Example: ```Audit_ID = "PORNOV22"```

Account_Record = concat(Account_ID + Salt + Audit_ID + Client Balances)

Example: ```Account_Record = "287e29b8-de5b-4924-8906-b216f2d48cd62A496ECE PORNOV22BTC=76.83|ETH=19.26|XRP=68.95|USDC=6.32"```

Leaf = SHA256(Account_Record)


Not every element mentioned above is necessary for every audit. Either a Salt or Audit_ID value is needed so that static Account Records do not hash to the same value during different audits, but if both values are included then it will add redundancy.

The Merkle Tree is constructed by concatenating groups of these leaf nodes together and hashing the result. The size of the group is called the Merkle Width and is typically 2. The height of the Merkle Tree refers to the number of layers between the leaves and the Root, and is a function of the size of the dataset and the Tree Width.

In the diagram below, the width of the tree is two and the height is 3. The dataset consists of 4 accounts:


![Image1](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/1_4leafTree.png)



The standard implementation of Silver Sixpence Merkle Tree Generator, however, uses a width of 4, because some exchanges could potentially have millions of accounts which would result in a tree with more than 20 levels. If the width is increased to 4, then the height of the tree reduces by around 50%. The following diagram shows a tree with 16 leaves and width 4, producing a height of 3:

![Image2](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/2_Width4Tree.png)

Parent_node = SHA256(concat(child1, child2, child3, child4))

If we display the same dataset next to each other, one could visualize the reduction in height if the width is increased:

![Image3](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/3_WidthTreeCompare.png)

The reason for choosing a tree width of 4 is simply to provide a better user experience when displaying the Merkle Proofs. A lower tree height displays better on a smaller screen size.

If the number of leaves or nodes on a particular height is not a multiple of the tree width (i.e. #leaves(mod(width)) != 0), then the last node is duplicated and the resulting nodes concatenated so that there are always 4 children nodes to every parent node.
The tree below was built from a dataset with 13 leaf nodes, so the last node had to be duplicated 3 times so that it can be divided by 4:


![Image4](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/4_NodeDuplicate.png)


We produce the last parent node on Layer 1 as follows:

Parent_node = SHA256256(concat(child1, child2, child2, child2))

We do single SHA256 hashes all the child nodes to produce parents.

We will create a number of dummy accounts to be included in the leaf nodes. We typically generate at least 10% dummy accounts by spawning a random customer ID data with a 0-balance. This is done to add additional privacy to the data.

Consider a dataset of 40 accounts. 10% (four) dummy accounts are generated with a zero asset balance and inserted randomly between all the existing accounts. The leaf nodes are then generated by hashing the concatenated string of account data. In the image below, these leaf nodes form the bottom layer of the tree. 
Each node is then grouped with 3 neighboring nodes, concatenated and hashed to build the next layer. This step is repeated until only a single node remains, the Merkle Root.


![Image5](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/5_NodeDummyAccounts.png)

A user could verify that his/her account data was included in the total dataset that was included in the tree, by independently reproducing the Root from its account data.


# User Verification

Any exchange client could verify that his/her account data was included in the total dataset that was included in the tree, by independently reproducing the Root from its account data.

Let’s say Amoné has an account at fictitious Exchange *SixpenceCoin*. She logs into her account and notice that an independent Proof-of-Reserves audit has recently been done, and that *SixpenceCoin* has enough assets to cover all of their liabilities. Amoné wants to know if her Bitcoin balance was indeed included when the total liabilities were calculated. 

On her profile page, she has a tab that includes data from the audit. Amoné’s balances at the time of the audit (called the snapshot) is displayed, along with her unique *customer ID*, an identifier unique to the particular audit, a random number called *Salt* and a *Merkle Root*. Amoné decides to independently verify the auditor’s results. 
She goes through the following steps:

**Step 1: Create the Merkle Leaf**

Amoné first needs to build her Merkle Leaf, which is a hash her *account ID*, *Salt*, *Audit ID* and *Balances*. Each exchange or auditor may have their own preferred structure, but *SixpenceCoin* uses the following format:

*Customer_ID*: A hash of the client’s username

*Salt*: Pseudorandom hexadecimal string, trimmed to 16 characters

*Audit_ID*: POR + Month of Audit + Year of Audit

*Client_Balances*: (Asset1=Balance | Asset2=Balance | Asset3=Balance ...)

Example:
```
Customer_ID = “287e29b8-de5b-4924-8906-b216f2d48cd6”
Audit_ID = "PORNOV22"
Salt = "2A496ECE"
Client_Balances = "BTC=6.83|ETH=1.26|XRP=88.95|USDC=2.32"
```


Amoné sees that the audit snapshot was taken at 2022/11/24 23:59 and verifies that her balances at that time was indeed as indicated on her profile page. She then concatenates all of this data together to form her Account Record:

Account_Record = concat(Account_ID + Salt + Audit_ID + Client Balances)

```
Account_Record = "287e29b8-de5b-4924-8906-b216f2d48cd62A496ECE PORNOV22BTC=76.83|ETH=19.26|XRP=68.95|USDC=6.32"
```

Amoné copies the Account_Record string and heads over to  [https://silversixpence.co.za/](https://silversixpence.co.za/) where she pastes the string into the form and verifies the hash that SixpenceCoin displays in her profile. 

She then directs to the Auditors’ website [https://www.mazars.co.za/] (https://www.mazars.co.za/) and navigates to the *SixpenceCoin* November 2022 audit page.

Here, she is presented with a  field from which she can search for her Merkle Leaf hash. The web application builds her unique tree path with all the hashes to the Root. Amoné is now ready to do a Merkle-Proof.

**Step 2: Build the first Branch**

*SixpenceCoin* has 20 million clients, so a Merkle Tree that hashes 2 nodes together would be 25 levels high. The auditor for *SixpenceCoin* decided to increase the Merkle Tree width to 4, which means that 4 child nodes are each concatenated to produce the parent node. This reduces the tree height to 13 levels, which displays better on smaller screens and requires less code to validate.

Amoné already has one of the four leaves required to build the first branch – the one that he produced herself. She therefore needs hashes from three of her neighboring accounts. These hashes, together with their sequence, are provided by the auditor on their web-application. Amoné concatenates the four leaf nodes in their correct order and hashes the result to produce the first branch.

![Image7](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/7_ChildParentNodes.png)


**Step 3: Repeat**

Amoné now needs to repeat the process of concatenating nodes to build the next branch. The auditor will provide 3 neighboring children nodes for every level, which Amoné needs to concatenate to produce the parent node. This step is repeated until only a single node remains, which is called the Merkle Root. If she successfully managed to reproduce the public Merkle Root, then she can be certain that her asset balances were included in the total liabilities that were observed by the auditor.
The complete tree structure will look like the image below. Note that the auditor will only reveal the minimum amount of information for Amoné to do independent verification, which is the nodes highlighted in green below. The rest of the nodes are kept private, as they are not needed for full verification.


![Image8](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/8_UserVerifyTree.png)

The auditor website displays Amoné’s proof as a tower of vertical hashes. This is simply done to provide a better user experience. Since the auditor stores the client balances, they might display that to Amoné as well, if the exchange gave them the necessary permission.


![Image9](https://github.com/silversixpence-crypto/merkletree-verify/blob/main/images/9_SSfrontend.png)


# FAQ

## What is a proof-of-reserves audit

With cryptocurrency exchanges the saying goes: "Not your keys, not your coins". Proof of reserves provides an indication of whether customer funds are safe, meaning that sufficient funds are backing all deposits. If all clients decided to withdraw all their crypto holdings the exchange will have enough liquidity to process everyone's withdrawal requests.

## What is a proof-of-liabilities audit

Proof of reserves can give you an indication of the amount of assets under the exchanges' control. But if the exchange holds 10 bitcoin while clients deposited 20 bitcoin, then their liabilities exceed their assets. We therefore have to determine how much bitcoin is owed to users and compare the amount to the reserves in order to determine the health of the exchange. The best process to determine total liabilities, is to have a trusted, external auditor view all client balances. The auditor should then construct a Merkle Tree by including all of the client balances into the base layer (leaves) of the tree. By doing this, it enables every client of the exchange to independently and mathematically verify that his account balances were included in the audit.

# What is a hash

Calculating the hash of some input data prodoces a pseudorandom string of characters that are unique to that input data. If you change any character of the input data the resulting hash will be completely different.

For our application, we use SHA256 (Secure Hashing Algorithm). It takes input data of any length and produces a hash of length 256 bits which can be presented as 64 hexadecimal characters. The number of possible combinations in 256 bits are in the order of 10^77, which is a mere 1000 times less than the amount of atoms in the visible universe (10^80). To produce a colission in SHA256 (2 different inputs producing the same SHA256 output) you would need to conduct about 2^63 operations, which means that the algorithm should be secure for at least another 100 years.

We use a double SHA256 (SHA256d) hash to build the Merkle Trees. This is to add another layer of security.

# What is a merkle tree

In cryptography and computer science, a hash tree or Merkle tree is a tree in which every "leaf" (node) is labelled with the cryptographic hash of a data block, and every node that is not a leaf (called a branch, inner node, or inode) is labelled with the cryptographic hash of the labels of its child nodes. A hash tree allows efficient and secure verification of the contents of a large data structure. https://en.wikipedia.org/wiki/Merkle_tree

Our merkle trees has a width of four for improves user experience...

# Why do we generate a merkle tree as part of a proof-of-reserver audit

To prove to a user that his/her exchange balance is included in the audit...

# How do we generate the merkle tree

## Leaf

We use your data + SHA256 to generate a leaf that is unique to you. The leaves are level-0 of the merkle tree.

## Nodes

The first level of nodes we take four leaves, concatenate them and take the SHA256d of the resulting string. This will be a node on level-1. After all the leaves are hashed together to produce the level-1 nodes, we hash the level-1 nodes together four-by-four to produce the level-2 nodes. And so on until only a single root node is left.

## Balances or not?

Security vs show in webpage

## Uniqueness

Use salt or audit id





