---
title: Lightning ≈ Bitcoin
transcript_by: Anthony Ronning
tags: ['lightning']
category: ['residency']
---

Name: Dr. Christian Decker

Location: Chaincode Labs Lightning Residency 2018

Video: <https://youtu.be/8lMLo-7yF5k>

## Introduction

Good morning everyone my name is Chris and I'm a bitcoiners anonymous. So my job today is basically to tell you why lightning is Bitcoin and why it's not exactly Bitcoin. So there's a few fundamental differences between bitcoin onchain payment and lightning. I will always call Bitcoin onchain payments just because it's sort of the classical way of doing it.

The differences between Bitcoin and Lightning are sometimes small, sometimes bigger. There's trade-offs when to use which. Some of these trade-offs are basically the reason why we use lighting and some of these are really really annoying details or even big issues that you might encounter when coding a lightning app and I will tell you what to expect and what might go wrong and how to fix that hopefully.

## The Good

So let's begin with the good parts, the parts where Bitcoin is equal to lightning. So payments on lightning are denominated in Bitcoin. This is really fundamental. We are building smart contracts based on the Bitcoin blockchain or whatever blockchain you are using. So the Bitcoins that you use are interchangeable one to one to on chain Bitcoins so there is no need for us to create a token and you wouldn't believe how often I got into this discussion of why we don't do an ICO for Lightning coins. This is basically the reason we want to have bitcoin, we want to use bitcoin, we want to enhance the utility of bitcoin. We don't want to create a whole new thing that has barriers to entry and barriers to exit and makes the whole thing illiquid. Being Bitcoin is we help the Bitcoin ecosystem, we allow people to accept Bitcoin easier, we allow them to extend the use case for Bitcoin, so it's Bitcoin all the way.

Lightning by no means is a complete replacement for Bitcoin onchain payments. There's different levels at which on chain payments become much much better than off chain payments. You have to be aware of these trade-offs and you should choose the right tool for the right job. For example, the fees in Lightning are proportional to the amount you send whereas in Bitcoin on-chain payments, the fees that you pay are proportional to the description of the payment that you send. So in onchain payments, you're actually paying for each byte of the transaction that describes your payment and not the value that is transferred in that transaction. In lightning however since we have a liquidity network we actually need to leverage some fees that are both fixed per transfer so that there's a base fee that we leverage. Basically, I have to do some cryptographic operations to forward your payments so I will ask you to pay one Satoshi for each forwarded payment. And we also have proportional fees which basically mean "hey, I had to put up such-and-such liquidity to forward your payment, please pay me accordingly to what I just transferred." And there's a natural cutoff point that will automatically emerge where the proportional fees are just higher than on-chain fees for really large payments. Well that's okay because for really large payments like you want to buy your sixth Lamborghini, you probably have to fill out some paperwork anyway and it's not like you need to leave the store in the next two milliseconds right. So be aware of this trade-off, sometimes on chain payments might be what you are looking and sometimes lightning is the right decision. So this is about Bitcoin and Lightning being equals.

## Lightning > Bitcoin

Where does lightning actually beat on-chain payments. This is basically what we're looking for right? These are payments that are more private in that we don't have all of our individual transactions be reflected on the blockchain forever. The blockchain can no longer be data mined because what is on the blockchain are aggregates of multiple, maybe millions, of individual payments so picking out that one trace of the one coffee that I buy every day might get really hard for people just looking at the blockchain. We also have in the interactive process of forwarding payments, we don't have many privacy issues because we do use the Onion Routing to obfuscate the origin and the final destination of our payment. It's not perfect, but it's a lot better than just leaving a trace on the blockchain forever.

It's more scalable. Well, if instead of talking to the entire world to perform a transfer, I just talk to my peers directly and we have one-to-one communication for all of these payments. That's got to be more efficient right? And as I said, the onchain footprint can be really tiny compared to the amount of payments that go through a channel. Now that's not always the case, there might be cases where a channel is used for one payment and then it's closed again, but we're working on tooling that should allow you to pick decent routes for all of your channels and to pick decent peers so that you get the most utility out of your channels.

Speaking of fees, we have fewer fees of course. That's great for you, I hope. Well that's as I said is only true for certain size of payments. At some point the market will decide on where this cutoff point is where on-chain transactions are better with respect to off-chain payments. We do have real-time payments, I mean that's what probably most of the new use cases come down to. We actually have this real-time interaction where I can go to a cashier, just tap my phone on the point-of-sale and my phone blips once and I can leave the shop. I don't have to sit around and wait for my payments to complete. It also gives this certainty to the vendor that the payment actually arrived and as somebody yesterday mentioned the pay-per-call system allows you to actually have this real-time interaction even with automated systems in which you pay for an API call. You can pay for streaming of life data though the streaming payments aren't working just yet.

One thing that Bitcoin always struggled with is a cohesive invoice format. So the invoices are really flexible and I didn't like them initially because I was like "yeah it's too much standardization we should protocol less, we shouldn't be talking about how to expose this stuff to users," but I've heard so many people that were actually really happy about the invoicing stuff and it's expressiveness and it's ease of use that I've come around. The interesting part is that if an invoice can't be paid, we actually have a method of failing back on-chain. The invoice contains a number of extra data, it has a description field, it can point to external resources like if you have a PDF that is a human legible invoice, it can commit to the hash of that human readable invoice and like some of you heard yesterday, we also have hints about which channels do have enough capacity incoming to the recipient so we can actually give hints to which channels might be a good choice to talk to. So this is all the good stuff.

## The Bad

Now comes the bad part. I'll be bashing Lightning quite a lot here. I think we need to be honest about the trade-offs that we have here. Now these are trade-offs that are not going to be fixed and we don't have a good solution for, so this is what you'll actually have to deal with indefinitely. I also have a set which is temporary and can fix.

## Lightning < Bitcoin

First of all, how do we allocate funds? Fund allocation to channels is a hard problem. We basically have to foresee the future and say "hey, I'll in a week's time I will be talking to this guy and I'd like to have a channel with them", and as predictions always go they turn out to be completely bullshit. So what we try to do is basically encourage people to actually open channels at random to create as cohesive as possible of a network so if we have a network in which everyone is well connected to everybody else, we can actually route payments rather well. As compared to everybody's connected to Starbucks and Starbucks is this island and to talk to Walmart I have to open new channels. So if you can try to create bridges between these systems, but I also think that these bridges will automatically emerge once we have a working fee market simply because it will be a positive thing for people to earn some fees on the side. Maybe not as a form of income but to offset your own use of lightning. I don't think that routing will be a good business. It might pay for your use of the Lightning Network. I get 10 satoshis in fees and now I can spend like 10 satoshis on routing my own payments.

### Routing

Speaking of routing, routing can be difficult. We built Lightning as private as possible and for this we had to remove some of the information that we might have wanted to share. As Elaine showed yesterday, we don't communicate what the current balances of the endpoints are, we communicate what the total capacity of the channel is. But if there is $10 on this side, I have no idea whether I can transfer $1 in this direction or maybe $9 in this direction, we just have to try and see if it fails. If we were to publish that information we could basically see "okay there's this channel has a reduced capacity by 13 satoshis, this has a reduced capacity by 13 satoshis, this has a reduced capacity by 13 satoshis", guess where the payment went? And even if we were to communicate that information, it would be a massive amount of data, right? This would basically be a broadcast from every channel for every path going to every node in the network.

[Audience question]: compromise... broadcast to my neighbors...

CD: Well I mean, every potential user of your channel has to know your policy about forwarding payments, right? So basically every single node that might want to use your channel needs to be informed about the parameters of your channel. We have indirect signaling. So what we can do is disable individual directions of a channel, we can set fees to basically infinity so that people don't use that channel, and we can set fees to zero to basically say "hey, if you use this channel that's free." But we still have this reliance on information that we don't have direct access to and we're also relying on peers that we ourselves don't control. One odd detail is that we actually need to be online to receive so this is an interactive process when receiving a payment you need to actually acknowledge your receipt of the payment. That's a step backward from Bitcoin where we could just publish an address and just go away. In this case, we have to acknowledge your receipt and it also means that our funds are in hot wallets. So be sure not to keep your college funds in lightning channels, don't keep your pension in lightning channels. You're not going to make money off of this, this is your daily change with which you're going to buy coffee, maybe the occasional grocery shopping. Don't put your life savings in this. I've been hit a few times in my time with Bitcoin for considerable amounts, do not keep your funds in hot wallets, just don't. And exchanges are counted as hot wallets, by the way.

Somebody mentioned yesterday, but it's worth reiterating, payments are fast if they're fast but sometimes fast is slow. So if there is a channel that is non-responsive after it has added the HTLC. So it basically said "hey, yeah I'm going to forward this", he forwards this or doesn't and then he disappears. There's no way for us to say "hey will this resolve, will this actually be paid or might this come back and we need to retry". So sometimes payments will get stuck and these stuck payments are even slower than onchain payments because they have timeouts attached. So whenever this happens, try to make a refund. So somebody sent you payment, he's saying "hey, I don't know if that if that worked or not", what you can do is basically you can send them a payment back for the same payment hash such that if the payment succeeded they can get the payment back. If the payment did not succeed they cannot get the payment back. It's still going to be slow and funds will be locked up but it allows you to have a retry immediately without having this. Now we don't support refunds automatically just yet or I don't think any of the clients actually has the facilities to do so. But it is always nice to say "hey you had trouble paying me, if this payment goes through I will refund you no problem, just try a different route." That way it's complicated, it's nasty, but you can actually go around these slow payments. If you don't do that, you may have locked up the user's funds for weeks, which is going to happen anyways, but you'll also not have the ability to have retry stuff.

[audience question]: isn't that insecure in the sense that if I want to pay you a certain route the payment hash and you make the inverse payment hash, there's a route in the between that...

Yeah, if the return path is not disjoint that's indeed a problem but that's going to be solved with Schnorr hopefully. So we have payment decorrelation stuff in the works that will allow us to give these refunds, it's also not implemented to do these refunds currently because we need some additional tech for it. So currently you have to wait but once we have implemented them, do refunds.

[audience]: ...having the invoice with expiration, does it solve that?

No, so the invoice expiry is only on the sort of human side of things. The expiry is not expressed in the off-chain contracts at all. It basically is just "hey I expect you to initiate payment in half an hour" that might go through or it might not go through. But clients will refuse to initiate payment if this expiry already has expired but it still means that payments might get stuck. So the expiry time is nice to communicate it to users, "hey, I expect you to initiate payment at this time", it might not finish. These stuck payments are nasty and really hard to work around.

And finally one thing that a lot of people have run into when people have tested stuff is basically they open a channel, try to send the entire capacity over like I opened for 1000 satoshis and want to push 1000 satoshis over - that does not work. The reason is we have a reserve value that we don't allow people to go below because the penalty mechanism in Lightning actually requires you to have a stake in the outcome of the channel and if I don't have any money in the channel, well I can just try for free and if I'm the recipient of the channel then I don't even have to pay fees for them which is different from Eltoo where the initiator of the closure has to pay the fees. So if you open a channel for 1000 satoshis and try to push those to send over 1000 satoshis that is going to fail because 10 satoshis will need to stay on the sender's side. The same is true for refunds. It's been really fun for the blockstream sticker store to try to do refunds when the only payment people did was basically pay them. So I have 1000 satoshis on my channel, I use 100 of them to pay the lightning store, it doesn't work, I want my money back, but I can only push 19 satoshis back because blockstream is now forced to (the receiving side) is now forced to keep above 1% of the channel capacity. Try to explain that to users. It's a really nasty detail that has hurt quite a lot of people before and it's kind of hard to work around.

[audience question]

The fees in the settlement transactions for lightning channels are always paid by the funder, currently, because it's easy, we're sure that they have funds in the channel. If they spent it all they still have the reserve to pay fees and we make sure that these fees stay above the level at which both sides of the channel are comfortable with. Comfortable with meaning that I'm sure that I could confirm this transaction in a matter of two hours to maybe a day. If that's not the case we will close the channel just to sort of bail out in time for the transaction fees to be high enough and that's currently the reason why we closed like 90% of our channels is because one guy says "hey my fee is 1000 satoshis per byte" and the other one says "You're crazy it's like 10000." So if we disagree on fees we might actually bail out even if not needed. We're working on smoothing those fee estimates just to be sure that we don't have one guy seeing a block and the other one not seeing a block and I see this block and make my estimate based on that and you just have this huge mempool that you don't know when it's going to be clear. Fee smoothing has solved that problem mostly. In future iterations, we will have fee estimations at the time we spend and paid by the ones that closed the channel.

[audience question] ...

I'm trying to remove the penalty altogether because we now have a much more lightweight way of punishing people. Namely they have to pay the on chain fees for their attempt to close and if we're clever about which transaction we double spend in or to which transaction we attach our own update transaction, we can actually still make you pay fees despite you not getting your desired outcome. It's much lower penalty but I don't think that the majority of people will actually try to cheat. People always concentrate on cheaters,  but in my experience most of the cheat attempts are actually people that sort of just did something wrong and with no bad intention. So I think that paying a fee as a penalty is much nicer but still gets the end result of “yeah you don't try that.”


### Routing is hard

So just to illustrate why routing is hard. If we have two nodes and there's no connection between them you're not going to send stuff. I think that's obvious but I've had so many people connecting two nodes and not funding a channel and then saying "hey this doesn't work it's broken." The same goes for if you have a channel and the capacity is all on one side, you're not going to send from right to left because the right node does not have any balance in this channel. And more interestingly is the forwarding case in which you have like three nodes and you could send five from the left on to the middle one but only one to the right one so your maximum amount that you can send from here to here is one bitcoin minus reserve value. So we actually have this bottleneck issue in Lightning and so try to make the payment size of your systems sort of match the amounts of common channels in the network. Currently, everybody is opening channels that have like 20 cents. I spent like 4 hours debugging channels which did not return any funds and then I finally took a look at what the channel amount was and it was 19 cents. At which point it was clear hey "everything went to fees, sorry".

## The Ugly but temporary

The ugly but temporary. This is basically a slide that I stole from one of our internal presentations, it's what's coming up in November during the spec meeting. On the left side we have some really nice improvements, we have splice in and splice out which allows you to add and remove payments, and we have multipath routing which basically means that if I have a channel of 5 and I have a channel of 4, I can actually bumble the capacity of these channels if later the paths merge again. And I can use the entirety of my funds. What this allows you to do is basically have a wallet that is really nice because it shows you one balance and it shows you what happened to that balance over time. No indication of channels, no need to actually take any decision about how to allocate funds because well you can just bundle them. No need to differentiate between on chain and off chain because you can do on-chain payments with a splice out. So I can have a channel, temporarily close that, splice out some funds, and reopen the channel without interruption. And what might be interesting for you is we can have spontaneous and streamed payments, so those are payments that are not bound to an invoice. That means that I can pay per second for all the stuff, for my bandwidth, but it also means that I can do donations, which Rene has a cool video about how to do it anyways currently but they won't get notified currently that they got a payment. We have payment decorrelation which basically hides the connection between individual hops of the payments. So if you are twice in a row you won't be able to say "hey, this belongs to the same payment".

Dual funded channels is a big one, when you want to receive payments it's always hard currently because you open the channel, you're the only one that has balance, so you have to push out some funds before you're able to receive. Having both parties add funds to the channel solve that sort of. If you can convince the other guy to commit funds to your channel. Fee hooks, we will do away with the fee changes and the fee rate estimates such that we can actually push the fee rate estimate to the point where we need it and hopefully avoid having channels close over them.

Watchtower protocol, I guess most of you have heard this, we can outsource the punishment of a misbehaving node to some third party. We're still unclear on how to reward them for their services. But if it's your peers that are watching your other peers, that might be a bit of a tit for tat just to keep yourself secure. And what I'm looking forward to is that we might want to forward Bitcoin information on the Lightning Network such that you yourself get notified whenever channels open, whenever channels close, and also get the associated information such as the merkle proof that shows you this actually happened on chain or "look there's this channel closing maybe you don’t want to use that in the future" and instead of having to use bitcoind as your only source of truth to the bitcoin network. You can actually have this push mechanism that allows that.

audience

Yes. Yeah it basically is just a way for you to advertise your node on your website and get donations basically. That's a use case. You're not getting your receipt in that case. Whereas stream payments would allow you to issue an invoice once, say "I'd like to have a payment every 5 seconds from you for such and such an amount and I will give you a receipt for every single one of them". So you can actually prove that you paid.

audience ...

Basically have a repeatable invoice.

audience ... additional fees.

It will have additional fees.. Well you pay for transfer. So you basically have the base fee which is leveraged each time but that should be new anyway.

audience ... topology of the network ...

You are the expert on autopilot. And it's definitely something that we want to look into. I'm a bit sceptic about having autopilot communicate because as soon as they communicate you sort of trust what they're saying. Otherwise you could do away with them with the communication altogether so I'd like to explore first of all having autopilots that are based on local information only or verifiable information only and only then extend that to have them communicate among themselves.

[audience] ... are you aware of ... lightning node in a way that ... generate ...

So payments are fundamentally always going to create a commitment transaction. That needs to be signed.

[audience] ...

It sort of is because the invoices that we give out are basically signed by us and are a promise to deliver something in exchange for payment. The receipt that we get in combination with the invoice can be used to prove to somebody "hey, we paid" so your vending machine might not be the device that is receiving the payment, but you can still prove to it "hey this is by your owner, he promised you'd give me this, here is the proof that I paid."

[audience] ... do you provide an API to be able to set up 10000 coke vending machines ...

Oh yeah, I mean we can definitely do that, but it would need to be described in a way that we can actually put it down in a spec.

Thanks!

[Applause]