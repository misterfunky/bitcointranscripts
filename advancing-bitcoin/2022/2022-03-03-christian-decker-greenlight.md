---
title: Deploying Lightning at Scale (Greenlight)
speakers: ['Christian Decker']
date: 2022-03-03
transcript_by: Michael Folkson
categories: ['conference']
tags: ['lightning']
---

Video: <https://vimeo.com/703273144>

Blog post on Greenlight: <https://blog.blockstream.com/en-greenlight-by-blockstream-lightning-made-easy/>

Greenlight demo at London Bitcoin Devs: <https://btctranscripts.com/london-bitcoin-devs/2022-03-01-lightning-panel/>

# Intro

Jeff Gallas: Next up is Christian Decker. He is a Lightning developer at Blockstream. He has got a very simple title. It is Security, Simplicity and DevOps: the trade-offs between a Lightning PPAS and running your own node. That is already half of the talk.

Christian Decker: Thank you very much, I’m Chris and for once I am not going to shill eltoo, I promise. Today I am going to talk about my pet project that I have been working on for the last year or so. It is all about how we can onboard the next generation of Lightning users onto the network. I think you will like the technical challenges and the trade-offs that we make in all of this. I shortened the title to be “Deploying Lightning at Scale” which is much, much shorter. 

# What’s our goal?

So what is our goal with Lightning? What do we want to do? We want to give everybody the ability to join the Lightning Network in whatever fashion they want to and we want to enable them to have access to the Bitcoin economy and the Lightning economy. However that means that we have to take care about many different user profiles. Most of us here are pretty technical and are ok with running relatively complex systems. But many users aren’t. I might be able to run a Lightning node but I don’t want to force users to have to learn about all the intricacies of the Lightning Network before they are even allowed to look sideways at Lightning. What that means is we need to have a spectrum of solutions that match the different user profiles and potentially provide users with the ability to upgrade their experience over time as they learn to use the Lightning Network. The sad truth is that it would be nice to have fully self sovereign users taking care of their own infrastructure and know all of the intricacies of the Lightning Network. We as Bitcoiners very often have the reaction “No you are not allowed to even look at Lightning before you’ve read this 10,000 tome to first learn about Bitcoin, then you learn about Lightning, then you learn about invoices, then you learn about all of the risks when you don’t run your own infrastructure in a safe way”. Running watchtowers, running databases, running backups. Who here knows how to securely back up a Lightning node? I’m not raising my hand by the way. We have to somehow make it approachable for even non-technical users and we have to be ok with the users choosing their own trade-off when it comes to security and privacy. We should be there to help them through this choice by being objective about the trade-offs that we are facing for one choice or another. 

# A Lightning Software Taxonomy

Let’s quickly look at a Lightning Network taxonomy. What are the choices for new users that want to join the Lightning Network? First of all we have the split between custodial and non-custodial. Custodial wallets usually have the best user experience. There is no way around this. You don’t have to learn anything to use a custodial wallet. But we don’t like custodial wallets that much do we? It is basically trusting somebody else with your money. They might just abscond with your funds. Custodial wallets are then further split into account based and hosted channels, sometimes also called multi-tenancy systems, built on top of a single Lightning node that is under the control of some operator. You are trusting that operator with the entirety of your funds. If they run away you are out of luck. If they get hacked you are out of luck. On the non-custodial side we have the self hosted version which has been so far the go-to case for many of you I believe. In that case you have to learn about how to operate a Lightning node, how to set up all of the infrastructure: databases, watchtowers, backups etc. If I drop my phone in the lake what do I do now? With great flexibility comes a lot of responsibility as well. Then we have the split into do we want to have a node per device? That is often a choice that has been made so far where we build a node into the Lightning application. There are now really good tools to do that, LDK is an excellent example of how to do that, but it also means that if you use more than one app you are now operating two wallets. You are splitting funds across these two wallets. If you have 50 here and 50 here you can’t pay 60. There is no way of combining them. It also means that the management overhead that you have is multiplied by the number of apps that you have. If you have two apps you now have to manage twice the number of channels, you have to manage twice the number of backups, you have to pay twice the watchtowers and so on and so forth. It also means that if you drop your phone in the lake, bye bye funds basically. Then there is the remote control node scenario where you have some sort of box at home. You are remoting into that node from the outside. Personally I quite like this because it means that I can have multiple apps be frontends to the same node. The funds are concentrated on a single node and I don’t run into this issue that I might not be able to pay an invoice despite having sufficient funds, if only they were combinable. The upside is I can drop my phone in the lake again and my funds are still safe. That’s kind of nice. Today I am going to present a mix between these scenarios. That is what I call remote control and remote signer. We will see how this can be used to get the best of both worlds. Make it easier to assist new users until they have learnt about the Lightning Network to take on responsibility for their own node and how they can migrate from this scenario over to this scenario. It enables a lot of flexibility and at a later point in time, once you have seen the upside of what this can do for you, you are incentivized to learn and upgrade your experience to take care of your own node. What I didn’t mention is that remote controlled node and remote control/remote signer setups also have a way smaller capture feature. Namely if you integrate the node into the app then you don’t have the flexibility of trying out the new app that might have a new cool interface. You have to tear down the entire node, transfer funds and then spin up the entire node again just to find out this UI actually doesn’t work for me so I’ll migrate back. With the remote setups you can switch between front ends and even use them concurrently as often as you want. You could have the same node be fronted by a web browser and a mobile app. Or you could give your kids access to parts of that node in the form of an allowance for example.

# Tradeoffs, tradeoffs everywhere

Trade-offs, trade-offs everywhere. We have to be clear about what these trade-offs are. Only when we are clear about what these trade-offs can the user that we’re trying to onboard into Lightning make an informed decision and make the right choice. We can’t do much else than propose and offer solutions to the issues that they face. But ultimately the choice is theirs. We probably should respect their choices or encourage them maybe to learn more and upgrade their experience.

# Greenlight (Onboarding the next generation of users)

The system I am going to talk about today is called Greenlight. It is something that we’ve been mulling over for the last couple of years in the background. Only recently we decided that we’d give it a shot. Let’s see how it works. Like I said this is to onboard the next generation of users. It is probably not for you but it is something that you can use to offer a very cool user experience that includes Lightning payments without having to bundle nodes with it and without you having to care about the intricacies of Lightning itself. It frees developers from having to write node automation software themselves as well.

# We host the node, you keep the keys

The basic idea is we host the node, you keep your keys. It turns out there are many pieces in this Lightning infrastructure that can be shared quite easily. There are quite a few pieces to run Lightning. You have the node itself, many people stop at that point. Then you have the bitcoind backend which usually is the costly part, downloading 450GB of data, crunching through millions and millions of signatures. That doesn’t make for a quick and easy onboarding experience. Then you have the database itself, this database has to be secure, ideally it would be replicated. And you’d have to run backups, backups are a very complex topic in Lightning and we are trying to make it easier. For now real time backups or bust basically. Then you have the watchtowers. Who here has configured a watchtower? I can say I have because I run on Greenlight. Learning about the network topology, that’s a big issue for mobile phones for example. You whip out the mobile phone and then you wait for a couple of seconds until it synchronizes the network graph so you can perform a payment. There are other ways around this, trampoline by Bastien (Teinturier) is one of the solutions that is excellent but it would be nice to be able to do that locally. Quite a few more things that one might consider. One key point here is that many of these can be run efficiently and shared across many, many nodes. Every single user having to set up this is a huge overhead when in reality we could do that on your behalf. 

# Splitting a node

What we did is we looked at the c-lightning architecture and right from the get go we had this part. This is the aspirationally named hsmd. This is the only part that ever touches keys. This is the part that actually signs off any change in balance, any coin movement whatsoever is going through the hsmd. When I say it is aspirationally called hsmd, we hope to eventually have hardware wallets that will be able to run this code and have the keys segregated into a separate system that has very narrow interfaces that can be easily audited and be made secure. How do we go about running the node for you while you are in control of the funds? Let’s take this and move it over to the user. All the rest can be run on our servers. It is easy? It turns out that it is not that easy, it is quite a long research project and it is still not quite done. 

# Remote signers

What we can do is take this sequence diagram and see how this works. On the right hand side we have the Lightning Network, if you were to self host a node you would run all of this yourself. With a setup similar to Greenlight what you do is have the Lightning Network which is the public network, then you have the node operator which in Greenlight is Blockstream and then you’d have the third persona which is signer and front end. These could be the same device but splitting them out makes it much more flexible. We can also have as many front ends as we want, there is no limit to this. You can attach your mobile phone, your browser, whatever to it and that’s ok. How does this work? The front end running on a phone let’s say scans a QR code and instructs the node to pay an invoice. The node then takes this command and transforms it into a series of updates to the internal state. These are updates to the channels, to the onchain funds and so on. It cannot sign off on any of these changes. What it does next is it asks for the signer running at the user for the necessary signatures. The signer receives not only the signature request but it also receives as context the command itself and the state of the channels before this change was applied. It then authenticates this change making sure it comes from a front end that is allowed to issue commands and not some rogue operator on the node itself or an attacker that has taken over the server running the node. Only if these changes are authenticated it will send back the signatures to the node. The node takes the signatures, writes it all to the database, performs the changes, sends out the messages to the peer and in this case adds the HTLCs. It will then receive from the network “Success” or “Try again”. Then this loop restarts, going through this loop as many times as necessary for us to complete the payment. The last leg would report a success back to the front end. One thing that you can see is the signer can be used to create an audit trail of all changes in the node.

# More benefits

What are the benefits of this? Like I said it is just a remote controlled node. Many front ends are possible. It also makes for trivial recovery. What we built here is a way where you can just from your seed talk to our servers and recover access to your node. Even if I were to drop my phone in a lake I still have my seed phrase, I can contact the server, it gives me access again and I can continue working with my node without any downtime, without closing channels or anything like that. No splitting of funds, that is true for every remote node setup. You can really easily go from this hosted setup to a self-hosted setup. This is where upgrading your own experience comes in. We want you to have a good experience upfront and then you are incentivized to learn about the intricacies of Lightning. Then we give you the opportunity to go for a self hosted setup,  we give you a backup of your node, you can import it into your hardware and we are out of the loop completely. Any number of frontends. And all of this will be open source. We are already sharing this with a lot of teams that we are integrating with. As soon as we go for a public beta this will become public. This allows you to have what some call a Uncle Jim setup. If you have somebody in your friends or family group then they can run this for you. All you have to do is talk to their service and they can run any number of nodes for their friends and family. It turns out c-lightning is really light, we can run about 92 c-lightning nodes on a single Raspberry Pi with 4GB of RAM. You have to be really selective about who are your friends in that case, it is just you, your family and your 87 closest friends. So be selective. The topic here is onboard, give them a good experience, now is the time to educate and then eventually off board. 

# Onboard, Educate, Offboard 

How do we do it? You can teach users to incrementally take on more and more ownership of their node. We can tell you what a watchtower is and you can install it on your own node and take on watching the network for your node. We can stream backups from our servers to your node so that if we were to disappear from one day to the next you could just recover locally and never have any downtime or have to close channels, anything like that. We can drive payments locally. These are very interesting setups where it is very nice to explore some things. We can do trampoline routing, if you have a high latency connection from your phone to the node for example you can outsource the driving of the payment, the sending out HTLCs, completing routes, retrying, all of that you can outsource to an external service. Or you can move that onto your phone. What you can do is complete routes on your phone and all the node operator sees are onions. Just as if they were any participant in the network forwarding a payment. In that case the node operator doesn’t even see where the payment is going. You can have dedicated signers. You could have a Raspberry Pi for your friends, run thousands of these signers 24/7 so that your node is always reachable.

# Make it your own

The ultimate step, once you’ve done all these learning steps you can just come to us and say “I don’t need you guys anymore” and we’ll happily let you go. You can just ask for a backup of the node, we will mark the node as exported and you can import into your own structure. 

# Privacy considerations

I mentioned before that in order to make an informed decision we also have to be honest about the upsides and downsides. One topic that I’d like to address here is that Greenlight is definitely not as private as running your self hosted node. This is a good reason for you to eventually off board. Once you’ve learned about Lightning there is no reason to stick with us. We’ll happily let you go. The server sees invoices and payments. We are working on driving payments from the front end such that we only see onions. There is work ongoing where we can encrypt invoices and we need the signer to accept payments anyway so why not let the signer give us the payment hash instead of having it on the server. On the upside, since we have access to the logs we can help you debug issues that you may have. You can tell us there’s something wrong and we can look into the logs and tell you exactly what went wrong. A nice side benefit for us is this informs us the future direction that c-lightning and the Lightning Network as a whole could go in future. It allows us to verify what is working, what isn’t working. It allows us to test whether splitting payments eagerly or delaying payment splitting works better. With pathfinding whether using one method works better than the other and so on. This has already given us a couple of things that we have moved back into the open source c-lightning project. These nodes that we are running run vanilla c-lightning. Anything we learn here is applicable to the wider network without much translation. We can assist you if you have issues. Ultimately it is the choice of the user whether that is a compromise he is willing to make. I think if we are clear about the trade-offs we can with this method get many, many more users to actually be interested in Lightning and not just stick with the usual default custodial wallet that everybody considers the state of the art. Still give them a way to upgrade their experience, become a fully self sovereign participant in the Lightning Network and be an informed citizen in our community.

# Where are we today?

So where are we today? Greenlight is working, you can register in less than a second, you can spin up the node in about 200 milliseconds. It takes about half a second for us to pump down the entirety of the gossip data. After about half a second you have a fully functional node that you can use to perform payments. That is usually shorter than it takes the app on your phone to load. We are pretty confident that this quite nice. We are working on LSP integrations. Part of our pitch was to be able to take the user by the hand and guide them through the process, liquidity being one of the big issues that every Lightning user faces nowadays. We want to have a solution for that and propose channels from a number of LSPs that we are integrating with. Public beta coming soon. I was about to say 2 weeks but I’m aware that not everybody takes that promise with the grain of salt that we’re used to. It is just soon now. Two weeks usually translates into 18 months. It shouldn’t be that long but it will be done when it’s done. If you are interested in exploring these options please talk to me. Like I said we will open source all of this and you will be able to run this on your infrastructure. Please come and compete with us. I don’t want to run the entire Lightning Network. Thank you, questions.

# Q&A

Q - This doesn’t necessarily solve the onboarding, signing up and then receiving a payment. You still have the channel. There is still this period of “We need to create the channel that is going to facilitate it all”.

A - That is the tricky part that we are trying to get working with the LSP integrations. Part of that is we are asking LSPs to ping us when there is an incoming payment. We can open channels on demand with zero conf and zero reserve. Since we have some control over the node we can make sure that zero conf is actually secure for both parties. That is still work in progress but ultimately the hope is when you go to an invoice creation screen you get told “You are not wired up with the network” or whatever the verbiage is. Have I mentioned that I suck at UX? I am so happy that I can now concentrate on the infrastructure piece and have the UI experts take on the responsibility of what they do best, namely talking to users. On demand, when it is necessary, you can ask them “Would you like us to open a channel to you?” That is the flow there.

Q - Is the API one-to-one with normal c-lightning?

A - I mentioned before that one of the parts that was backported from Greenlight into c-lightning, that is the RPC. We have a very extensive JSON RPC in c-lightning. With Greenlight we have introduced a gRPC binding with mTLS authentication. That has now been backported into c-lightning. We have the entirety of the c-lightning API be exposed over gRPC. That has been a longstanding request that we don’t have a good RPC because it is just local. The R is a lie there. Now we re-expose that through a plugin and that is part of what we built for Greenlight and were able to backport into c-lightning proper.

Q - Do you have custom plugins available on Greenlight?

A - We don’t currently have user provided plugins. There are plans to eventually add two types of plugins, one is vetted plugins. These run on our infrastructure so we’d better make sure that they are somewhat safe. We do partition the nodes quite well but we don’t want to have anything that uses more resources and be a noisy neighbor in this setup. The second thing is that we can create web hook based, third party plugins. You could have an external service and delegate some rights to that external service. Whenever there is an incoming payment we would then ping whatever integration that is and send a message to Slack for example if you have an incoming payment, or Telegram or something like that. Those are the two directions that we are currently looking into though nothing is definitive at this point. 

Q - I was wondering if it is mostly focused on endpoint nodes and not forwarding nodes as well.

A - This is definitely not meant for routing nodes. One of the things that we have to take care of here, routing nodes are a very different beast when it comes to remote signers. Endpoint nodes are much easier to build these kinds of signers for. If you have a routing node you have the skills to run this yourself so don’t be lazy. What I didn’t mention is that all of the learnings that we are having here are also a bit of a research project for us. Ultimately we hope that we can take the signer and make it into a hardware signer as well. Hardware wallets for Lightning nodes, hopefully sometime soon.
