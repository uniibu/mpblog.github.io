---
title: "Cold Addresses: MoneyPot's Privacy Service"
layout: post
permalink: "/cold-addresses"
---

![cold address demo modal](/img/post-assets/cold-addresses/demo-modal.png)

MoneyPot's cold address feature provides a layer of privacy on top of bitcoin. Cold addresses allow you to safely deposit or receive money from people and services without revealing that you are depositing to MoneyPot nor what you go on to do with that money in future transactions.

But to understand how it offers you privacy, we must establish how the rest of MoneyPot works.

## Normal Deposits

Normal deposit addresses in MoneyPot are associated with our hot wallet. The hot wallet is, in effect, a large shared pool of coins. It is what we use to immediately fund withdrawals from our system. If the hot wallet grows large enough, the surplus is skimmed off into various storage addresses for security.

Because of this, you will typically find that after depositing to MoneyPot, the coins are rapidly moved and mixed with the coins of other users.

While not trivial to do, people studying the MoneyPot hot wallet system are able to associate various addresses with MoneyPot and monitor various hot wallet movements. One possible way for people to do this is by making deposits and withdrawals themselves, posing as a normal users, to see exactly where the coins go and what they are spend-linked with.

Some large services and exchanges have been known to monitor what you use your coins for in order to comply with local laws. To protect the privacy of our users against such analysis, we have developed **cold addresses**.

## Cold Address Deposits

Unlike a normal deposit address which deposits into our hot wallet, a cold address deposits into our offline cold storage. This means when you deposit to a cold address, you will not see the money move from the address as we process withdrawals. All the while, your MoneyPot account is credited as normal and you are free to withdraw or gamble. 

Withdrawals are processed from our hot wallet like normal, but because the cold address belongs to our offline cold storage, there is absolutely no information leak, and the address is completely indistinguishable from any other address except that the money doesn't move as bitcoin churns within our system. It is quite literally a total dead-end for any blockchain analysis.

This also has one huge advantage over traditional mixing services: the coins do not look mixed. They just look like you sent it to a place that hasn't (yet) used them.

## Spending from cold storage

Eventually (normally several months later), MoneyPot will need to get access to the money from our cold storage, but how we handle this is extremely important. For any amount greater than 10,000 bits (0.01 bitcoin) they will *never be spend-linked* with any other cold-address or address that has been associated with MoneyPot at all. All funds will carefully be sent through external services before funneled back into MoneyPot.

## How generate cold addresses

If you require such privacy for your deposits, you can generate cold addresses by simply checking the "Cold Address" checkbox on the new address form of the [Receive](https://www.moneypot.com/me/receive) tab (must be logged in to MoneyPot). 

Cold addresses stand out from regular addresses with a special blue "Cold" indicator.

![normal vs cold address glyph](/img/post-assets/cold-addresses/cold-glyph.png)

We just charge a 1% fee on deposits into cold addresses to pay for this service.
