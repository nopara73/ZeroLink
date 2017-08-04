# ZeroLink: The Bitcoin Fungibility Framework

## Authors

Ádám Ficsór (nopara73),  
[TumbleBit](https://github.com/NTumbleBit/NTumbleBit), [HiddenWallet](https://github.com/nopara73/HiddenWallet),  
adam.ficsor73@gmail.com

## Support

186n7me3QKajQZJnUsVsezVhVrSwyFCCZ  
[![QR Code](http://i.imgur.com/grc5fBP.png)](https://www.smartbit.com.au/address/186n7me3QKajQZJnUsVsezVhVrSwyFCCZ)

## Abstract

[Fungibility](https://en.wikipedia.org/wiki/Fungibility) is an essential property of good money. Since the invention of [Bitcoin](https://bitcoin.org/bitcoin.pdf) in 2008, numerous fungibility improvements have been proposed, implemented, deployed and used. Some of them build on top of Bitcoin, some of them created their own cryptocurrencies. Up until now no proposal, including privacy centric altcoins, addressed the privacy issue in its full scale. No proposal attempted to provide protections against all the different ways a user's privacy can be breached. As a result, it is not possible to transact with cryptocurrencies in a fully anonymous way. ZeroLink is the first attempt to do that. It evaluates all the research and techniques known to date and combines them in a practical and coherent way.  
The scope of ZeroLink is limited to Bitcoin's first layer. As long as the Bitcoin network is being used, there will always be need for on-chain privacy. Even if an off-chain privacy solution, like the [TumbleBit Payment Hub](https://eprint.iacr.org/2016/575.pdf), [Mimblewimble](https://github.com/ignopeverell/grin) or privacy centric [sidechains](https://elementsproject.org/sidechains/) or [drivechains](http://www.drivechain.info/) become widely adopted, at the end of the day, the entrance and exit transactions will always have to be settled on-chain.  
ZeroLink does not alter the Bitcoin protocol, it does not attempt to make every Bitcoin transaction indistinguishable from each other in the name of fungibility. Such goal would be unrealistic. **ZeroLink's sole objective is to completely break all links between a set of coins and another set of coins.**  
Furthermore the authors are fully committed to build a production ready implementation and deploy it, therefore not get stuck in the research phase.  

## Table Of Contents

I. [Introduction](#i-introduction)  
II. [Chaumian CoinJoin](#ii-chaumian-coinjoin)  
&nbsp;&nbsp;&nbsp;A. [Protocol](#a-protocol)  
&nbsp;&nbsp;&nbsp;B. [Issues](#b-issues)  
III. [Wallet Privacy Framework](#iii-wallet-privacy-framework)  
&nbsp;&nbsp;&nbsp;A. [Pre-Mix Wallet](#a-pre-mix-wallet)  
&nbsp;&nbsp;&nbsp;B. [Post-Mix Wallet](#b-post-mix-wallet)

## I. Introduction

### CoinJoin

[CoinJoin](https://bitcointalk.org/index.php?topic=279249.0) (CJ) was first detailed in 2013 by Gregory Maxwell on Bitcointalk. When multiple participants add inputs and outputs to a common CJ transaction, it confuses blockchain observers.  
[![Wikipedia: CoinJoin](https://upload.wikimedia.org/wikipedia/en/thumb/f/f0/CoinJoinExample.svg/640px-CoinJoinExample.svg.png)](https://en.wikipedia.org/wiki/CoinJoin)

A stronger variant is, if the non-change outputs have the same value, no one can tell which input was intended to fund which of these non-change outputs.  

When multiple participants add inputs to a common CJ transaction and some of the outputs have the same value no one can tell which input was intended to fund which of these outputs.  
CoinJoin based privacy techniques are the most Blockchain space efficient, therefore they are also the cheapest on-chain solutions.  
There is practical limit to what anonymity set they can achieve. While a natural limiting factor would be the [maximum standard transaction size](https://bitcoin.stackexchange.com/a/35882/26859), in which case it goes approximately [from 350 to 470](https://bitcoin.stackexchange.com/questions/57073/what-is-the-maximum-anonimity-set-of-a-coinjoin-transaction/57091), but it can be surpassed, as Maxwell notes:  
> If you can build transactions with m participants per transaction you can create a sequence of m*3 transactions which form a three-stage [switching network](https://en.wikipedia.org/wiki/Clos_network) that permits any of m^2 final outputs to have come from any of m^2 original inputs (e.g. using three stages of 32 transactions with 32 inputs each 1024 users can be joined with a total of 96 transactions).  This allows the anonymity set to be any size, limited only by participation.

For practical reasons, ZeroLink does not attempt to incorporate such switching network into its design, instead it lets the implementer to scale up if the need ever arises.

### Chaumian CoinJoin

Chaumian CoinJoin was also briefly described by Maxwell:  
> Using chaum blind signatures: The users connect and provide inputs (and change addresses) and a cryptographically-blinded version of the address they want their private coins to go to; the server signs the tokens and returns them. The users anonymously reconnect, unblind their output addresses, and return them to the server. The server can see that all the outputs were signed by it and so all the outputs had to come from valid participants. Later people reconnect and sign.  

Every mix via Chaumian CoinJoin comes with a guarantee that Tumbler can neither violate anonymity, nor steal bitcoins. Furtheremore Chaumian CoinJoin is by no means complex. Its simplicity allows it to be one of the most, if not the most performant on-chain mixing technique. A mixing round can be measured in seconds or minutes. 

### Distributed CoinJoin

It is possible to distribute this scheme. Tim Ruffing's [CoinShuffle](http://crypsys.mmci.uni-saarland.de/projects/CoinShuffle/coinshuffle.pdf) and [CoinShuffle++](https://crypsys.mmci.uni-saarland.de/projects/FastDC/draft-paper.pdf) are novel attempts to do so. However distributed systems are hard to get right and their maintenance is problematic: they require various tradeoffs, they are more complex, they open the door for various attacks, updating or upgrading them are difficult. The implementation of Chaumian CoinJoin is straightforward, thus existing wallets can easily implement it. The Tumbler is untrusted, consequently it does not have the risk of coin stealing, nor the risk of privacy breaching, and so distributing this system might not be fully justified from a practical point of view.  
As Maxwell noted:  
>  I don't know if there is, or ever would be, a reason to bother with a fully distributed version with full privacy, but it's certainly possible.  

Of course distributed systems are more resilient, therefore distribution should certainly be an interest of future research.  

### P2P anonymous communication protocols

As Maxwell noted:  
> Any transaction privacy system that hopes to hide user's addresses should start with some kind of anonymity network. This is no different. Fortunately networks like [Tor](https://www.torproject.org/), [I2P](https://geti2p.net/), [Bitmessage](https://bitmessage.org/), and [Freenet](https://freenetproject.org/) all already exist and could all be used for this. (Freenet would result in rather slow transactions, however)  

Another advantage of [CoinShuffle++](https://crypsys.mmci.uni-saarland.de/projects/FastDC/draft-paper.pdf) is that it does not require such anonymity network as an external dependency, rather it implements its own P2P mixing protocol. DiceMix:  
>  We conceptualize these P2P anonymous communication protocols as P2P mixing, and present a novel P2P mixing protocol, DiceMix, that needs only four communication rounds in the best case, and 4 + 2f rounds in the worst case with f malicious peers.  

ZeroLink requires such P2P anonymous protocols at mixing and at transaction broadcasting. [Tor](https://www.torproject.org/) is the most widely deployed such protocol, therefore ZeroLink chooses to rely on it.  
Note also a ZeroLink compliant application should not use a Tor proxy to the clearnet, but rather stay inside the Tor network and constrain its communication with [hidden services](https://www.torproject.org/docs/hidden-services.html.en). This constraint is needed in order to dodge various [attacks](https://en.wikipedia.org/wiki/Tor_(anonymity_network)#Weaknesses).  

Elimination of the Tor dependency should be an interest of future research.  
 
### Privacy Is Teamwork

The theoretical anonymity set of a mixing technique is misleading. A good example is Mycelium's undeployed CoinShuffle implementation: [ShufflePuff](https://github.com/DanielKrawisz/Shufflepuff). It matters little what tracks users leave on the Blockchain and how effective a the implemented mixing technique is if, due to the architechture of their wallets, a third party can re-establish the links.  
If one user of the mix gets deanonymized the real anonymity set of the rest of the users gets lower, too. Therefore it is not acceptable that a set of users are using a mixing technique in a flawed way.

### Transactions And Transaction Chains

Any Bitcoin mixing techique must use a common denomination, otherwise simple amount analysis can re-establish the links, as Kristov Atlas did in his [CoinJoin Sudoku](http://www.coinjoinsudoku.com) analysis of Blockchain.info's [SharedCoin](https://github.com/sharedcoin/Sharedcoin). Since the service has been discontinued.
This notion leads to mixing in multiple rounds. For example if a user wants to mix 8 bitcoins and the mixing denomination is 1 bitcoin, then it must use 8 mixing rounds.  
As Mike Hearn [put it](https://groups.google.com/forum/#!msg/bitcoinj/Ys13qkTwcNg/9qxnhwnkeoIJ):
> The problem starts when we realise that what we actually care about is not transactions but rather transaction chains.  

When a Bitcoin wallet does not find enough value on an unspent transaction output (utxo), then it joins together that utxo with another utxo the wallet contains.  
If our post-mix wallet would function as a normal Bitcoin wallet too, the observer would notice post-mix transactions. Those are joining together mixed outputs. Since pre-mix wallets naturally divide and join utxos in order to fund a mixing round with the correct amount, similarly to CoinJoin Sudoku, a simple amount analysis on transactions chains, instead of transactions could re-establish links between pre-mix and post-mix wallets.  

It is also worth noting that if Gregory Maxwell's [Confidential Transactions](https://elementsproject.org/elements/confidential-transactions/) (CT) are introduced to Bitcoin in the future, it could potentially solve the "common denomination issue". Such technique is Tim Ruffing's [ValueShuffle](https://eprint.iacr.org/2017/238.pdf), which is CoinShuffle with CT.

### Theoretical And Real Anonymity Set

We refer to theoretical anonymity set as the anonymity set that is achieved by a bitcoin mixing technique within one round and does not weigh in external factors, like flawed wallet architechture or network analysis.  
We refer to real anonymity set when these external factors are weighted in and transaction chains are analyzed.  

### Alternative On-Chain Mixing

The Classic Tumbler mode of Ethan Heilman's [TumbleBit](https://eprint.iacr.org/2016/575.pdf) and Gregory Maxwell's [CoinSwap](https://bitcointalk.org/index.php?topic=321228.0) are not CoinJoin based, on-chain privacy techniques. They are both multiple times more expensive and slower than Chaumian CoinJoin. For example Nicolas Dorier's [NTumbleBit](https://github.com/NTumbleBit/NTumbleBit): TumbleBit Classic Tumbler implementation requires 4 transactions, therefore roughly 4 times transaction fees, CoinJoin requires only 1. While NTumbleBit's Classic Tumbler requires hours to complete a round, CoinJoin minutes.  
Tim Ruffing's CoinShuffle, CoinShuffle++, ValueShuffle and Chris Belcher's and Adam Gibson's [JoinMarket](https://github.com/JoinMarket-Org/joinmarket) (JM) are CoinJoin based techniques. We detailed Ruffing's techniques previously, so we need not go in depth here.  
JoinMarket introduced a novel maker-taker concept, where market makers are waiting until a taker wants to execute a CJ transaction and asks market-makers to provide liquidity for his CoinJoin for a small fee. A single JM style CJ of course gets expensive quickly as the anonymity set grows and it achieves plausible deniability rather than unlinkability, because how the makers use their coins after the mix will noticeably differ from the takers' behaviour. JM also provides more complex techniques, like `patientsendpayment.py` and `tumbler.py`. Gibson's detailed analysis of `tumbler.py` can be found in his GitHub: [Analysis of tumbler privacy](https://github.com/AdamISZ/JMPrivacyAnalysis/blob/master/tumbler_privacy.md).  

It is also worth noting that when [Schnorr signatures](https://www.elementsproject.org/elements/schnorr-signatures/) are introduced to Bitcoin in the future, CoinJoin based techniques will get even more Blockchain space efficient.

More detailed comparisons can be found in the article: [TumbleBit vs CoinJoin](https://medium.com/@nopara73/tumblebit-vs-coinjoin-15e5a7d58e3).  

### RFC2119

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" in this document are to be interpreted as described in [RFC2119](https://tools.ietf.org/html/rfc2119).

## II. Chaumian CoinJoin

### A. Protocol

Alice and Bob are the same user, but the Tumbler does not know this.  

![Chaumian CoinJoin](http://i.imgur.com/eVUVM6i.png)

#### 1. Input Registration Phase

Many Alices register their 
 - confirmed utxos as the inputs of the CoinJoin,
 - proofs that they can spend those utxos (messages signed with the corresponding private keys),
 - their desired change outputs,
 - and blinded outputs to the Tumbler.

Tumbler checks if inputs have enough coins, are unspent, confirmed, weren't registered twice and that the provided proofs are valid, then signs the blinded output.  
Alices unblind their signed and blinded outputs.

#### 2. Output Registration Phase

Bobs register their signed outputs to the Tumbler.

#### 3. Signing Phase

Tumbler builds the unsigned CoinJoin transaction and gives it to Alices for signing.  
When all the Alices signed arrive, the Tumbler combines the signatures and propagates the CoinJoin on the network.

### B. Issues

#### DoS attacks

There are various ways malicious users can abort a round and there are various ways to defend against it:

1. Banning IP addresses,  
2. Complete with subset,  
3. Banning the registration of provided utxos and related utxos of malicious Alice.  

Due to the nature of anonymity networks, which tend to reuse IP addresses, banning IP addresses SHOULD NOT be utilized.  

The "complete-with-subset" model MAY be implemented, however it is not clear if its benefits justify its complexity.  

We present a DoS defense based on the utxo registration banning technique. This model makes it economically infeasible to execute DoS attacks.  
We should also note that DoS protection should not be considered as a given, but the Tumbler operator MUST evolve the strictness and sophistication of such protections if the need arises.  

Such protection requires the Tumbler to identify the malicious Alice's utxos it registered as inputs for the CoinJoin. We will cover the identification in detail later, for now let's consider it given.  

In order to explain this scheme we define the term generation according to the following illustration:

![Generations](http://i.imgur.com/RIwvRvL.png) 

When malicious Alice is detected Tumbler SHOULD ban all the outputs Alice registered with and all their (-1)., 0. and 1. generation related outputs. Of course only the ban of unspent utxos are needed, spent ones cannot be registered anyway. Tumbler SHOULD also extend the bans to future, not yet created outputs.  

This should be sufficient to prevent most attackers from trying, however a sophisticated attacker could make another transaction, so its outputs falls into the 2. generation, where the ban is not extended.  
If such technique is used to distrupt another round the Tumbler SHOULD extend its ban to all generations of outputs from 0. to infinity. In this case the 0. generation outputs are not the malicious outputs' generation those are used to disrupt a round the second time, but the first time.  

A ban SHOULD time out after 1 month.  

##### Why is this defense is effective?

There is a way for a both persistent and sophisticated attacker to still disrupt the Tumbling.  
As it will be detailed later, the most sophisticated attacker can delay the execution of a round to maxiumum up to 3 minutes. Therefore there can be a minimum of `24h*(60m/3m)=`480 rounds per day an attacker have to disrupt.  

To execute the first attack the attacker must posess approximately `480/2=`240 bitcoins, assuming 1 bitcoin Tumbler denomination. The attacker first have to pre-divide its reserves into 1 bitcoin outputs, then make another transactions, so when the attack is executed the outputs don't come from the same (-1)., but (-2). generation, which is not banned. Assuming $1 transaction fees, it would take approximately $500 dollar to disrupt the mixing for half a day, by not signing in the end. However the attacker could also make 2 other transactions per malicious outputs, so they end up in the 2. generation, which is not banned, and keep up the attack fo another half a day. This also costs the attacker $500, therefor **keeping up a DoS attack for 1 day would costs approximately $1000 dollar**.  

Because of the 1 month ban time out, it is possible to keep up such attack forever if the attacker has `240*31=`7440 bitcoin reserves for this purpose and willing to pay the $1000 daily attack cost.  

Using an exchange or a bitcoin mixer to bypass the commitment to huge initial bitcoin reserves is also possible, but hardly feasible. This would put the attacker's bitcoins into third party risk and bring additional costs. 

##### What if the malicous output is directly coming out of a mix?

If the output being used to attack is coming out of tumbling the Tumbler SHOULD proceed normally.  
It is not a problem to ban tumbled outputs, becase they SHOULD NOT be tumbled again. Tumbling them again would mean those outputs are being joined together with other outputs, what SHOULD NOT be allowed in a post-mix wallet. More on this later.  

##### DoS 1: What if an Alice spends her input immaturely?

If it happens at Input Registration phase the Tumbler SHOULD ban the malicious Alice.  

If it happens at later phases the round falls back to input registration phase, and all the so far provided CJ outputs SHOULD be banned by the Tumbler.  

Clients MUST not ever register with the same CJ output twice, even if the round fails, otherwise the Tumbler could work with that information.  

##### DoS 2: What if an Alice refuses to sign?

The same strategy applied as in DoS 1.

##### DoS 3: What if a Bob does not provide output?

The same strategy applied as in DoS 1 and DoS 2, but with the difference that Alices who do not wish to be banned reveal their registered outputs in a new Blame Phase.

#### When to change between phases?

We could stick the phases into timeframes, but in practice, if a wallet software does not have enough liquidity, that would often result in low, even zero anonymity set rounds. Also fixed timeframes would lead to unnecessarily long rounds, therefore a more sophisticated solution is needed:  

##### Long rounds

To solve the unnecessarily long timeframe issue it's the most efficient if we let the Tumbler to make the decisions when to switch between phases when all information has successfully arrived. Otherwise time out a phase with one minute.  
However this makes various timing attacks by the Tumbler possible those could deanonymize some Alices.  
To make sure the Tumbler is honest about its phases all clients must setup another identity, let's call it Satoshi that monitors the phases, so the Tumbler does not know who to lie to.  

##### Low anonymity set rounds

When Tumbler has reached a desired anonymity set at input registration phase, another connection confirmation phase follows. This phase is intended to sort out disconnected Alices. This will also prevent banning disconnected Alices, who do not intend to malicously not register their signed outputs later, but fail to due to their connectivity.

In order to identify Alices at input registration phase the Tumbler must assign unique identifiers to all Alices, with those unique identifiers Alices can confirm their connection. Note, depending on the anonymity network used, IP addresses are not good ways to identify Alices.  

If some Alices did not confirm their registration within the input confirmation phase timeout, then the desired anonymity set is not reached, so we fall back to input registration phase.  

#### Dynamically growing and shrinking anonymity sets

Achieving initial liquidity might be problematic. We can set desired minimum anonymity set. Without reaching this number the round does not switch out of the input registration phase, but how should that desired minimum anonymity set number be chosen?  

A simple alghoritm can work, but more sophisticated ones can be applied too:  
Choose the minimum anonymity set to 3 and the maximum to 300. What was the previous desired anonymity set? If the previous input registration phase took more than 3 minutes then decrement this round's desired anonymity set, otherwise increment it.

#### How long a round takes?  

The first phase: Input Registration, using our recommended dynamic anonymity set algorithm at low liquidity could take hours or days. At medium liquidity it will average to 3 minutes, at high liquidity it will run within a few seconds.  

If actors disconnect during Input Registration, Connection Confirmation will time out after 1 minute, otherwise this phase should execute quickly.  

The remaining phases, assuming no malicious actors, optimal anonymity network utilization the bottle neck is the size of the transaction being downloaded by the clients, which at high liquidity would be approximately 100k byte. Even in this case the whole round should execute within a couple of seconds.  

Assuming sophisticated malicious actors at Output Registration, the round aborts within 2 minutes, because the phase's timeout is 1 minute and these Alices could potentially delay their connection confirmation up to 0:59 seconds after the start of Connection Confirmation.  

Assuming worst case sphisticated malicous actors at Signing, the round aborts within 3 minutes, because the timeout of signing phase is 1 minute and they could potentially delay their connection confirmation and output registration up to 0:59 seconds after the start of Connection Confirmation and Output Registration phases.

## III. Wallet Privacy Framework

### A. Pre-Mix Wallet

A pre-mix wallet can be any Bitcoin wallet, without much privacy requirements.  
Pre-mix wallets MUST either get bitcoin addresses of the post-mix wallet where the mixed coins are going directly through a secure connection or through the sharing of the post-mix wallet's [extpubkey](https://bitcoin.org/en/glossary/extended-key) (commonly known as 'extended public key'). In the former case the post-mix wallets must be also online while mixing is in process. In the latter case pre-mix wallets MUST NOT share the extpubkey or any of its derived keys of the post-mix wallets with any third party.  
Pre-mix wallets SHOULD be mixing from Segregated Witness outputs. This lowers the size of the transaction, which not only enables cheaper fees, but also enables achieving higher theoretical anonymity set.  

If the pre-mix wallet normally uses a privacy breaching way to query its address balances (eg. centralized server or bloom filters) and it is using the post mix wallets' extended public key to decide what addresses to mix to, then it bumps into the issue of how it can query the balances of those addresses in a not privacy breaching way. There might be a number of ways to solve this. For example the pre-mix wallet might want to acquire all the Chaumian CoinJoin transactions that ever happened in the Blockchain. In this case it SHOULD also acquire all the future Chaumian CoinJoin transactions, in order to avoid mixing twice to the same address which is what can happen if the same post-mix wallet extpubkey is fed into another wallet for mixing.  

Pre-mix and post-mix wallets MAY also be just separate wallet accounts within the same wallet. From an end user perspective the following GUI workflow illustrates how such wallet might work:  

![HiddenWalletTumbleBit1](http://i.imgur.com/xT0Ezvm.png)  
![HiddenWalletTumbleBit2](http://i.imgur.com/rdOGZKG.png)

### B. Post-Mix Wallet

The privacy requirements of the post-mix wallet are much stronger, than the pre-mix wallet's.  
It is not only important for the post-mix wallet to work in a way that doesn't breach privacy, but it is also important that it works in only one way. As an example if the post-mix wallet enables its users to use both [P2WPKH and P2WSH](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki), then a set of users will naturally converge to one type of usage that would lead an observer to valuable conclusions.  
In future multiple implementations are created for post-mix wallets, those will want to use the liquidity of existing ZeroLink deployments as it is important, these implementations are indistinguishable from the first implementation. For example, when they are sending a transaction they MUST order the outputs of in the same way as the first implementation does.  

#### Coin Selection - Disable Input Joining

If the post-mix wallet would function as a normal Bitcoin wallet, too, the observer would notice post-mix transactions, those are joining together mixed outputs. In this case the real anonymity set of all the users who participated in the same mixes would suffer.  
We could add coin control feature to the post-mix wallet account, as Bitcoin Core does. While that would result in more conscious post-mix wallet management. In the end, users would eventually join inputs together.  
![Coin Control Feature](http://i.imgur.com/i67J7JS.png)  
We rather prevent input joining in post-mix wallets altogether. This of course naturally restricts the useability of the wallet. This prevents the users from making bigger denomination payments at first, then they are constrained to spend a maximum of their biggest change amount. This is also expected to be violated in many ways, for example a user could keep sending out its freshly mixed coins to another wallet and join their inputs together there. This restriction however is necessary in order to narrow the gap between the theoretical and real anonymity set of all users of the mixer.  
In order to make the usage of the post-mix wallet less painful the wallet MAY also implement some kind of coin control feature, but with the above restriction. The wallet MAY also offer the user to donate smaller change outputs, instead of getting them back. This could even finance the development of such wallet.  

#### Uniform ScriptPubKeys - Use Segregated Witness

Post-mix wallets MUST use P2WPKH outputs as defined in [BIP141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki#Witness_program).  
If post-mix wallets would enable the usage of different scriptpubkeys, then some groups of users would naturally tend to use one way more often than others, leading blockchain observers to various conclusions.

#### Indexing Of Transaction Inputs And Outputs - Use BIP69

A post-mix wallet MUST use Lexicographical Indexing of Outputs, as defined in [BIP69](https://github.com/bitcoin/bips/blob/master/bip-0069.mediawiki).  
A post-mix wallet, due to its design, will only have one input and maximum two outputs at all times.  
Uniform indexing is necessary in order for multiple post-mix wallet implementations to look the same.  
Lexiographical indexing is also beneficial from a privacy point of view. If a wallet software would always generate the change output on the second index, observers would always know what the change output is.

#### Fee Estimation, RBF

Another common way blockchain analysis applies to figure out which wallet a transaction was constructed with, is by examining the fee  and its fee patterns. Therefore post-mix wallet implementations MUST use unified fee estimating algorithm.  
If post-mix wallets would enable the selection of transaction fees some users would tend to always use one type of transacion while others would use different ones.  
Post-mix wallets MUST use the result of Bitcoin Core RPC's [`estimatefee 2`](https://bitcoin.org/en/developer-reference#estimatefee) for dynamic fee estimation. To obtain the result of course querying a public API is sufficient too.  

Replace-by-Fee, [RBF](https://bitcoin.org/en/glossary/rbf) is a frequently used feature and is expected to be used in the future more frequently. While its usage is highly beneficial, it also opens the door for users to use it in many different ways, therefore post-mix wallets MUST not utilize this feature.  
Creation of a user independent algorithmic utilization of RBF should be an interest of future research. Bram Cohen's [article](https://medium.com/@bramcohen/how-wallets-can-handle-transaction-fees-ff5d020d14fb) might be a good starting point.

#### Spending Unconfirmed Transactions

It is possible to spend the output of a transaction that did not confirm yet. Post-mix wallets MUST not do such thing. It is not only dangerous, but it will also lead to some users tendency to use it more often, than others.

#### Retrieving Private Transaction Information

Retrieving private transaction information from the blockchain is the [most challenging](https://hackernoon.com/bitcoin-privacy-landscape-in-2017-zero-to-hero-guidelines-and-research-a10d30f1e034) part of implementing any wallet that aims to not breach its users' privacy. Querying the balances of a central server obviously shares private information with that central server. Bloom filtering SPV wallets are [also not a sufficient](https://groups.google.com/forum/#!msg/bitcoinj/Ys13qkTwcNg/9qxnhwnkeoIJ) way to go.  
At this time there are three types of wallet architechtures, those don't breach the privacy of the users:
- Full Nodes - Since they download all the transactions the network has nobody can tell who's interested in what transactions.  
- Full Block Downloading SPV Wallets - Such wallets also download all transactions the network has, but from the creation of the wallet, so they don't need weeks for Initial Block Downloading and they also do not store hundreds of gigabytes of blockchain data. They throw away what they don't need. There are currently 3 implementations of such wallet, all in the testing phase: [Jonas Schnelli's PR to Bitcoin Core](https://github.com/bitcoin/bitcoin/pull/9483), nopara73's [HiddenWallet](https://github.com/nopara73/HiddenWallet) and Stratis' [BreezeWallet](https://github.com/stratisproject/Breeze).
- [Transaction Filtered Full Block Downloading Wallet](https://medium.com/@nopara73/full-node-level-privacy-even-for-mobile-wallets-transaction-filtered-full-block-downloading-wallet-16ef1847c21) - Which only exists as an idea to date.  
  
The good news is that there is an easier and user friendly way to achieve it. The post-mix wallet MAY accept deposits to be directly made to its addresses, without mixing. Since the input joining is disallowed there is no reason not to enable that. However if the post-mix wallet disables it, it can simply query all the Chaumian CoinJoin transactions and all its ZeroLink compliant children, since it is not interested in any other information. This would result in drastically better user experience, because it does not need to wait hours for blockchain syncing.  

Furthermore, because  every time a CJ transaction fails a new post-mix wallet output is registered, post-mix wallets SHOULD be monitored in huge depth. While it is not unlikely that an attacker ever tries to disrupt any round, because of the reasons detailed above, neverthless a post-mix wallet is recommended to monitor 1000 clean addressess after the last used one. In this case a post-mix wallets would still show the right balances if the pre-mix wallet participates in disrupted rounds continously for 2 days.

#### Private Transaction Broadcasting

Private transaction broadcasting is a tricky topic.  

As [Dandelion: Privacy-Preserving Transaction Propagation](https://github.com/gfanti/bips/blob/master/bip-dandelion.mediawiki) BIP candidate explains:
> Bitcoin transaction propagation does not hide the source of a transaction very well, especially against a “supernode” eavesdropper that forms a large number of outgoing connections to reachable nodes on the network. From the point of view of a supernode, the peer that relays the transaction *first* is the most likely to be the true source, or at least a close neighbor of the source. Various application-level behaviors of Bitcoin Core enable eavesdroppers to infer the peer-to-peer connection topology, which can further help identify the source.

It gets even harder. If a ZeroLink compliant wallet is not a full node and constantly relaying, it is not a full node and only connects to other nodes on the network to broadcast its transactions and that would result in privacy breach for sure.  

Therefore ZeroLink compliant wallets MUST connect to Tor hidden service nodes and broadcast the transactions to them.  
ZeroLink compliant wallets SHOULD also change Tor circuit between every transaction broadcasts.  

It might also be a sufficiently private way to push transactions to a public API over Tor. This would definitely be easier to implement, but this external dependency cannot be expected to be relied on by all post-mix wallet implementations and all of them should use the same way of broadcasting transactions.  

Private transaction broadcasting should be an interest of future research.
