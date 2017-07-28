# AnonWork: The Bitcoin Anonymity Framework

## Authors

Ádám Ficsór ([nopara73](https://github.com/nopara73)),  
[TumbleBit](https://github.com/NTumbleBit/NTumbleBit), [HiddenWallet](https://github.com/nopara73/HiddenWallet), [DotNetTor](https://github.com/nopara73/DotNetTor)  
adam.ficsor73@gmail.com

## Donating

186n7me3QKajQZJnUsVsezVhVrSwyFCCZ  
[![QR Code](http://i.imgur.com/grc5fBP.png)](https://www.smartbit.com.au/address/186n7me3QKajQZJnUsVsezVhVrSwyFCCZ)

## Abstract

[Fungibility](https://en.wikipedia.org/wiki/Fungibility) is an essential property of good money. Since the invention of [Bitcoin](https://bitcoin.org/bitcoin.pdf) in 2008 countless fungibility improvements have been proposed, implemented, deployed and used. Some of them builds on top of Bitcoin, some of them invented their own cryptocurrencies to address the privacy issue. As of today, 2017 no proposal, including privacy centric altcoins addressed the problem in its full scale. No proposal attempted to provide protections against all the different ways a user's privacy can be breached. As a result, we are still not able to transact in a fully anonymous way. The Bitcoin Anonymity Framework is the first attempt to do that. It evaluates all the research and techniques, known to date and combines them in a practical and coherent way.  
The scope of AnonWork is limited to Bitcoin's first layer. As long as the Bitcoin network is being used, there will always be need for on-chain privacy. Even if an off-chain privacy solution, like the [TumbleBit Payment Hub](https://eprint.iacr.org/2016/575.pdf) gets deployed and adopted, the entrance and exit transactions will always happen on-chain.  
**The main goal of AnonWork is to make a set of coins unlinkable to another set of coins.**  
Furthermore the authors are also committed to build and deploy a production ready implementation, therefore not get stucked in the research phase.  

## Table Of Contents

I. [Introduction](#i-introduction)  
II. [Chaumain CoinJoin](#ii-chaumain-coinjoin)  
&nbsp;&nbsp;&nbsp;A. [Protocol](#a-protocol)  
&nbsp;&nbsp;&nbsp;C. [Attacks](#b-attacks)  
III. [Wallet Privacy Framework](#iii-wallet-privacy-framework)  
&nbsp;&nbsp;&nbsp;A. [Pre-Mix Wallet](#a-pre-mix-wallet)  
&nbsp;&nbsp;&nbsp;B. [Post-Mix Wallet](#b-post-mix-wallet)  
IV. [Conclusions](#iv-conclusions)  

## I. Introduction

### CoinJoin

The idea of [CoinJoin](https://bitcointalk.org/index.php?topic=279249.0) (CJ) was introduced in 2013 by Gregory Maxwell. When multiple participants add inputs to a common CJ transaction and some of the outputs have the same value no one can tell which input intended to fund which of these outputs.  
CoinJoin based techniques are generally the most Blockchain space efficient, therefore the cheapest on-chain Bitcoin privacy techniques. The maximum reachable theoretical anonymity set goes [from 350 to 470](https://bitcoin.stackexchange.com/questions/57073/what-is-the-maximum-anonimity-set-of-a-coinjoin-transaction/57091).

### Chaumain CoinJoin

Chaumain CoinJoin was also briefly described by Maxwell:  
> Using chaum blind signatures: The users connect and provide inputs (and change addresses) and a cryptographically-blinded version of the address they want their private coins to go to; the server signs the tokens and returns them. The users anonymously reconnect, unblind their output addresses, and return them to the server. The server can see that all the outputs were signed by it and so all the outputs had to come from valid participants. Later people reconnect and sign.  

Every mix made via Chaumain CoinJoin comes with a guarantee that Tumbler can neither violate anonymity, nor steal bitcoins. Chaumain CoinJoin is by no means complex. Its simplicity allows it to be one of the most, if not the most performant on-chain mixing technique. A mixing round can be measured in seconds or minutes. 

### Distributed CoinJoin

It is certainly possible to distribute this scheme. Tim Ruffing's [CoinShuffle](http://crypsys.mmci.uni-saarland.de/projects/CoinShuffle/coinshuffle.pdf) and [CoinShuffle++](https://crypsys.mmci.uni-saarland.de/projects/FastDC/draft-paper.pdf) are novel attempts to do so. However distributed systems are hard to get right: they require various tradeoffs, they are more complex, they open the door for various attacks, updating or upgrading them are difficult. The implementation of Chaumain CoinJoin is fairly straightforward, therefore existing wallets can easily implement it. The server is untrusted, so it does not have the risk of coin stealing, nor the risk of privacy breaching, therefore distributing this system might not be fully justified from a practical point of view.  
As Maxwell noted:  
>  I don't know if there is, or ever would be, a reason to bother with a fully distributed version with full privacy, but it's certainly possible.  

Of course distributed systems are more resilient, therefore distribution should certainly be an interest of future research.  

### P2P anonymous communication protocols

As Maxwell noted:  
> Any transaction privacy system that hopes to hide user's addresses should start with some kind of anonymity network. This is no different. Fortunately networks like Tor, I2P, Bitmessage, and Freenet all already exist and could all be used for this. (Freenet would result in rather slow transactions, however)  

Another advantage of CoinShuffle++ is that it does not require such anonimity network as an external dependency, rather it implements its own P2P mixing protocol, DiceMix:  
>  We conceptualize these P2P anonymous communication protocols as P2P mixing, and present a novel P2P mixing protocol, DiceMix, that needs only four communication rounds in the best case, and 4 + 2f rounds in the worst case with f malicious peers.  

AnonWork requires such P2P anonymous protocols at mixing and at transaction broadcasting. [Tor](https://www.torproject.org/) is the most widely deployed such protocol, therefore AnonWork chooses to rely on it. Note also an AnonWork compliant application should not use a Tor proxy to the clearnet, but rather stay inside the Tor network and constrain its communication with [hidden services](https://www.torproject.org/docs/hidden-services.html.en). This is to protect against of various [attacks](https://en.wikipedia.org/wiki/Tor_(anonymity_network)#Weaknesses).  

Elimination of Tor dependency should be an interest of future research.  
 
### Privacy Is Teamwork

The theoretical anonymity set of a mixing technique is misleading. As an example it matters little what tracks the user leaves on the Blockchain if, due to the architechture of its wallet, a central server is already aware of all its wallet addresses, so it knows the link between an input and an output of a mix. 
If one user of the mix gets deanonymized the real anonymity set of the rest of the users gets lower, too, therefore it is not acceptable that a set of users are using a mixing technique in a flawed way.

### Transactions And Transaction Chains

Any Bitcoin mixing techique must use a common denomination, otherwise simple amount analysis can re-estabilish the links, as Kristof Atlas did in his [CoinJoin Sudoku](www.coinjoinsudoku.com).  
This notion leads to multiple rounds. For example if a user wants to mix 8 bitcoins and the mixing denomination is 1 bitcoin, then it must use 8 mixing rounds.  
As Mike Hearn [put it](https://groups.google.com/forum/#!msg/bitcoinj/Ys13qkTwcNg/9qxnhwnkeoIJ):
> The problem starts when we realise that what we actually care about is not transactions but rather transaction chains.  

For example if our post-mix wallet would function as a normal Bitcoin wallet, too, the observer would notice post-mix transactions, those are joining together mixed outputs. In this case the real anonymity set of all the users who participated in the same mixes would suffer.  

If [Confidential Transactions](https://elementsproject.org/elements/confidential-transactions/) are introduced to Bitcoin in the future, it could potentially solve the "denomination issue".

### Theoretical And Real Anonymity Set

We refer to theoretical anonymity set as the anonymity set that is achieved by a bitcoin mixing technique within one round and does not weigh in in external factors, like flawed wallet architechture.  
We refer to real anonymity set when these these external factors are weighted in and transaction chains are analyzed.  

### Alternative On-Chain Mixing

TumbleBit Classic Tumbler mode and [CoinSwap](https://bitcointalk.org/index.php?topic=321228.0) are not CoinJoin based. They are both multiple times more expensive and slower than Chaumain CoinJoin. For example while TumbleBit requires 4 transactions, therefore 4 times transaction fees, CoinJoin requires only 1. While TumbleBit requires hours to complete a round, CoinJoin minutes.  
CoinShuffle and [JoinMarket](https://github.com/JoinMarket-Org/joinmarket) are CoinJoin based techniques. We detailed the former previously, so we omit to discuss it here.  
JoinMarket introduced a novel maker-taker concept, where market makers are waiting until a taker wants to execute a CJ transaction and asks market-makers to provide liquidity for his CoinJoin for a small fee. This of course gets expensive for bigger anonimity sets and it rather achieves plausability, than unlinkability, because how the makers use their coins after the mix will noticeably differ from the takers' mixed coins.  

When in the future [Schnorr signatures](https://www.elementsproject.org/elements/schnorr-signatures/) are introduced to Bitcoin, CoinJoin based techniques will get even more Blockchain space efficient, therefore cheaper.

Find more detailed comparisons [here](https://medium.com/@nopara73/tumblebit-vs-coinjoin-15e5a7d58e3).  

### RFC2119

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" in this document are to be interpreted as described in [RFC2119](https://tools.ietf.org/html/rfc2119).

## II. Chaumain CoinJoin

### A. Protocol

### B. Attacks

## III. Wallet Privacy Framework

### A. Pre-Mix Wallet

### B. Post-Mix Wallet

## IV. Conclusions
