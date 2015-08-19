---
title: "Cold Addresses: Moneypot's Mixing Service"
layout: post
permalink: "/cold-addresses"
---

![cold address demo modal](/img/post-assets/cold-addresses/demo-modal.png)

Moneypot's cold address feature provides a layer of privacy on top of Bitcoin.

But to understand how it offers you privacy, we must establish how the rest of Moneypot works.

## Moneypot's hot wallet and cold wallet

Whenever a Moneypot user sends money to a Bitcoin address (i.e. withdraws from their Moneypot account), Moneypot sends the funds from its hot wallet. A hot wallet is basically the pool of bitcoin that Moneypot has direct access to at a given moment.

If the hot wallet is depleted by a big withdrawal or if a user tries to withdraw more money than is in the hot wallet, then Moneypot staff must refill the hot wallet by moving funds into it from the cold wallet.

For security reasons, the hot wallet holds as few bitcoin as necessary to fund the day-to-day needs of Moneypot. The rest of Moneypot's funds are stored offline in a cold address.

This means that the hot wallet bitcoin churns much more rapidly than the funds in the cold address.

## How hot addresses work

When you generate a Bitcoin address from your Moneypot account without selecting the "Cold Address?" checkbox (let's call this a "Hot Address"), then any Bitcoin you deposit to that address ends up in Moneypot's hot wallet.

In other words, by default, deposits go straight into Moneypot's hot wallet where they will fund withdrawals.

When you deposit 1 bitcoin into Moneypot's hot wallet, Moneypot credits you +1 bitcoin and then dumps your bitcoin into its hot wallet where it might use that bitcoin to fund any number of withdrawals for other users.

Whenever you withdraw bitcoin from Moneypot, Moneypot funds the transaction with any bitcoin in its hot wallet. It might fund the withdrawal with bitcoin deposited by 15 users or it might fund the withdrawal with the bitcoin that you just deposited (unlikely).

TODO: Explain how taint analysis can link your bitcoin and then introduce a "How cold addresses work" section that explains how cold addresses address this issue
