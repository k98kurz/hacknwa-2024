# Blockchain Technologies

## Introduction

Three sections:

1. Blockchain primitives and some history
2. Layer 2 technologies
3. Advanced cryptography

## What is a blockchain?

Prepend-only linked list with cryptographic verification features that is read
in reverse by convention and maintained by a replicated state machine governed
by a consensus mechanism.

---

# Blockchain Primitives

## Blitzkrieg Explication

Three + 1 kinds of primitives:

1. Cryptography
2. Data structure
3. Consensus
4. Virtual machine

---

# Cryptography

Two cryptographic primitives used in blockchains:

1. Cryptographic hash functions
2. Digital signature algorithms

---

# Cryptographic Hash Functions

A function _`h`_ that takes a variable length input and produces a fixed length
output. It must have the following qualities:

1. Deterministic computation: given an input _`I`_, the output _`O = h(I)`_ will be the same
every time it is calculated.
2. Collision resistance: given inputs _`I1`_ and _`I2`_ and outputs _`O1 = h(I1)`_ and
_`O2 = h(I2)`_, the likelihood that _`O1 == O2`_ is astronomically low as long as _`I1 != I2`_.
E.g. in SHA256 with 2^256 possible outputs, that likelihood is approximately 1/2^256.
(2^256 ≈ 10^77; there are estimated to be about 10^80 atoms in the universe, or about 1000 atoms
per SHA256 output)
3. Irreversible: given an output _`O = h(I)`_, it is impossible to calculate the input _`I`_.
4. Unpredictable: given an input _`I`_ and output _`O = h(I)`_, it is impossible to predict how
_`O`_ will change after some change to _`I`_.

# Presenter Notes

10^80 is called the Eddington number.

---

# Example using SHA256 bitstrings:

Inputs:

- 00000000
- 00000001

Outputs and their XOR:

- 110111000110100000010111001110011111111101100110111101010011000100111001010...
- 100101111110101000100100010111100110100010001010101010011000101001110111101...
- 010010111000001000110011011001111001011111101100010111001011101101001110111...

Changing 1 input bit changes approximately half of the output bits.

---

# Digital Signature Algorithms

A digital signature algorithm can be described in four functions:

1. Private key generator: `prvkey = f1(seed)`
2. Public key generator/deriver: `pubkey = f2(prvkey)`
3. Signer of arbitrary data using a private key: `sig = f3(data, prvkey)`
4. Verifier of signatures using a public key: `valid = f4(sig, data, pubkey)`,
which returns false if the signature is not valid for the data and the public key.

# Presenter Notes

A system for creating cryptographic proofs using a private key that can be
verified by anyone with the corresponding public key. Can be used for authenticaiton,
authorization, or attestation.

---

# Digital Signature Algorithms

Probabilistic security guarantees:

1. Authentication: ensure the signer of a block of data is who they claim to be.
2. Integrity: ensure the content of the block of data has not been altered since
it was signed.
3. Non-repudiation: prevent the signer from denying that they signed the data
block, providing evidence of their attestation of or interaction with the data.

# Presenter Notes

Modern digital signature algorithms provide the following probabilistic security
guarantees. Elliptic curve DSA and Schnorr are most common, but lattice-based
post-quantum DSAs are being finalized by NIST (Signal protocol already adopted
a PQ latice system for DHE and DSA).

---

# Data Structures

Both fundamental blockchain data structures are directed acyclic graphs (DAGs):

1. Linked list
2. Merkle tree

---

# Linked List

A linked list is a sequence of items that each contain a reference (link) to the
next item in the sequence.

<pre>
item0 = {
    data0,
    link_to_item1
}

item1 = {
    data1,
    link_to_item2
}
</pre>

---

# Linked List

In effect, a linked list is a directed graph where each node has in-degree and
out-degree each of either 0 or 1.

[node] --> [node] --> [node]

In a blockchain, the order is reversed by convention:

[block] <-- [block] <-- [block]

Importantly, the way that edges in this graph are encoded (hash function)
makes the formation of a cycle impossible. Hence, a blockchain is a directed
acyclic graph with maximum in-degree and out-degree of 1 for each node.

---

# Merkle Tree

<pre>
[leaf0] [leaf1] [leaf2] [leaf3]
     \     /      \     /
     [node0]      [node1]
           \      /
            [root]

node0 = hash(hash(leaf0) || hash(leaf1))
node1 = hash(hash(leaf2) || hash(leaf3))
root = hash(node0 || node1)
</pre>

A Merkle tree is a tree structure that uuses a cryptographic hash function to
combine a number of items, called "leaves", in pairs to form nodes, then
recursively combines nodes in pairs until it arrives at a single value, called
the root.

# Presenter Notes

- Diagrams are normally upside down with the root at the top and the leaves
at the bottom, but that's not how trees work.
- A Merkle tree is also a directed acyclic graph: the root links to the nodes
above/before it, and those nodes link to prior nodes or leaves. The way these
links are encoded and aggregated (cryptographic hash function) ensures that
cycles are impossible.

---

# Merkle Tree

<pre>
[leaf0] [leaf1] [leaf2] [leaf3]
     \     /      \     /
     [node0]      [node1]
           \      /
            [root]

node0 = hash(hash(leaf0) || hash(leaf1))
node1 = hash(hash(leaf2) || hash(leaf3))
root = hash(node0 || node1)
</pre>

## Proof for leaf1

<pre>
bool verify(root, leaf, proof) {
    let sum <- hash(leaf)
    for step in proof {
        if step is right {
            sum <- hash(sum || step)
        } else {
            sum <- hash(step || sum)
        }
    }
    return sum == root
}

proof = [{hash(leaf0): left}, {node1: right}]
assert verify(root, leaf1, proof)
</pre>

# Presenter Notes

- A Merkle tree root commits to a data set of arbitrary size.
- Using a Merkle tree, a data set inclusion proof can be generated with size
that scales with the logarithm of the number of set members.

---

# Merkle Tree (Taproot Edition)

<pre>
[leaf0] [leaf1] [leaf2] [leaf3]
     \     /      \     /
     [node0]      [node1]
           \      /
            [root]

node0 = xor(hash(hash(leaf0)), hash(hash(leaf1)))
node1 = xor(hash(hash(leaf2)), hash(hash(leaf3)))
root = xor(hash(node0), hash(node1))
</pre>

## Proof for leaf1

<pre>
bool verify(root, leaf, proof) {
    let sum <- leaf
    for step in proof {
        sum <- xor(hash(sum), hash(step))
    }
    return sum == root
}

proof = [leaf0, node1]
assert verify(root, leaf1, proof)
</pre>

# Presenter Notes

- With the Taproot upgrade, the Bitcoin core dev team introduced a new, more
flexible version of the Merkle tree that removes handedness in proofs.

---

# Merkle Tree (theoretically faulty version)

<pre>
[leaf0] [leaf1] [leaf2] [leaf3]
     \     /      \     /
     [node0]      [node1]
           \      /
            [root]

node0 = xor(hash(leaf0), hash(leaf1))
node1 = xor(hash(leaf2)), hash(leaf3))
root = xor(node0, node1)
</pre>

## Proof algorithm

<pre>
bool verify(root, leaf, proof) {
    let sum <- hash(leaf)
    for step in proof {
        sum <- xor(sum, step)
    }
    return sum == root
}
</pre>


---

# Merkle Tree (theoretically faulty version)

<pre>
[leaf0] [leaf1] [leaf2] [leaf3]
     \     /      \     /
     [node0]      [node1]
           \      /
            [root]

node0 = xor(hash(leaf0), hash(leaf1))
node1 = xor(hash(leaf2)), hash(leaf3))
root = xor(node0, node1)
</pre>

## Proof algorithm and hack

<pre>
bool verify(root, leaf, proof) {
    let sum <- hash(leaf)
    for step in proof {
        sum <- xor(sum, step)
    }
    return sum == root
}

spoof = xor(node1, hash(newleaf))
assert verify(root, newleaf, [spoof, node1])
</pre>

# Presenter Notes

- By including the intermediate hashing step, now the attacker must find
a preimage for the spoofed node derived from the xor operation, which is
computationally infeasible.

---

# Consensus Mechanisms

Two types:

- Trusted: RAFT, Paxos, proof-of-authority, etc
- Trustless: proof-of-work
- Depends: proof-of-stake

## Proof-of-work

- Probabilistic settlement mechanism: accumulation of reinforcing proofs-of-work
reduces likelihood of successful rewrite
- Hashcash: difficulty defined as preceding null bits in a hash function output
- Invented by Adam Beck to fix email spam, then used by Satoshi in Bitcoin for
consensus

# Presenter Notes

- Proof-of-stake: trusted if initial minting and distribution were manual;
trustless if initial minting and distribution were proof-of-work

---

# History Detour: Bitcoin Whitepaper

[bitcoin.pdf](bitcoin.pdf)

---


# Virtual Machine

Bitcoin script system: implementation detail omitted from whitepaper. All
digital signatures are supplied in the form of unlocking scripts (witness data)
that evaluates with the locking script to `true`.

Exapmle locking script:

```
OP_DUP OP_HASH160 <pubkey hash> OP_EQUALVERIFY OP_CHECKSIG
```

Example unlocking script:

```
<signature> <pubkey>
```

# Presenter Notes

- Pay-to-pubkey-hash example
- Stack machine: unlocking script executes first
- Bitcoin script has changed significantly over the years, primarily through
soft-forks (templates) and node policies (disabling or redefining ops)

---

# Virtual Machine

[Not always easy to get correct](https://rekt.news)

[Not even easy to experiment with](https://pypi.org/project/tapescript/)

---

# Layer 2 (aka Off-Chain Solutions)

Blockchains are inefficient databases and trade throughput for decentralization
(or rather decentralization theater/hype in most cases). Scaling throughput is
attempted via layer-2/off-chain systems that use on-chain transactions for
initialization and settlement.

---

# Payment Channels





