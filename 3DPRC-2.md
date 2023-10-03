# 3DPRC-2 
The object tokenization standard 

Authors: PaulS, Michael Co, Mikhail

## What is 3DPRC-2?

3DPRC-2 (3Dpass Request for Comments), proposed by PaulS in September 2023, is a standard p2p protocol for the tokenization of objects operating within “The Ledger of Things” decentralized blockchain platform. 

Every tokenized object acquires its unique on-chain identity HASH ID. By means of utilization of recognition algorithms implemented, all the assets approved by the network, will be protected from being copied to the extent of the error of the algorithm precision.

### 3DPRC-2 is implemented as the following components:

**1. Advanced version of “Proof of Scan”**

The protocol is weaved into “The Ledger of Things” PoW component in a way to tackle the user objects authentication along with the ones being mined. The protocol ensures for users to get a complete service always resulting as either the object acceptance (the asset is allowed to be created) or its rejection (copy is found on the db). The network is responsible for the user object authentication as much as for any block on the blockchain irrespective to the actual dollar value attached to; 

**2. “0 knowledge proof”**

Every judgement provided by miners about the object authenticity is protected by a secret knowledge of its HASH ID** being unavailable for them, until they get the object processed. Every proof is being verified by the majority of the network to make a final decision on whether to accept or reject the block containing the judgement;

**3. [PoScan Substrate-based pallet](https://github.com/3Dpass/3DP/tree/test/pallets/poscan) (storage and [API](https://github.com/3Dpass/3DP/wiki/3DPRC%E2%80%902-PoScan-API))**

The PoScan pallet is integrated into the network runtime providing the access to the network decentralized storage by means of the object tokenization [API](https://github.com/3Dpass/3DP/wiki/3DPRC%E2%80%902-PoScan-API), which allows for:
- the user object authentication and its protection from being copied to the extent for the recognition algorithm precision;
- non-fungible digital asset creation;
- property rights definition and its transfers;
- backed cryptocurrency issuance (fungible tokens backed by the asset)

## The Object categories and recognition algorithms
This is apparent, that it does not make any sense to compare objects by its HASH ID, provided they got processed with different recognition algorithms/parameters. However, HASH IDs need to be compared in order to guarantee for users the absence of copies on the blockchain data base.

By means of categorization of the object types, we are setting up some “standard” algorithms (presets) to be available for use within each category. And every preset defines the level of precision, at which the object is going to be recognized. 

Initial list of categories is presented as follows: 
- 3D objects
  - Algo: `grid2d_v3a -s 12 -g 8` (learn more about [grid2d](https://github.com/3Dpass/pass3d))
- 2D drawings
- Music
- Biometrics
- Radio signals
- Movements
- Texts

The categories and presets are being moderated by the Asset committee** vote. The committee members are being assigned by the Council**.

## The Object authentication protocol
The protocol represents a sequence of actions performing by validators and miners actively participating in the network consensus at the time an object is being submitted by user. 

**1. Submitting an object**

While placing an order, 3dpass user provides the following:
- an object to tokenize: `rock.obj` (in this example we are going to take 3D model in .obj** format)
- the object HASH ID (`top10` hashes from the [Grid2d](https://github.com/3Dpass/pass3d)** recognition algorithm output; preset: `-a grid2d_v3a -s 12 -g 8 -d 10`; [pass3d](https://github.com/3Dpass/pass3d)** recognition toolkit is being used as the implementation of Grid2d):
```
admin@admin pass3d % ./target/release/pass3d -s 12 -g 8 -a grid2d_v3a -d 10 -i rock.obj
"7fbe267b208d0ab9c34fa184d4593d0ae39d5bb56852ecc633cb1f6d9eb5aae0"
"460c81f58510e211d162d458459f81575e630817c1b87bc3c0d2c3eec3ae68b7"
"8047c7a4d2a75ce466c9510edb37ac1e98826ef383ab5d4c860b5010e42faa6e"
"5001723ef6d85862733d25765f0700e381cc3a1f6d6b7dbb4efb8aeccc4615d4"
"b8efd617a07099edcdbf2596d10787a6e16a4c59a5e36e46adedc1dd2424b8c9"
"10c8d4984cf840845a7d7785871c70790bf1c55e11a3da54a41d3f21be5a7ea7"
"833a2fc1f26ab9eb9bcc5b4e668f5d6bb91d6f87fb0798f99ce6bb4730dfc218"
"abdedd28e03147d94e0d054d0bf24781bac01fdd81401cd3fb226b5a6e1a5371"
"97c538a99641202385572df245d9cfb300f895a6c94e7caf074023a5ead1c62c"
“db2453f53270c9003a63b4d643d9bcc991ad7630d70a0d1511781f74a7a00fee"
```
- Authentication fee P3D/Byte: Let’s take 100 P3D as an example. This fee will be distributed among the miners and validators taking part in the process (70 miners /30 validators). The fee will be charged for each confirmation ordered (1 confirmation = 1 block). The fee can be set up by Council** vote.
- Storage fee: P3D/Byte (regular network storage fee)
- Number of confirmations: let’s take 6, for example. The more confirmations the user orders, the more reliable result He gets, especially, when it comes to the network potentially being attacked at the time. This is unlikely to happen, however, there is always a possibility for every blockchain network to get through this sort of experience. It is expected that the user is able to follow the gap between Best block and the Block finalized (normally the gap = 2 blocks) to estimate the network state, before submitting the order.

There is a simple copy check on the object submit extrinsic (transaction). If there is a copycat on the blockchain discovered, the transaction will fail (learn more about [pass3d](https://github.com/3Dpass/pass3d) output to know how objects are supposed to be compared by its HASH IDs)

**2. Estimation/Anti-SPAM protection**

Once having the object submitted and fees paid, Validators (the most reliable nodes of 3Dpass network) from the current GRANDPA validator set start estimating how long would it take for the object to get processed by the network in average. They will try to process the object and, if successful, provide the time in milliseconds at which they managed to get it done like this:
```
Validator 1 - 128
Validator 2 - 36 
Validator 3 - 32 …
```
There is a limit of 3 blocks for the estimation phase to complete.

The validators will have to import the block containing the object and get it processed once again at the next step when miners have added their judgements. Assuming the block target time in 3dpass set up as 60 sec/block, there is a time frame limit of 10 seconds every validator must have finished processing within. 

In disregards to the reason why a given validator didn't manage the estimation in time (10 sec), its vote won't be taken. On top of that, every “weird” estimation result (vote) will be statistically ruled out and the average processing time - calculated.

For, example:
```
Validator 1 - 1128 - (ruled out)
Validator 2 - 36 +
Validator 3 - 32 +
Validator 4 - 0.1- (ruled out) 
Validator 5 - 24 + 
Validator 6 - 18 + … 
```
The threshold for the object to pass is established at “2/3 +1” out of the actual number of validators in the set. This is the same threshold ratio as the one being utilized for the GRANDPA finalization. The main requirement for this procedure to work correctly would be having “2/3+1” validators providing true data on the object processing. 

30% of the fee paid by the user will distributed among the validators-estimators in equal. Implying, there is about 400 validators in the set, 30 P3D/400= 0.075 P3D each validator gets. 

It is assumed, that most of the actual validators are motivated enough to keep the network healthy, otherwise the network as a whole would be considered unsafe. Although, the validators are not providing any substantial proof of their work, which could not be taken for 100% truth by miners taking its turn at the next step. 

This is a snapshot example of the object `Estimated` from the storage (short version, the data is presented partially. See more [POSCAN API](https://github.com/3Dpass/3DP/wiki/3DPRC%E2%80%902-PoScan-API)):

```
[
    [
      4
    ]
    {
      state: {
        Estimated: 2,693
      }
      obj: 0x6f200a7620302e313520302e383520302e320a7620302e313520302e383620....
      category: {
        Objects3D: Grid2dLow
      }
      hashes: [
        0x7fbe267b208d0ab9c34fa184d4593d0ae39d5bb56852ecc633cb1f6d9eb5aae0
        0x460c81f58510e211d162d458459f81575e630817c1b87bc3c0d2c3eec3ae68b7
        0x8047c7a4d2a75ce466c9510edb37ac1e98826ef383ab5d4c860b5010e42faa6e
        0x5001723ef6d85862733d25765f0700e381cc3a1f6d6b7dbb4efb8aeccc4615d4
        0xb8efd617a07099edcdbf2596d10787a6e16a4c59a5e36e46adedc1dd2424b8c9
        0x10c8d4984cf840845a7d7785871c70790bf1c55e11a3da54a41d3f21be5a7ea7
        0x833a2fc1f26ab9eb9bcc5b4e668f5d6bb91d6f87fb0798f99ce6bb4730dfc218
        0xabdedd28e03147d94e0d054d0bf24781bac01fdd81401cd3fb226b5a6e1a5371
        0x97c538a99641202385572df245d9cfb300f895a6c94e7caf074023a5ead1c62c
        0xdb2453f53270c9003a63b4d643d9bcc991ad7630d70a0d1511781f74a7a00fee
      ]
      whenCreated: 2,690
      whenApproved:
      owner: d1f5KGsoZ3xzB6Ecmv92DPizD1x5eToNHM1CPfSC1Xu4nzecN
      estimators: [
        [
          d1ePg4fK97U913xAnZzZ9dUf1W1XUA4bYVC2Zre8K9vjSixnc
          36
        ]
        [
          d1asoD7V6hdff4ExvtjRbdVT398Rg8fKEruYsKFS5P9mkT4fy
          32
        ]
      ]
      estOutliers: []
      approvers: []

      numApprovals: 6
      estRewards: 70,000,000,000,000,000
      authorRewards: 30,000,000,000,000,000
    }
  ]
]
```
The object becomes `NotApproved`, if it doesn’t pass the estimation within 3 blocks timeframe limit. 

**3. Gathering confirmations/approvals**

Once having the order status at `Estimated`, the actual block author, who found new block, is getting privileged to provide the judgement “Approved” about the object. There is no option for miners to provide a negative judgement available.

Any honest block author is expected to do the processing job on the user object. The proof of work is required at this step to be attached to the block. If the proof turns out to be incorrect or fake, the entire block will be rejected by the majority of the network. 

There is an additional copy check on the block import for every full 3dpass node. If there is a copycat of the user object on the blockchain discovered, the block will be rejected, either.
In order to get a proof the block author should get the object processed with the same preset as the user actually have, with the only exception for the `top11` hashes are to be claimed from pass3d: 

```
admin@admin pass3d % ./target/release/pass3d -s 12 -g 8 -a grid2d_v3a -d 11 -i rock.obj
"7fbe267b208d0ab9c34fa184d4593d0ae39d5bb56852ecc633cb1f6d9eb5aae0"
"460c81f58510e211d162d458459f81575e630817c1b87bc3c0d2c3eec3ae68b7"
"8047c7a4d2a75ce466c9510edb37ac1e98826ef383ab5d4c860b5010e42faa6e"
"5001723ef6d85862733d25765f0700e381cc3a1f6d6b7dbb4efb8aeccc4615d4"
"b8efd617a07099edcdbf2596d10787a6e16a4c59a5e36e46adedc1dd2424b8c9"
"10c8d4984cf840845a7d7785871c70790bf1c55e11a3da54a41d3f21be5a7ea7"
"833a2fc1f26ab9eb9bcc5b4e668f5d6bb91d6f87fb0798f99ce6bb4730dfc218"
"abdedd28e03147d94e0d054d0bf24781bac01fdd81401cd3fb226b5a6e1a5371"
"97c538a99641202385572df245d9cfb300f895a6c94e7caf074023a5ead1c62c"
"db2453f53270c9003a63b4d643d9bcc991ad7630d70a0d1511781f74a7a00fee"
"ab198306dcee347bd84a942ffc9ab06c8458865fee03c5d1b20ab45b807dc1c5" - the 11th hash (proof of the first miner work)
```
Miner knows nothing about this `11th hash`, until having the object processed. This secret, called “0 knowledge proof”, will be leveraged by the network majority while importing the block. 

It is totally up to the block author whether or not to trust this collective estimation result coming from validator set. So, there is always a risk for him to loose his block rewards. And there is a benefit to get rewarded on top by user, if the judgement is accepted. 

It is assumed, that every miner will rely on himself and make an independent decision about both aspects the object authenticity and the processing time. If any of those fail, the judgement should never be provided. 

If the block author decides to skip, the block will not contain the judgement, meaning, there is no confirmation will be gained for the object. 

If the block was accepted by the network majority, then “+1” confirmation the object gets. Now the next block author takes its turn, and the proof of work will be the `12th hash` from the grid2d output: 

```
admin@admin pass3d % ./target/release/pass3d -s 12 -g 8 -a grid2d_v3a -d 12 -i rock.obj
"7fbe267b208d0ab9c34fa184d4593d0ae39d5bb56852ecc633cb1f6d9eb5aae0"
"460c81f58510e211d162d458459f81575e630817c1b87bc3c0d2c3eec3ae68b7"
"8047c7a4d2a75ce466c9510edb37ac1e98826ef383ab5d4c860b5010e42faa6e"
"5001723ef6d85862733d25765f0700e381cc3a1f6d6b7dbb4efb8aeccc4615d4"
"b8efd617a07099edcdbf2596d10787a6e16a4c59a5e36e46adedc1dd2424b8c9"
"10c8d4984cf840845a7d7785871c70790bf1c55e11a3da54a41d3f21be5a7ea7"
"833a2fc1f26ab9eb9bcc5b4e668f5d6bb91d6f87fb0798f99ce6bb4730dfc218"
"abdedd28e03147d94e0d054d0bf24781bac01fdd81401cd3fb226b5a6e1a5371"
"97c538a99641202385572df245d9cfb300f895a6c94e7caf074023a5ead1c62c"
"db2453f53270c9003a63b4d643d9bcc991ad7630d70a0d1511781f74a7a00fee"
"ab198306dcee347bd84a942ffc9ab06c8458865fee03c5d1b20ab45b807dc1c5"
"82fd9527291ba30cdddeb5786bac083f8092987e379c9433d81a2eb1bb8c447a" - the 12th hash (proof of the second miner work)
```
Again, the miner knows nothing about this 12th hash, until having the object processed. If there is no hash below the top11 available, `null` will be taken for proof.

This procedure repeats itself, up until the block at which required number of confirmations is reached. In this particular example the number is 6, so the final 6th judgement will contain the HASH ID expanded to `top16` hashes: 
```
admin@admin pass3d % ./target/release/pass3d -s 12 -g 8 -a grid2d_v3a -d 16 -i miner/rock.obj
"7fbe267b208d0ab9c34fa184d4593d0ae39d5bb56852ecc633cb1f6d9eb5aae0"
"460c81f58510e211d162d458459f81575e630817c1b87bc3c0d2c3eec3ae68b7"
"8047c7a4d2a75ce466c9510edb37ac1e98826ef383ab5d4c860b5010e42faa6e"
"5001723ef6d85862733d25765f0700e381cc3a1f6d6b7dbb4efb8aeccc4615d4"
"b8efd617a07099edcdbf2596d10787a6e16a4c59a5e36e46adedc1dd2424b8c9"
"10c8d4984cf840845a7d7785871c70790bf1c55e11a3da54a41d3f21be5a7ea7"
"833a2fc1f26ab9eb9bcc5b4e668f5d6bb91d6f87fb0798f99ce6bb4730dfc218"
"abdedd28e03147d94e0d054d0bf24781bac01fdd81401cd3fb226b5a6e1a5371"
"97c538a99641202385572df245d9cfb300f895a6c94e7caf074023a5ead1c62c"
"db2453f53270c9003a63b4d643d9bcc991ad7630d70a0d1511781f74a7a00fee"
"ab198306dcee347bd84a942ffc9ab06c8458865fee03c5d1b20ab45b807dc1c5" - 11th hash (proof of the first miner work)
"82fd9527291ba30cdddeb5786bac083f8092987e379c9433d81a2eb1bb8c447a" - 12th hash (proof of the second miner work)
"10eac3abc33a75b16c1ca33aaf97db92542a452453265bf5c088dc33be750137" - 13th hash (proof of the third miner work)
"fdbf00e59e8b4d7ae44950751694ebeb2bb6ac969e166d4500d9bfbd886d7523" - 14th hash (proof of the forth miner work)
"2405d8048e0fead22be4aa4394104136e15c0ca578299068c2a9dbf3c7c68b59" - 15th hash (proof of the fifth miner work)
“905f5f8032ff6c956422ac0560431820e2bfa47bb0cf58368fd49dbc1e23e48c" - 16th hash (proof of the sixth miner work)
```
Once the last confirmation ordered by user is approved by the majority of the network, the object becomes `Approved` and available for further operations with the asset. 

The object becomes `NotApproved` 5 blocks after last confirmation or the block, at which it was `Estimated`, if there is still no new confirmation available.  

Every block author provided the network with correct judgement is getting rewarded with its share, which is equal to 70%*fee/n, where n is the number of confirmations ordered by user. In this example n=6 and 70% fee = 70 P3D So, every honest miner gets 11.666666666666 P3D.

This is a snapshot example of the object approved from the storage (short version, the data is presented partially. See more [POSCAN API](https://github.com/3Dpass/3DP/wiki/3DPRC%E2%80%902-PoScan-API)):
```
[
    [
      4
    ]
    {
      state: {
        Approved: 2,699
      }
      obj: 0x6f200a7620302e313520302e383520302e320a7620302e313520302e383620....
      category: {
        Objects3D: Grid2dLow
      }
      hashes: [
        0x7fbe267b208d0ab9c34fa184d4593d0ae39d5bb56852ecc633cb1f6d9eb5aae0
        0x460c81f58510e211d162d458459f81575e630817c1b87bc3c0d2c3eec3ae68b7
        0x8047c7a4d2a75ce466c9510edb37ac1e98826ef383ab5d4c860b5010e42faa6e
        0x5001723ef6d85862733d25765f0700e381cc3a1f6d6b7dbb4efb8aeccc4615d4
        0xb8efd617a07099edcdbf2596d10787a6e16a4c59a5e36e46adedc1dd2424b8c9
        0x10c8d4984cf840845a7d7785871c70790bf1c55e11a3da54a41d3f21be5a7ea7
        0x833a2fc1f26ab9eb9bcc5b4e668f5d6bb91d6f87fb0798f99ce6bb4730dfc218
        0xabdedd28e03147d94e0d054d0bf24781bac01fdd81401cd3fb226b5a6e1a5371
        0x97c538a99641202385572df245d9cfb300f895a6c94e7caf074023a5ead1c62c
        0xdb2453f53270c9003a63b4d643d9bcc991ad7630d70a0d1511781f74a7a00fee
      ]
      whenCreated: 2,690
      whenApproved: 2,699
      owner: d1f5KGsoZ3xzB6Ecmv92DPizD1x5eToNHM1CPfSC1Xu4nzecN
      estimators: [
        [
          d1ePg4fK97U913xAnZzZ9dUf1W1XUA4bYVC2Zre8K9vjSixnc
          36
        ]
        [
          d1asoD7V6hdff4ExvtjRbdVT398Rg8fKEruYsKFS5P9mkT4fy
          32
        ]
      ]
      estOutliers: []
      approvers: [
        {
          accountId: d1jygGfK97U913xAnZzZ9dUf1W1XUA4bYVC2Zre8KTrgnjs6
          when: 2,695
          proof: 0xab198306dcee347bd84a942ffc9ab06c8458865fee03c5d1b20ab45b807dc1c5
        }
        {
          accountId: d19ouGfK97U9edj69nZzZ9dUf1W1XUA4bYVC2Zre8K9Jgclj
          when: 2,696
          proof: 0x82fd9527291ba30cdddeb5786bac083f8092987e379c9433d81a2eb1bb8c447a
        }
      ]
      numApprovals: 6
      estRewards: 70,000,000,000,000,000
      authorRewards: 30,000,000,000,000,000
    }
  ]
]
```

## New algorithm requirements
This protocol is proposed to be a standard for the tokenization of objects corresponding to the categories specified (see more “THE OBJECT CATEGORIES AND RECOGNITION ALGORITHMS”)

- Every recognition algorithm to add must provide “0 knowledge proof” about the object, so that the network can verify and agree upon its authenticity. 3D object authentication procedure utilizing the grid2d recognition algorithm can be used as a reference. 
- Every recognition algorithm must be open source and free from license restrictions, which could prevent its distribution for free.
- Every recognition algorithm must be able to get objects processed within the timeframe limit of 10 sec, while running on average full 3dpass node.
- Every recognition algorithm must rely on public data, stored on 3dpass blockchain (objects, HASH IDs, zero knowledge proof).

## Property rights definition
One of the most important challenges related to the tokenization of objects all across the entire blockchain world is how could the property rights be initially defined. Simply speaking, how could we verify the actual owner of the object/asset at the time the object is being submitted by someone claiming they have all the rights required for.

3DPRC-2 proposes a trustworthy procedure for the property rights definition and its correction after any potential changes, which might had happened as an outcome of any argument or adjudication, any other legal action leading to the rights correction outside of the blockchain storage, which needs to be taken by “The Ledger of Things” for direct order and, finally, got executed. 

Let’s classify the property rights into the following aspects: 

**1. Common public assets**

Intangible or tangible assets the property rights has expired for (50 years after the author’s death has actually lasted by the time the asset appears on “The Ledger of Things”). For example, it could be a piece of classic music or Venus de Milo statue. Those assets are most commonly priceless and obtained by the humanity as a whole, representing the legacy of human civilization. Everyone is allowed for its replications commercial use (ex, anyone can listen to and make a copy of any piece of classic music without any license fee requirements/restrictions). Although, the original object properties can be identified by the recognition algorithms (its shape, weight, melody, voice, etc.)

**2. Private assets**

Intangible or tangible assets, the property rights of which are established and still valid by the time the asset appears on “The Ledger of Things”. The asset is obtained by either a person/group of persons or an organization. In terms of the world wide intellectual property legislation, the property rights have been valid since the first reliable publication made by its author. Tangible assets are usually published on the government registries (real state, vehicles, etc).  

**3. Unpublished assets**

Intangible or tangible assets, the property rights of which are NOT established by the time the asset appears on “The Ledger of Things”. For example, it could have been something like new 3D statue or new pop music track, anything unpublished yet.

### Common public assets
3DPRC-2 will always allow for anyone to issue crypto currencies (fungible and non-fungible) backed by any common public asset stored on “The Ledger of Things”, due to the fact that there is no restrictions could be applied.

### Private assets
3DPRC-2 will always guarantee for anyone, who is the actual object owner, to establish their property rights towards the asset stored on “The Ledger of Things”, provided the ownership has been verified by means of special property rights extension(option) of The Object authentication protocol (see more THE OBJECT AUTHENTICATION PROTOCOL).

The extended option of the protocol will use a list of trustworthy external resourced, which data can be considered reliable enough for making decisions about the on-chain private assets, to verify the asset owner.  

For example: 
- https://www.wipo.int/ - WIPO (World Intellectual Property Organization)**

The list of resources can be moderated by the Asset committee** vote.

**The ownership verification protocol:**

1. By means of leveraging  pass3d** recognition toolkit, the object owner must have created the multi-object HASH ID, the seed of which would be presented as a combination of the following components:
   A. The object tokenized (ex. 3D model in .obj format)
   B. The owner biometric data (optional)
2. By means of using 3dpass wallet** signature, the object owner must have composed a message in the format as follows: 
```
-- Start message --
object: 0x6f200a7620302e313520302e383520302e320a7620302e313520302e383620.... category: Objects3D: Grid2dLow hashes: 7fbe267b208d0ab9c34fa184d4593d0ae39d5bb56852ecc633cb1f6d9eb5aae0 460c81f58510e211d162d458459f81575e630817c1b87bc3c0d2c3eec3ae68b7 8047c7a4d2a75ce466c9510edb37ac1e98826ef383ab5d4c860b5010e42faa6e 5001723ef6d85862733d25765f0700e381cc3a1f6d6b7dbb4efb8aeccc4615d4 b8efd617a07099edcdbf2596d10787a6e16a4c59a5e36e46adedc1dd2424b8c9 10c8d4984cf840845a7d7785871c70790bf1c55e11a3da54a41d3f21be5a7ea7 833a2fc1f26ab9eb9bcc5b4e668f5d6bb91d6f87fb0798f99ce6bb4730dfc218 abdedd28e03147d94e0d054d0bf24781bac01fdd81401cd3fb226b5a6e1a5371 97c538a99641202385572df245d9cfb300f895a6c94e7caf074023a5ead1c62c db2453f53270c9003a63b4d643d9bcc991ad7630d70a0d1511781f74a7a00fee
-- End message --

-- Start P3D wallet signature --
0xfa020a57c5ef765a65610d5985b7bb391f95abf79adb9eaa598e751d5f9ea02b00673d879f826f347565473db512ce472711c6f7742bf546df3723ec610f928c
-- End P3D wallet signature --

-- Start public key --
d1ELrwRkSw5bXwg7NVSbpWMdjdc7oBF32ESx2cZpCUfqXZwhe
-- End public key --
```
The message must be signed with the owner P3D account (in the example above `d1ELrwRkSw5bXwg7NVSbpWMdjdc7oBF32ESx2cZpCUfqXZwhe`).

3. The object owner must have published the message on the db of any reputable resource from the actual list available on the network storage (for example https://www.wipo.int/ - WIPO). The message must be accessible via the open public API (as an additional properties, for example), so that the validators can verify the data from all around the globe automatically.
4. Once having all these 3 steps above done, the object owner submits the authentication order (see more THE OBJECT AUTHENTICATION PROTOCOL) with  the option ‘ownership.proof’. A public link/reference to the object on the external data base must be provided as the proof.
5. The object authentication protocol will be executed as it is described in the  THE OBJECT AUTHENTICATION PROTOCOL chapter, but with the couple additional rules. The rules are:
   A. The network nodes will try to verify the message via the API (external worker (oracle) will be used on the node side). The judgement “Approved” will be accepted if the message will be verified on block import, otherwise the block is going to be rejected;
   B. If any copycat of the object has been discovered, the actual owner/owners will be applied.

**Unpublished assets**

The combination of such aspects as the timestamp, copy protection, data change protection make “The Ledger of Things” one of the most reliable decentralized data bases to make the first publication for any new object created. It would be enough for its owner to get the object through the authentication procedure and claim they have all the rights required. 

## Digital asset structure
It is being worked on…

## Backed currency issuance
It is being worked on…

## PoScan API
- Explore the actual API methods available on WIKI: [PoScan API](https://github.com/3Dpass/3DP/wiki/3DPRC%E2%80%902-PoScan-API)
- PoScan pallet implementation: [PoScan pallet](https://github.com/3Dpass/3DP/tree/test/pallets/poscan)

## Reference
- Grid2d recognition algorithm: [https://3dpass.org/grid2d](https://3dpass.org/grid2d) 
- HASH ID: The object identity comes out as a result of the object recognition ([https://3dpass.org/features#3drecognition-hash-id](https://3dpass.org/features#3drecognition-hash-id)) 
- pass3d recognition toolkit: [https://github.com/3Dpass/pass3d]([https://github.com/3Dpass/pass3d)
.obj format: [https://en.wikipedia.org/wiki/Wavefront_.obj_files](https://en.wikipedia.org/wiki/Wavefront_.obj_files)
- Asset committee: A group of reputable members presented as 3dpass network accounts elected by Council** vote and eligible to moderate the assets-related content available, such as: 
  - Asset categories and algorithms;
  - The list of external resources for the verification of ownership.  
- Council: A group of reputable members presented as 3dpass network accounts elected by the majority of P3D holders [https://3dpass.org/governance#council](https://3dpass.org/governance#council)
- WIPO: [https://en.wikipedia.org/wiki/World_Intellectual_Property_Organization](https://en.wikipedia.org/wiki/World_Intellectual_Property_Organization)
- 3dpass wallet: [https://github.com/3Dpass/wallet](https://github.com/3Dpass/wallet)


