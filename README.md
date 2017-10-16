# Abstract 
This document represents jury.online protocol. Current version shows interfaces and code of the naive implementation
with simplified logic and few security measures used.

Purpose of this document is to show the general structure of jury.online platform to the public, to raise questions and
open discussion by community. Current version should be considered as a prototype and shouldn't be used for real
operation.

More information is available at [jury.online](https://jury.online).

# Specification

Complete operation is done not only via Ethereum smart contracts, protocol specification also involves requirements for
identities representation (cryptographic keys derivation and linking), data storage and encryption. This document will
be added with the details after they are finalized.

# Contracts

This section describes functions and data needed to be stored in smart contracts.

Our implementation is still to come in the nearest future.

## Deal

Deal is the main contract which stores the most of data and logic. Initiator of the deal provides needed information and
deploys contract into the blockchain.  Counterparties must provide their agreement on the terms of deal in order to
start the execution.

### Addresses

First counterparty.
```solidity
address sender;
```

Second counterparty. This party can accept the deal. If the value is zero, then anyone can become the second
counterparty.
```solidity
address receiver;
```

Pool provides jurors for the deal if dispute resolution is needed.
```solidity
address pool;
```

Address of pseudo-random number generator used for jurors selection.
```solidity
address prng;
```

Address of rating contract.
```solidity
address rater;
```

### Timestamps
All time intervals and timestamps are given in seconds.

Indicates the last moment when this deal can be accepted by the other party.
```solidity
uint acceptTime;
```

Indicates the last moment for successful (without dispute) deal execution.

```solidity
uint executionTime;
```

Indicates time interval for arguments and comments providing. 
```solidity
uint commentDuration;
```

Time interval for judge to provide the verdict.
```solidity
uint verdictDuration;
```

### Statuses

```solidity
bool accepted;
```

Function called by the receiver of the deal to indicate his agreement on deal terms and start of execution (if all other
requirements are met).
```solidity
function accept();
```

### Dispute resolution type
Dispute resolution type specifies the form of resolution: is it done by a given number of randomly selected jurors from
a single pool; sole verdict of some unbiased expert; combination of several variants. For this prototype we consider the
first case.

Number jurors needed for resolution.
```solidity
uint jurorsCount;
```

### Other
Every parameter of the deal can be changed by the initiator of the deal if the deal is still not accepted. So each
parameter needs a function to change it.

Deal subject and description. For the prototype we consider it as a string with complete information, it can also be a link to some
online document or ipfs identificator of some data.
```solidity
string description;
```

Parties' collaterals in wei. Depositing agreed amounts is necessary to start the deal.
```solidity
uint senderAmount;
uint receiverAmount;
```

Parties' contributions to potential dispute resolution price. For example a single party may pay the full price, or the
price is divided equally, or any other ratio.
```solidity
uint senderJOTRatio;
uint receiverJOTRatio;
```

Revoke the deal if it's still not accepted.
```solidity
function revokeDeal();
```

Information about appeals, for the prototype it's just a number of possible appeals.
```solidity
uint appealCount;
```

Function `openDispute` can be called by either of the counterparties and indicates their dissatisfaction in deal
execution.

```solidity
function openDispute();
```

In order to start a dispute resolution process, parties must provide payment for the resolution. It's nominated in JOT.
Price estimate is available beforehand, at the moment of creation of the deal, but its real value depends on current
availabilities of jurors and may slightly differ from the estimate.

After the dispute is opened parties have `commentDuration` time interval to provide their comments and arguments for
correct dispute resolution. It can be just a string with text comments, but as well it can hold a link to some online
document.
```solidity
string senderComments;
string receiverComments;
```
After the comment period is over, process of selecting jurors is started.

For random judge selection both parties must generate a part of the seed used for random number generation. In the simplest
case seed can be thought as an integer in interval `0..2^256-1`. For secure
operation party generates a seed, stores it in secret and posts hash of the seed.
```solidity
bytes32 senderHash;
bytes32 receiverHash;
```

After both parties provided hashes of their seeds they can no longer change the seed. Now parties disclose their seeds.
`discloseSeed` checks if the provided value corresponds to the previously sent hash. If the hash of the disclosed seed
is different from the sent hash value, then sending party is trying to cheat. If both parties are trying to cheat, the process of
obtaining of common seed is repeated from the start.

```solidity
function discloseSeed(uint _seed);
```

Actual seeds of the counterparties.
```solidity
uint senderSeed;
uint receiverSeed;
```

After the second seed is known, the common seed is calculated.
```solidity
uint commonSeed;
```

Jurors comments and reasons for the verdict.
```solidity
[]string jurorsComments;
```

## Pool 
From the naive point of view pool stores a list of jurors' addresses which is later used to select judges for a deal.

Pool information can be updated only by its administrators.
```solidity
mapping (address => bool) administrators;
```

Active jurors addresses.
```solidity
uint poolLength;
[]address jurors;
```

List update, which is called periodically.
```solidity
function update([]address _jurors);
```

## Prng

Pseudo-random number generator contract is needed for fair random choice of jurors. It needn't to be cryptographically
secure, but it must output sufficiently uniform data.

Pseudo-random generation of `amount` elements out of `1..range` interval with no duplicates. Returns list of generated
numbers.
```solidity
function generate(uint seed, uint amount, uint range) returns ([]uint);
```

## Rating 

Rating smart contract is a naive approach to organize statistics and rating for jurors and counterparties. 

Rating contract reads information from the deal and takes all the needed data from it. This function is called whenever
a deal is finished with any outcome.
```solidity
function updateRating();
```
