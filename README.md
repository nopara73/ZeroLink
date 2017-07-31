# ZeroLink: The Bitcoin Fungibility Framework

## Authors

Ádám Ficsór (nopara73),  
[TumbleBit](https://github.com/NTumbleBit/NTumbleBit), [HiddenWallet](https://github.com/nopara73/HiddenWallet)  
adam.ficsor73@gmail.com

### Reviewers

TDevD,  
[SamouraiWallet](https://github.com/Samourai-Wallet)

## Donating

186n7me3QKajQZJnUsVsezVhVrSwyFCCZ  
[![QR Code](http://i.imgur.com/grc5fBP.png)](https://www.smartbit.com.au/address/186n7me3QKajQZJnUsVsezVhVrSwyFCCZ)

## Abstract

[Fungibility](https://en.wikipedia.org/wiki/Fungibility) is an essential property of good money. Since the invention of [Bitcoin](https://bitcoin.org/bitcoin.pdf) in 2008 countless fungibility improvements have been proposed, implemented, deployed and used. Some of them builds on top of Bitcoin, some of them invented their own cryptocurrencies to address the privacy issue. As of today, 2017 no proposal, including privacy centric altcoins addressed the problem in its full scale. No proposal attempted to provide protections against all the different ways a user's privacy can be breached. As a result, we are still not able to transact in a fully anonymous way. The Bitcoin Fungibility Framework is the first attempt to do that. It evaluates all the research and techniques, known to date and combines them in a practical and coherent way.  
The scope of ZeroLink is limited to Bitcoin's first layer. As long as the Bitcoin network is being used, there will always be need for on-chain privacy. Even if an off-chain privacy solution, like the [TumbleBit Payment Hub](https://eprint.iacr.org/2016/575.pdf) gets deployed and adopted, the entrance and exit transactions will always happen on-chain.  
**The main goal of ZeroLink is to make a set of coins unlinkable to another set of coins.**  
Furthermore the authors are also committed to build and deploy a production ready implementation, therefore not get stuck in the research phase.  

## Table Of Contents

I. [Introduction](#i-introduction)  
II. [Chaumian CoinJoin](#ii-chaumian-coinjoin)  
&nbsp;&nbsp;&nbsp;A. [Simplified Protocol](#a-simplified-protocol)  
&nbsp;&nbsp;&nbsp;B. [Issues](#b-issues)  
&nbsp;&nbsp;&nbsp;C. [Full Protocol](#c-full-protocol)  
III. [Wallet Privacy Framework](#iii-wallet-privacy-framework)  
&nbsp;&nbsp;&nbsp;A. [Pre-Mix Wallet](#a-pre-mix-wallet)  
&nbsp;&nbsp;&nbsp;B. [Post-Mix Wallet](#b-post-mix-wallet)  
IV. [Conclusions](#iv-conclusions)  

## I. Introduction

### CoinJoin

The idea of [CoinJoin](https://bitcointalk.org/index.php?topic=279249.0) (CJ) was introduced in 2013 by Gregory Maxwell. When multiple participants add inputs to a common CJ transaction and some of the outputs have the same value no one can tell which input intended to fund which of these outputs.  
CoinJoin based techniques are generally the most Blockchain space efficient, therefore the cheapest on-chain Bitcoin privacy techniques.  
There is no limit for the maximum theoretical anonimity set. While a natural limiting factor would be the maximum standard transaction size, in which case it goes [from 350 to 470](https://bitcoin.stackexchange.com/questions/57073/what-is-the-maximum-anonimity-set-of-a-coinjoin-transaction/57091), it can be surpassed, as Maxwell notes:  
> If you can build transactions with m participants per transaction you can create a sequence of m*3 transactions which form a three-stage [switching network](https://en.wikipedia.org/wiki/Clos_network) that permits any of m^2 final outputs to have come from any of m^2 original inputs (e.g. using three stages of 32 transactions with 32 inputs each 1024 users can be joined with a total of 96 transactions).  This allows the anonymity set to be any size, limited only by participation.

For practical reasons, this document will not attempt to incorporate such switching network into its design, it rather lets the implementator to scale up if the need ever arises.

### Chaumian CoinJoin

Chaumian CoinJoin was also briefly described by Maxwell:  
> Using chaum blind signatures: The users connect and provide inputs (and change addresses) and a cryptographically-blinded version of the address they want their private coins to go to; the server signs the tokens and returns them. The users anonymously reconnect, unblind their output addresses, and return them to the server. The server can see that all the outputs were signed by it and so all the outputs had to come from valid participants. Later people reconnect and sign.  

Every mix made via Chaumian CoinJoin comes with a guarantee that Tumbler can neither violate anonymity, nor steal bitcoins. Chaumian CoinJoin is by no means complex. Its simplicity allows it to be one of the most, if not the most performant on-chain mixing technique. A mixing round can be measured in seconds or minutes. 

### Distributed CoinJoin

It is certainly possible to distribute this scheme. Tim Ruffing's [CoinShuffle](http://crypsys.mmci.uni-saarland.de/projects/CoinShuffle/coinshuffle.pdf) and [CoinShuffle++](https://crypsys.mmci.uni-saarland.de/projects/FastDC/draft-paper.pdf) are novel attempts to do so. However distributed systems are hard to get right: they require various tradeoffs, they are more complex, they open the door for various attacks, updating or upgrading them are difficult. The implementation of Chaumian CoinJoin is fairly straightforward, therefore existing wallets can easily implement it. The Tumbler is untrusted, so it does not have the risk of coin stealing, nor the risk of privacy breaching, therefore distributing this system might not be fully justified from a practical point of view.  
As Maxwell noted:  
>  I don't know if there is, or ever would be, a reason to bother with a fully distributed version with full privacy, but it's certainly possible.  

Of course distributed systems are more resilient, therefore distribution should certainly be an interest of future research.  

### P2P anonymous communication protocols

As Maxwell noted:  
> Any transaction privacy system that hopes to hide user's addresses should start with some kind of anonymity network. This is no different. Fortunately networks like Tor, I2P, Bitmessage, and Freenet all already exist and could all be used for this. (Freenet would result in rather slow transactions, however)  

Another advantage of CoinShuffle++ is that it does not require such anonymity network as an external dependency, rather it implements its own P2P mixing protocol, DiceMix:  
>  We conceptualize these P2P anonymous communication protocols as P2P mixing, and present a novel P2P mixing protocol, DiceMix, that needs only four communication rounds in the best case, and 4 + 2f rounds in the worst case with f malicious peers.  

ZeroLink requires such P2P anonymous protocols at mixing and at transaction broadcasting. [Tor](https://www.torproject.org/) is the most widely deployed such protocol, therefore ZeroLink chooses to rely on it. Note also an ZeroLink compliant application should not use a Tor proxy to the clearnet, but rather stay inside the Tor network and constrain its communication with [hidden services](https://www.torproject.org/docs/hidden-services.html.en). This is to protect against of various [attacks](https://en.wikipedia.org/wiki/Tor_(anonymity_network)#Weaknesses).  

Elimination of Tor dependency should be an interest of future research.  
 
### Privacy Is Teamwork

The theoretical anonymity set of a mixing technique is misleading. As an example it matters little what tracks the user leaves on the Blockchain if, due to the architechture of its wallet, a central server is already aware of all its wallet addresses, so it knows the link between an input and an output of a mix. 
If one user of the mix gets deanonymized the real anonymity set of the rest of the users gets lower, too, therefore it is not acceptable that a set of users are using a mixing technique in a flawed way.

### Transactions And Transaction Chains

Any Bitcoin mixing techique must use a common denomination, otherwise simple amount analysis can re-estabilish the links, as Kristov Atlas did in his [CoinJoin Sudoku](http://www.coinjoinsudoku.com) analysis of Blockchain.info's [SharedCoin](https://github.com/sharedcoin/Sharedcoin) (service "temporarily" cancelled due to awareness of its privacy limitations and reports of stuck transactions.)  
This notion leads to multiple rounds. For example if a user wants to mix 8 bitcoins and the mixing denomination is 1 bitcoin, then it must use 8 mixing rounds.  
As Mike Hearn [put it](https://groups.google.com/forum/#!msg/bitcoinj/Ys13qkTwcNg/9qxnhwnkeoIJ):
> The problem starts when we realise that what we actually care about is not transactions but rather transaction chains.  

For example if our post-mix wallet would function as a normal Bitcoin wallet, too, the observer would notice post-mix transactions, those are joining together mixed outputs. In this case the real anonymity set of all the users who participated in the same mixes would suffer.  

If [Confidential Transactions](https://elementsproject.org/elements/confidential-transactions/) are introduced to Bitcoin in the future, it could potentially solve the "denomination issue".

### Theoretical And Real Anonymity Set

We refer to theoretical anonymity set as the anonymity set that is achieved by a bitcoin mixing technique within one round and does not weigh in in external factors, like flawed wallet architechture.  
We refer to real anonymity set when these these external factors are weighted in and transaction chains are analyzed.  

### Alternative On-Chain Mixing

TumbleBit Classic Tumbler mode and [CoinSwap](https://bitcointalk.org/index.php?topic=321228.0) are not CoinJoin based. They are both multiple times more expensive and slower than Chaumian CoinJoin. For example while TumbleBit requires 4 transactions, therefore 4 times transaction fees, CoinJoin requires only 1. While TumbleBit requires hours to complete a round, CoinJoin minutes.  
CoinShuffle and [JoinMarket](https://github.com/JoinMarket-Org/joinmarket) are CoinJoin based techniques. We detailed the former previously, so we omit to discuss it here.  
JoinMarket introduced a novel maker-taker concept, where market makers are waiting until a taker wants to execute a CJ transaction and asks market-makers to provide liquidity for his CoinJoin for a small fee. This of course gets expensive for bigger anonymity sets and it rather achieves plausability, than unlinkability, because how the makers use their coins after the mix will noticeably differ from the takers' mixed coins.  
Finally there's also [ValueShuffle](https://eprint.iacr.org/2017/238.pdf) which is still a CoinJoin based technique, however it requires the not yet deployed Confidential Transacions.  

When in the future [Schnorr signatures](https://www.elementsproject.org/elements/schnorr-signatures/) are introduced to Bitcoin, CoinJoin based techniques will get even more Blockchain space efficient, therefore cheaper.

Find more detailed comparisons [here](https://medium.com/@nopara73/tumblebit-vs-coinjoin-15e5a7d58e3).  

### RFC2119

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" in this document are to be interpreted as described in [RFC2119](https://tools.ietf.org/html/rfc2119).

## II. Chaumian CoinJoin

### A. Simplified Protocol

Alice and Bob are same user, but the Tumbler does not know about that.  

![Chaumian CoinJoin](http://i.imgur.com/eVUVM6i.png)

#### 1. Input Registration Phase

Many Alices register their 
 - utxos as the inputs of the CoinJoin,
 - proofs that they can spend those utxos (signed messages with the corresponding private keys),
 - their desired change outputs,
 - and blinded outputs to the Tumbler.

Tumbler checks if inputs have enough coins, are unspent and proofs are valid, then sign the blinded output.  
Alices unblind their signed and blinded outputs.

#### 2. Output Registration Phase

Bobs registers their signed outputs to the Tumbler.

#### 3. Signing Phase

Tumbler builds the unsigned CoinJoin transaction and gives it to Alices for signing.  
When all the Alices signed arrive, the Tumbler combines the signatures and propagates the CoinJoin on the network.

### B. Issues

#### DoS attacks

There are various ways malicious users can paralyze a round. The Tumbler as new attacks are being executed it MUST adopt and implement protections against. However there are some obvious DoS attacks we address and recommend countermeasures.

##### DoS 1: What if an Alice spends her input immaturely?

If it happens at input registration phase the Tumbler SHOULD ban the malicious Alice's provided inputs, if there still are some unspent, and the outputs of the provided inputs spending transaction. The ban SHOULD have a one month timeout.  

If it happens at later phases the round falls back to input registration phase, and all the so far provided CJ outputs SHOULD be banned by the Tumbler.  

Clients MUST not ever register twice the same CJ output, even if the round fails, otherwise the Tumbler could work with that information.  

##### DoS 2: What if an Alice refuses to sign?

The same strategy applied as in DoS 1.

##### DoS 3: What if a Bob does not provide output?

The same strategy applied as in DoS 1 and DoS 2, but with the difference that Alices who do not wish to be banned reveal their link outputs.

#### When to change between phases?

We could stuck the phases into timeframes, but in practice, if a wallet software does not have enough liquidity, that would often result in low, even zero anonimity set rounds, also fixed timeframes would lead to unnecessarily long rounds, therefore a more sophisticated solution is needed:  

##### Long rounds

To solve the unnecessarily long timeframe issue it's the most efficient if we let the Tumbler to make the decisions when to switch between phases when every information successfully arrived. Otherwise time out a phase with one minute.  
However this makes various timing attacks by the Tumbler possible those could deanonymize some Alices.  
To make sure the Tumbler is honest about its phases all clients must setup another identity, let's call it Satoshi that monitors the phases, so the Tumbler does not know who to lie to.  

##### Low anonimity set rounds

When Tumbler reached a desired anonimity set at input registration phase, another connection confirmation phase follows. This phase is intended to sort out disconnected Alices. This will also prevent banning disconnected Alices, who do not indend to malicously not register their signed outputs later, but fail to due to their connectivity.

In order to identify Alices at input registration phase the Tumbler must assign unique identifiers to all Alices, with those unique identifiers Alices can confirm their connection. Note, depending on the anonimity network used, IP addresses are not good way to identify Alices.  

If some Alices did not confirm their registration within the input confirmation phase timeout, then the desired anonimity set is not reached, so we fall back to input registration phase.  

#### Dynamically growing and shrinking anonimity sets

Achieving initial liquidity might be problematic. We can set desired minimum anonimity set. Without reaching this number the round does not switches out of input registration phase, but how should that desired minimum anonimity set number be chosen?  

A simple alghoritm can work, but more sophisticated ones can be applied too:  
Choose the minimum anonimity set to 3 and the maximum to 300. What was the previous desired anonimity set? If the previous input registration phase took more than 3 minutes then decrement this round's desired anonimity set, otherwise increment it.

### C. Full Protocol

## III. Wallet Privacy Framework

### A. Pre-Mix Wallet

A pre-mix wallet can be any Bitcoin wallet, without much privacy requirements.  
Pre-mix wallets MUST either get bitcoin addresses of the post-mix wallet where the mixed coins are going, directly through a secure connection or through the sharing of the post-mix wallet's [extpubkey](https://bitcoin.org/en/glossary/extended-key). In the former case the post-mix wallet must be also online while mixing is in process. In the latter case pre-mix wallet MUST NOT share the extpubkey or any of its derived keys of the post-mix wallet with any third party.  
Pre-mix wallets SHOULD be mixing from Segregated Witness outputs. This lowers the size of the transaction, which not only enables cheaper fees, but also enables achieving higher theoretical anonymity set.  

If the pre-mix wallet normally uses a privacy breaching way to query its address balances (eg. centralized server or bloom filters) and it is using the post mix wallets' extended public key to decide what address to mix to, then it bumps into the issue of how it can query the balances of those addresses in a not privacy breaching way. There might be a number of ways to solve this. For example the pre-mix wallet might want acquire all the Chaumian CoinJoin transactions ever happened in the Blockchain. In this case it SHOULD also acquire all the future Chaumian CoinJoin transactions, in order to avoid mixing twice to the same address, what can happen if the same post-mix wallet extpubkey is feeded into another wallet for mixing.  

Pre-mix and post-mix wallets MAY also be just separate wallet accounts within the same wallet. From an end user perspective the following GUI workflow illustrates how such wallet might work:  

![HiddenWalletTumbleBit1](http://i.imgur.com/xT0Ezvm.png)  
![HiddenWalletTumbleBit2](http://i.imgur.com/rdOGZKG.png)

### B. Post-Mix Wallet

The privacy requirements of the post-mix wallet are much stronger, than the pre-mix wallet's.  
It is not only important for the post-mix wallet to work in a way that doesn't breach privacy, but it is also important that it works in only one way. As an example if the post-mix wallet enables its users to use both [P2WPKH and P2WSH](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki), then a set of users will naturally converge to one type of usage that would lead an observer to valuable conclusions.  
When in the future multiple implementations are created for post-mix wallets, those will want to use the liquidity of existing ZeroLink deployments it is important, these implementations are indistinguishable from the first implementation. For example, when they are sending a transaction they MUST order the outputs of in the same way as the first implementation does.  

#### Coin Selection - Disable Input Joining

If the post-mix wallet would function as a normal Bitcoin wallet, too, the observer would notice post-mix transactions, those are joining together mixed outputs. In this case the real anonymity set of all the users who participated in the same mixes would suffer.  
We could add coin control feature to the post-mix wallet account, as Bitcoin Core does. While that would result in more conscious post-mix wallet management, in the end users would eventually join inputs together.  
![Coin Control Feature](http://i.imgur.com/i67J7JS.png)  
We rather prevent input joining in post-mix wallets altogether. This of course naturally restricts the useability of the wallet. This prevents the users to make bigger than the denomination payments at first, then they are constrained to spend a maximum of their biggest change amount. This is also expected to be violated in many ways, for example a user could keep sending out its freshly mixed coins to another wallet and join their inputs together there. This restriction however is necessary in order to narrow the gap between the theoretical and real anonymity set of all users of the mixer.  
In order to make the usage of the post-mix wallet less painful the wallet MAY also implement some kind of coin control feature, but with the above restriction. The wallet MAY also offer the user to donate smaller change outputs, instead of getting them back. This could even finance the development of such wallet.  

#### Uniform ScriptPubKeys - Use Segregated Witness

Post-mix wallets MUST use P2WPKH outputs as defined in [BIP141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki#Witness_program).  
If post-mix wallets would enable the usage of different scriptpubkeys, then some group of users would naturally tend to use one way more often than others, leading blockchain observers to various conclusions.

#### Indexing Of Transaction Inputs And Outputs - Use BIP69

A post-mix wallet MUST use Lexicographical Indexing of Outputs, as defined in [BIP69](https://github.com/bitcoin/bips/blob/master/bip-0069.mediawiki).  
A post-mix wallet, due to its design, will only have one input and maximum two outputs at all times.  
Uniform indexing is necessary in order for multiple post-mix wallet implementations to look the same.  
Lexiographical indexing is also beneficial from a privacy point of view. If a wallet software would always generate the change output on the second index, observers would always know what the change output is.

#### Fee Estimation, RBF

Another common way blockchain analysis applies to figure out which wallet a transaction was constructed with is by examining the fee its fee patterns, therefore post-mix wallet implementations MUST use unified fee estimating algorithm.  
If post-mix wallets would enable the selection of transaction fees some users would tend to use always one type of transacions while others would use different ones.  
Post-mix wallets MUST use the result of Bitcoin Core RPC's [`estimatefee 2`](https://bitcoin.org/en/developer-reference#estimatefee) for dynamic fee estimation. To obtain the result of course querying a public API is sufficient, too.  

Replace-by-Fee, [RBF](https://bitcoin.org/en/glossary/rbf) is a frequently used feature and is expected to be used in the future even more frequently. While its usage is highly beneficial, it also opens the door for users to use it many different ways, therefore post-mix wallets MUST not utilize this feature.  
Creation of a user independent algorithmic utilization of RBF should be an interest of future research. Bram Cohen's [article](https://medium.com/@bramcohen/how-wallets-can-handle-transaction-fees-ff5d020d14fb) might be a good starting point.

#### Spending Unconfirmed Transactions

It is possible to spend the output of a transaction that did not confirm yet. Post-mix wallets MUST not do such thing. It is not only dangerous, but it will also lead to some users to tend to use it more often, than others.

#### Retrieving Private Transaction Information

Retrieving private transaction information from the blockchain is the [most challenging](https://hackernoon.com/bitcoin-privacy-landscape-in-2017-zero-to-hero-guidelines-and-research-a10d30f1e034) part of implementing any wallet that aims to not breach its users' privacy. Querying the balances of a central server obviously shares private information with that central server. Bloom filtering SPV wallets are [also not a sufficient](https://groups.google.com/forum/#!msg/bitcoinj/Ys13qkTwcNg/9qxnhwnkeoIJ) way to go.  
At the time of writing there are three types of wallet architechtures those don't breach the privacy of the users:
- Full Nodes - Since they download all the transactions the network has, so nobody can tell who's interested in what transactions.  
- Full Block Downloading SPV Wallets - Such wallets also download all transactions the network has, but from the creation of the wallet, so they don't need weeks for Initial Block Downloading and they also not store hundreds of gigabytes of blockchain data, they throws away what they don't need. There are currently 3 implementations of such wallet, all in the testing phase: [Jonas Schnelli's PR to Bitcoin Core](https://github.com/bitcoin/bitcoin/pull/9483), nopara73's [HiddenWallet](https://github.com/nopara73/HiddenWallet) and Stratis' [BreezeWallet](https://github.com/stratisproject/Breeze).
- [Transaction Filtered Full Block Downloading Wallet](https://medium.com/@nopara73/full-node-level-privacy-even-for-mobile-wallets-transaction-filtered-full-block-downloading-wallet-16ef1847c21) - Which only exists as an idea to date.  
  
The good news are there is an easier, user friendly way to achieve it. The post-mix wallet MAY accept deposits to be directly made to its addresses, without mixing. Since there input joining is disallowed there is no reason not to enable that. However if the post-mix wallet disables it, it can simply query all the Chaumian CoinJoin transactions and all its ZeroLink compliant children, since it is not interested in any other information. This would result drastically better user experience, because it does not need to wait hours for blockchain syncing.  

Furthermore, because of every time when CJ transaction fails, new post-mix wallet output is registered, post-mix wallets SHOULD be monitored in huge depth. 21 to 100 would be probably sufficient.

#### Private Transaction Broadcasting

Private transaction broadcasting is a tricky topic.  

As [Dandelion: Privacy-Preserving Transaction Propagation](https://github.com/gfanti/bips/blob/master/bip-dandelion.mediawiki) BIP candidate explains:
> Bitcoin transaction propagation does not hide the source of a transaction very well, especially against a “supernode” eavesdropper that forms a large number of outgoing connections to reachable nodes on the network. From the point of view of a supernode, the peer that relays the transaction *first* is the most likely to be the true source, or at least a close neighbor of the source. Various application-level behaviors of Bitcoin Core enable eavesdroppers to infer the peer-to-peer connection topology, which can further help identify the source.

It gets even worse harder. If an ZeroLink compliant wallet is not a full node an does not constantly relaying, it is not a full node and only connects to other nodes on the network to broadcast its transactions that would result in privacy breach for sure.  

Therefore ZeroLink compliant wallets MUST connect to Tor hidden service nodes and broadcast the transactions to them.  
ZeroLink compliant wallets SHOULD also change Tor circuit between every transaction broadcasts.  

It might also be a sufficiently private way to push transactions to a public API over Tor. This would definitely be easier to implement, but this external dependency cannot be expected to be relied on by all post-mix wallet implementations and all of them should use the same way of broadcasting transactions.  

Private transaction broadcasting should be an interest of future research.

## IV. Conclusions
