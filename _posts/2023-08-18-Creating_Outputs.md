---
title: UTXO model in Shimmer
date: 2023-08-18 08:00:00 +0800
categories: [Tutorial,Documentation]
tags: [outputs,ledger,qt]   
img_path: /assets/img/
image:
  path: posts/utxo/UTXO_model_in_Shimmer.png
---

This post explains the Unspent Transaction Output (UTXO) model on [Shimmer](https://shimmer.network/).
The linked video and [code repo](https://github.com/EddyTheCo/OutsShimmerppExamples) show how to create the different Outputs using [Shimmer++](https://eddytheco.github.io/Shimmerpp/about/#main-components) libraries.
Make sure to watch the video to get a more complete understanding of the UTXO model! 

The UTXO model defines a ledger state where balances are not directly associated with addresses but with the outputs of transactions.
Every output must have a base token amount greater than 0 and fulfill the [Dust Protection Requirements](https://wiki.iota.org/shimmer/tips/tips/TIP-0019/).
The output states the relation between balance and address through its [unlock conditions](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#unlock-conditions).
 
The unspent outputs are very important because they are the final result of the protocol.
The protocol is followed by the nodes with the sole purpose of changing their ledger state(creating unspent outputs).
The nice part of the protocol is that it does that in a secure, permissionless, and decentralized manner.
When referring to decentralization, I mean that everyone can own outputs on the ledger and validate ledger state transitions.

# UTXO model in Shimmer explained simple

A transaction must be proposed to the node to modify its ledger.
The transaction is composed of two parts,
- [x] `Transaction essence`
- [x] `Unlocks`

The `Transaction Essence` is composed of
- [x] `Inputs`
- [x] `Outputs` 

where `Inputs` are references to unspent outputs existing on the ledger when the transaction is created.
The `Outputs` propose new unspent outputs to be created on the ledger with the acceptance of the transaction.
The latter are mainly composed of
- [ ] `Features`
- [x] `Unlock Conditions`

The `Outputs` can hold native tokens(user-defined tokens).
Every output sets its `Unlock Conditions`, simply set the conditions to use that output as input in another transaction.
Following this, the `Unlocks` property of the transaction proves to the node that the creator  of the transaction can fulfill the `Address`-related `Unlock Conditions` of the `Inputs`.

The `Outputs` in the `Transaction Essence` can be of four different types
* Basic Output
* NFT Output
* Alias Output
* Foundry Output

I find it simple to understand the UTXOs on the ledger as a public database.
A database whose entries are the unspent outputs.
Importantly, not everyone can modify any entry on the database, the `Unlock Conditions` of the entry sets who and how it can be modified.
Some types of outputs have [chain constraints](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#chain-constraint-in-utxo),
so how the entry on the database depends also on the transition rules defined for the given type of output and its current state.


## [Basic Output](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#basic-output)

It is called basic although can hold many of the `Features` and `Unlock Conditions` an output can have. 
The different `Features`  of the `Basic Output` are
- [ ] Metadata Feature
- [ ] Tag Feature
- [ ] Sender Feature

### [Metadata Feature](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#metadata-feature)

`Outputs` with this feature carry additional data, and this data will exist on the database until the output is spent.
A simple example of the use of this feature is an IoT device that creates output on the ledger with the `Metadata Feature`.
The metadata has the device geolocation coordinates. 
In a simple public manner, the device can share its position for client applications to consume this data.

### [Tag Feature](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#tag-feature)

A `Tag Feature` makes it possible to tag outputs with an index.
In the previous example, the IoT device has to share some output identifier with the client for this to be able to find 
that output on the ledger.
Adding a `Tag Feature` with the index "IoT position" makes it easier for the client to find that output and the data.

 
### [Sender Feature](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#sender-feature)

Normally, entries on the public database do not say who created them. 
If an entry(Output) has the `Sender  Feature` it links the creation of the output with some `Address`.
This link is [secured by the protocol](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#additional-semantic-transaction-validation-rule-1).
Following the example of the IoT device, adding the `Sender Feature` to the output allows the client applications to certify from which IoT device this metadata came from.

The different `Unlock Conditions` that a `Basic output` can hold are
- [x] Address Unlock Condition
- [ ] Storage Deposit Return Unlock Condition
- [ ] Timelock Unlock Condition
- [ ] Expiration Unlock Condition

### [Address Unlock Condition](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#address-unlock-condition)

This condition sets that to use the output as input in another transaction(unlock the output) one has to prove ownership of a certain address.

A side note, currently the different  `Address` types on Shimmer are

* Ed25519 Address
* Alias Address
* NFT Address

To prove ownership over an `Ed25519 Address` one has to sign some data using the private key of the address.
Refer to [proving ownership of the Alias and NFT Address](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#unlocking-chain-script-locked-outputs) for the other types.

By setting as `Address Unlock Condition`  of the IoT output an address the IoT device controls,
the device can destroy that output on the ledger and create a new one with its location updated.

### [Storage Deposit Return Unlock Condition](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#storage-deposit-return-unlock-condition)
If an output has this condition, to consume the output in a transaction a new `Basic Output` without `Features` nor `Native Tokens` and with only an `Address Unlock Condition` must be created.
The condition sets the `Return Address` and `Return Amount` values that must be used in the `Address Unlock Condition` and `Amount` of the new output for the transaction to be valid.

This condition allows [microtransactions on Layer 1](https://wiki.iota.org/shimmer/tips/tips/TIP-0019/#microtransactions-on-layer-1).
The outputs with this condition and `Native Tokens`, allow the creator of the output to change the ownership of the tokens without 
losing the storage deposit of the output.

As an example, a client dapp validates that the coordinates published by the IoT device are correct.
To recognize the good behavior of the device the client creates a `Basic Output` with an `Address Unlock Condition` set equal to the `Sender Feature`  value of output the device uses to publish its coordinates.
The latter output has a certain `Native Token` and a `Storage Deposit Return Unlock Condition`.
The `Return Address` on the `Storage Deposit Return Unlock Condition` is an address the client controls and the `Return Amount` is equal to the output `Amount` value. 
By doing this the `Native Tokens` on the output are now owned by the IoT device with the condition that it has to take care of the storage deposit carrying those tokens. 


The `NFT Output` also can have this condition, and in the same manner, this can be used to change the ownership of NFT outputs without losing control over the base tokens needed for the creation of the Output. 


### [Timelock Unlock Condition](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#timelock-unlock-condition)

Outputs with this condition can not be used in a transaction as input until a certain `Unix Time` has passed.
The notion of time in the protocol is  introduced via the `Timestamp` value of [blocks with a `Milestone Payload`](https://wiki.iota.org/shimmer/tips/tips/TIP-0008/).

As an example, the IoT device to update its position in a new output could consume the output with the previous location data.
To avoid this the basic output with the data could include this condition.
In that case, the device can be certain previous outputs will not be consumed until a certain `Unix Time` has passed.


### [Expiration Unlock Condition](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#expiration-unlock-condition)

This condition sets a `Unix Time`, to unlock the output after this time, proof of  ownership of the 
`Address` in the `Address Unlock Condition` is not needed. 
Instead, a prove of ownership of the `Return Address` value of the `Expiration Unlock Condition` has to be given.


As an example, when the client application creates the output to transfer some tokens to the IoT device for its good behavior.
The client will add an `Expiration Unlock Condition` to the output, with `Return Address` an address the client controls.
In that manner, after some time the base tokens on the output will be always available to the client.
From one side the device could consume that output before the `Unix Time` has passed or the output can be only unlocked by the client. 



## [NFT Output](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#nft-output)

This output supports the same `Features` and `Unlock Conditions` that the `Basic Output`.
Different from the `Basic Output`, the `NFT Output` has an `NFT ID` and also `Immutable Features`.
This ID identifies a chain of UTXO spends and the `Immutable Features` form part of the [chain constraint](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#chain-constraint-in-utxo) for this type of output.
When an output with chain constraint is consumed, that transaction has to create a single subsequent output that carries the state forward. 
The chain constraint sets the conditions for how a transaction can transition the chain state. 
For this type of output, the subsequent state of the chain has to be an `NFT Output` with the same `Immutable Features` and `NFT ID` implicit values.

The different `Immutable Features` that a `NFT Output` can hold are
- [ ] Issuer Feature
- [ ] Metadata Feature

### [Issuer Feature](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#issuer-feature)

If a `NFT Output` has the `Issuer Feature` it links the creation of the chain of UTXO spends with some Address.
In the case of the `NFT Output` the creation of the chain is normally called NFT mint.


As an example, the IoT device could use an `NFT Output` instead of a basic one to publish its location.
The producer of the devices mint a NFT for each device and an address controlled by the producer is set in the `Issuer Feature`.
The producer transfers ownership of the `NFT Output` to the device. 
Now the client application apart from certifying which IoT device published the data(by using the `Sender Feature`), also can certify that the device comes from a certain producer.

**The Immutable Metadata Feature** can  be used by the producer. 
The producer could add to this field the different physical properties the IoT device has, making the `NFT Output` a Digital twin of the physical device.


## [Alias Output](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#alias-output)

This type of output is also subjected to chain constraint with the global ID of the chain `Alias ID`.
It allows the `Sender Feature` and `Metadata Feature` and also the `Immutable Features`, `Issuer Feature` and `Metadata Feature`.
The output introduces new fields like `State Index`, `State Metadata`, and `Foundry Counter` but admits only two `Unlock Conditions`
- [x] State Controller Address Unlock Condition
- [x] Governor Address Unlock Condition

Depending on which unlock it is fulfilled the chain constraints are different. 
Due to this, one can say the `Alias Output` has [two levels of control](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#consumed-outputs-1).

### [State Controller Address Unlock Condition](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#state-controller-address-unlock-condition)

This condition sets the address the ownership has to be provided to unlock the `Alias Output` with State Controller control level.

### [Governor Address Unlock Condition](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#governor-address-unlock-condition)

This condition sets the address the ownership has to be provided to unlock the `Alias Output` with Governor control level.

As an example, the producer can create an `Alias Output` with an address he controls as a `Governor Address Unlock Condition`.
The `State Controller Address Unlock Condition` is set to an address controlled by a certain factory.
The producer as Governor can change which factory can unlock the `Alias Output` to mint new NFTs-Digital-Twins.
In this case, the different factories can mint the NFTs-Digital-Twins of the produced devices, and set as issuer the same `Alias Address` linked to the producer.


## [Foundry Output](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#foundry-output)

A foundry output is an output that controls the supply of user-defined native tokens.
It can mint and melt tokens according to the policy defined in the Token Scheme field of the output. Foundries can only be created and controlled by aliases.
It only allows the `Metadata Feature` in both `Features` and `Immutable Features`.
The output also has a global ID called `Foundry ID` and admits only one `Unlock Condition`


- [x] Immutable Alias Address Unlock Condition

### [Immutable Alias Address Unlock Condition ](https://wiki.iota.org/shimmer/tips/tips/TIP-0018/#immutable-alias-address-unlock-condition)

This condition sets the `Alias Address` the ownership has to be provided to unlock the `Foundry Output`.
As part of the chain constraints, the latter address it is not allowed to change in future transitions of the output.
Due to that, a `Foundry Output` is always controlled by the same `Alias Address` that was set at its creation.

As an example, the developer of the client Dapp will have to create a `Foundry Output` that mints a certain amount of `Native Tokens`.
The different clients can choose to buy the tokens from the developer to recognize the good(bad) behavior of the IoT devices.

## Conclusion

We have described in simple terms :smile: the UTXO model in Shimmer.
Also, I have shown examples of how this and the protocol in general can be used in the field of IoT devices.
The use cases of this level of security and decentralization are infinite and can be used in many different areas.

The purpose of these posts is to create a community around the Shimmer ecosystem.
A community that shares knowledge and contributes to the development of applications that trust the Shimmer nodes.
Find bugs, and typos, learn and teach us what you know by contributing!
 
In future posts, I will explain how to use the Shimmer++ libraries to extend QML. 
We will develop a custom QML type(a GUI object), that can interact with the protocol and can be easily reused in different GUI applications.
Please let me know in the comments if you find it useful. Let me know your doubts about the Stardust protocol, the Layer 1 of Shimmer, and Shimerpp.

## Watch the video! 
{% include embed/youtube.html id='ibErRWgGI1M' %}
