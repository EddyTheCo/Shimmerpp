---
title:  IoT, DePIN and NFT as tickets
date: 2024-02-25 08:00:00 +0800
categories: [Showcase, Discussion]
tags: [IoT,NFT]
img_path: assets/img/
image:
  path: posts/gui/Dlockers.png
---

This post showcases a DePIN example on Shimmer and explains the idea behind using NFTs as tickets in an IoT application.
The linked video showcases an implementation of that idea and how to work with the apps.
If you want to be part of the decentralization, the [code repo](https://github.com/EddyTheCo/DLockers) of the applications is waiting for your contributions.

## What is DePIN?

DePINs is an acronym for Decentralized Physical Infrastructure Networks. 
These networks provide a physical resource and use distributed ledgers to operate.

A sort of centralized version of these Physical Infrastructure Networks is the ride-sharing services like Uber.
Uber drivers (providers) contribute physical resources (vehicles) and services (chauffeuring) and get paid for these services by the users.
To be able to use the service, the providers and clients have to obey Uber's policies.
The data like reputation scores, discounts, and balances  are stored and managed by Uber.
Providers and users do not have the freedom to change the algorithm used.

The 'De' in DePIN comes from the use of a distributed ledger.
The latter makes the network permissionless, where anyone can contribute their resources and anyone 
can also obtain the services offered by a DePIN.
The distributed ledger allows the providers and users to store and manage the balances and data on their own.
Due to the latter, the providers and clients have the freedom to change the algorithm that uses that data. 


## Dlockers, DePIN on Shimmer

Before this DePIN narrative on crypto, I thought of a Proof of Concept of peer-to-peer communication using Iota.
The idea was around a small business model that any person could start providing and could be used by everyone.
The simplest implementation I could show was an IoT device that allows you to rent a box by paying with crypto.
The devices could be seen as decentralized(because anyone could have one) lockers. 
It will be like the Amazon lockers but the providers do not depend on anyone to start providing that service.
The idea is that any private person could start renting physical space and get paid in crypto.
 
### How it works

The Dlockers servers are physical boxes that keep a record of bookings performed by clients.
The client must pay a certain amount to book a box.
The bookings give the right to the client to use the boxes' physical space for a certain amount of time.

The [server](https://github.com/EddyTheCo/DLockers/tree/main/Server) creates a [`Basic Output`](https://wiki.iota.org/tips/tips/TIP-0018/#basic-output) on the ledger that is associated with an address(server ID).
This Output has metadata that describes the state of the server:

- the geographic coordinates of the server.
- the address for receiving payments.
- the amount of coins needed to book 1 hour.
- the different time intervals the box is booked.

The latter `Output` is tagged as `DLockers` so the clients can find these Servers by searching the ledger of their nodes.

The [client](https://github.com/EddyTheCo/DLockers/tree/main/Client) monitors these tagged `Outputs` to show the different Servers on the map.
When the client tries to book a slot on the server a `Basic Output` is created with `Address Unlock Condition` the payment address specified by the server.
This `Output` contains metadata with the slot of time the client wants to book and a `Sender` feature that is associated with the client account.
The `Output` also has an `Expiration Unlock Condition` such that if the server does not accept the booking the client will again control the assets on the `Output`.

The server is monitoring for new booking requests.
Once a new booking arrives the server checks if is possible to book in that interval and if the payment is correct.
If the booking is accepted, the server updates its state and creates a new `Basic Output` with its new state by consuming the old `Output`.
At the same time, the server creates an [`NFT Output`](https://wiki.iota.org/tips/tips/TIP-0018/#nft-output) with `Address Unlock Condition`
the address on the `Sender` feature of the client `Output`. 

The `NFT Output` has also an `Expiration Unlock Condition` and a `Storage Deposit Return Unlock Condition`.
The `Expiration Unlock Condition` is set in such a way that if the client does not use the `Output` before the time slot on the booking the `Output` and its assets will be again controlled by the server.
The `Storage Deposit Return Unlock Condition` forces the client to take care of the `Storage deposit` if wants to use the `NFT Output`.

The client controls an NFT issued by the server.
The NFT has `Immutable Metadata` that describes the time interval the client can open the box. 
To open the box, the client should be in the physical position of the server.
Once there, as requested by the client, the server starts monitoring an address for NFTs issued by itself.
The client `sends` the NFT to the address the server proposed, the client does not lose control of the NFT.
The server checks that `Immutable Metadata` in the NFT allows the client to open the locker.

In principle, the servers do not have to publish their state on the ledger, they could publish a link to its public API.
The use of the ledger to store the state of the server reinforces the security of the protocol, for example against denial-of-service (DDoS) attacks or high load on the public API.
The server could change the node in case its main node fails.

With the protocol explained so far anyone could create fake [Dlockers Servers](https://eddytheco.github.io/DLockers/MockupServer/) and get the money of the clients.
The clients will have the possibility of adding trusted servers once they have visited its physical location.
The different client applications could  list the servers based on the amount of 'Reputation' tokens that are associated with the server ID.
Every client is free to give a score to the server's service and associate this with the Server ID.
In the future, it will be possible to ask the server to lock some balance until the service is provided.

The use of native tokens as a reputation system allows for decentralized integration in different client applications.
The Non-Fungible Token owned by the client is proof that the server has given you rights to use the physical space.
Importantly this right can change ownership because the client could 'send' the NFT to another client.
Also, different servers could give discounts to clients by giving their own `Native Token`, and different servers could use
the same or new tokens to offer  discounts. 
More complicated logic can be explored in the future.


### The `D` in Dlockers

The fact that any person could run a node, server, or client, 
store the ledger state, and validate its transitions allows any person to use the protocol.
The system does not depend on centralized company servers to store your business data or to process payments.
As long as you maintain a node, your gainings and business data are cryptographically secured.
The use of the tokenization framework to assess the reputation, discounts, and physical space rights 
ensures that the protocol will work until there is one node working.
The use of free (libre) software for interacting with the ledger makes them a Dapp and allows the app to continue working even if the main developers disappear.


## Conclusions

We have explained what is DePIN narrative in crypto.
Also explained a Proof of Concept of a DePIN example running on Shimmer.
To be part of decentralization one must run a node, it is the only way one can store the ledger state and validate its transitions.
If one is not able to validate the ledger state one needs to put trust in another party, and this goes against the statement that [`everything is based on crypto proof instead of trust`](https://mmalmi.github.io/satoshi/).
The latter is the basis of crypto.

The purpose of these posts is to create a community around the Shimmer ecosystem.
A community that shares knowledge and contributes to the development of applications that trust the Shimmer nodes.
Find bugs, and typos, learn and teach us what you know by contributing!


Please let me know in the comments if you find it useful. Let me know your doubts about the Stardust protocol, the Layer 1 of Shimmer, and Shimmer++.



## Watch the video!
{% include embed/youtube.html id='UK_493BTI1M' %}
