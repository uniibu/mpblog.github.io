---
title: "Cold Addresses: Moneypot's Privacy Service"
layout: post
permalink: "/cold-addresses"
---

![cold address demo modal](/img/post-assets/cold-addresses/demo-modal.png)

Moneypot's cold address feature provides a layer of privacy on top of bitcoin. Cold addresses allow you to safely deposit, or receive money from people or services without revealing that you are depositing to moneypot, or what you go on to do with the money.

But to understand how it offers you privacy, we must establish how the rest of Moneypot works.

## Normal Deposits

Normal deposits address in Moneypot are associated with our hot wallet. The hot wallet is what we use to immediately process any withdrawals, whether it is from yourself or another MoneyPot user, in effect a big shared pool of coins. If the hot wallet grows large enough, excess is swept off to various storage addresses for security. Because of this, you will typically find after depositing to Moneypot the coins are rapidly moved, and mixed with other users coins.

While not trivial to do, people studying the MoneyPot system are able to associate various addresses with MoneyPot and monitor various hot wallet movements. One possible way for people to do this is by making deposits and withdrawals themselves, posing as a normal
users to see exactly where the coins go, and what they are spend-linked with.

Some large services and exchanges have been known to monitor what you use your coins for, in order to comply with local laws. To protect user privacy against such intrusions we have developed cold addresses.

## Cold Address Deposits

Unlike a normal deposit address, the cold address belongs to our offline cold storage. This means when you deposit to the address you will not see the money move. All the while, your Moneypot account is credit like normal, and you are free to withdraw or gamble. Withdrawals are processed from our hot wallet like normal, but because the cold address belongs to our offline cold storage, there is absolutely no information leak, and the address is completely indistinguishable from any other address in which the money doesn't move. It is quite literally a total dead-end for any blockchain analysis.

This also has one huge advantage over traditional mixing services, the coins do not look mixed. They just look like you sent it to a place that hasn't (yet) used them.


## Spending from cold storage

Eventually (normally several months later) Moneypot will need to get access to the money from our cold storage, but how we handle this is extremely important. For any amount greater than 10,000 bits (0.01 bitcoins) they will *never be spend-linked* with any other cold-address or address that has been associated with Moneypot at all. All funds will carefully be sent through external services, before funneled back into Moneypot.


