# ZeroLink: The Bitcoin Fungibility Framework

![](http://i.imgur.com/ODjt3wf.png)

## Authors

nopara73,  
[Hidden Wallet](https://github.com/nopara73/HiddenWallet),  
adam.ficsor73@gmail.com

TDevD,  
[Samourai Wallet](https://github.com/Samourai-Wallet),  
[PGP](http://pgp.mit.edu/pks/lookup?op=get&search=0x72B5BACDFEDF39D7)

### Acknowledgements

Special thanks for Adam Gibson and Chris Belcher from [JoinMarket](https://github.com/JoinMarket-Org/joinmarket), Ethan Heilman from [TumbleBit](https://eprint.iacr.org/2016/575.pdf), Dan Gershony from [Breeze Wallet](https://github.com/stratisproject/Breeze/) and Kristov Atlas from [Open Bitcoin Privacy Project](http://openbitcoinprivacyproject.org/) for tolerating my constant bugging and bothering to acquire their invaluable reviews, suggestions and feedbacks.

## Support

186n7me3QKajQZJnUsVsezVhVrSwyFCCZ  
[![QR Code](http://i.imgur.com/grc5fBP.png)](https://www.smartbit.com.au/address/186n7me3QKajQZJnUsVsezVhVrSwyFCCZ)

## [Publicity And External Resources](PublicityExternalResources.md)

## Abstract

While [fungibility](https://en.wikipedia.org/wiki/Fungibility) is an essential property of good money, [Bitcoin](https://bitcoin.org/bitcoin.pdf) has its limitations in this area. Numerous fungibility improvements have been proposed; however none of them have addressed the privacy issues in full. ZeroLink is first to offer protections against all the different ways a user's privacy can be breached. The scope of ZeroLink is not limited to a single transaction, it extends to transaction chains and it addresses various network layer deanonymizations, however its scope is limited to Bitcoin's first layer. Even if an off-chain anonymity solution gets widely adopted, ultimately the entrance and exit transactions will always be settled on-chain. Therefore there will always be need for on-chain privacy.  
  
Ideal fungibility requires every Bitcoin transaction to be indistinguishable from each other, but it is an unrealistic goal. ZeroLink's objective is to break all links between separate sets of coins. ZeroLink presents a wallet privacy framework coupled with Chaumian CoinJoin, which was first introduced in 2013 by Gregory Maxwell. A mixing round runs within seconds, its anonymity set can go beyond a single CoinJoin transaction's if needed, and its DoS resilience presumes a transaction fee environment above $1 Bitcoin.  
  
Hopefully, ZeroLink will enable the usage of Bitcoin in a fully anonymous way for the first time.  

## Table Of Contents

I. [Introduction](#i-introduction)  
II. [Chaumian CoinJoin](#ii-chaumian-coinjoin)  
&nbsp;&nbsp;&nbsp;A. [Simplified Protocol](#a-simplified-protocol)  
&nbsp;&nbsp;&nbsp;B. [Achieving Liquidity](#b-achieving-liquidity)  
&nbsp;&nbsp;&nbsp;C. [Optimizing Performance](#c-optimizing-performance)  
&nbsp;&nbsp;&nbsp;D. [DoS Attack](#d-dos-attack)  
&nbsp;&nbsp;&nbsp;E. [Sybil Attack](#e-sybil-attack)  
III. [Wallet Privacy Framework](#iii-wallet-privacy-framework)  
&nbsp;&nbsp;&nbsp;A. [Pre-Mix Wallet](#a-pre-mix-wallet)  
&nbsp;&nbsp;&nbsp;B. [Post-Mix Wallet](#b-post-mix-wallet)  
&nbsp;&nbsp;&nbsp;C. [Stealth Addresses](#c-stealth-addresses)  

## I. Introduction

### Overview

![](http://i.imgur.com/ZtLn8cA.png)

ZeroLink defines a pre-mix and a post-mix wallet and a mixing technique.  
Pre-mix wallet functionality can be added to any Bitcoin wallet without much overhead. Post-mix wallets on the other hand have strong privacy requirements, regarding coin selection, private transaction and balance retrieval, transaction input and output indexing and broadcasting. The requirements and recommendations for pre and post-mix wallets together define the Wallet Privacy Framework.  
Coins from pre-mix wallets to post-mix wallets are moved by mixing. Most on-chain mixing techniques, like [CoinShuffle](http://crypsys.mmci.uni-saarland.de/projects/CoinShuffle/coinshuffle.pdf),  [CoinShuffle++](https://crypsys.mmci.uni-saarland.de/projects/FastDC/draft-paper.pdf) or [TumbleBit's Classic Tumbler mode](https://eprint.iacr.org/2016/575.pdf) can be used. However ZeroLink defines its own mixing technique: Chaumian CoinJoin.

### CoinJoin
[![Wikipedia: CoinJoin](https://upload.wikimedia.org/wikipedia/en/thumb/f/f0/CoinJoinExample.svg/640px-CoinJoinExample.svg.png)](https://en.wikipedia.org/wiki/CoinJoin)

[CoinJoin](https://bitcointalk.org/index.php?topic=279249.0) was first detailed in 2013 by Gregory Maxwell on BitcoinTalk. When multiple participants add inputs and outputs to a common transaction, it obfuscates the transaction graph.  
A stronger variant is, if the non-change outputs have the same value, no one can tell which input intended to fund which of these non-change outputs.  

CoinJoin based privacy techniques are the most Blockchain space efficient, therefore they are the cheapest on-chain solutions.  
The limiting factor of CoinJoin's anonymity set is the [maximum standard transaction size](https://bitcoin.stackexchange.com/a/35882/26859), in which case it goes approximately [from 350 to 470](https://bitcoin.stackexchange.com/questions/57073/what-is-the-maximum-anonimity-set-of-a-coinjoin-transaction/57091). Although it can be surpassed, as Maxwell notes:  
> If you can build transactions with m participants per transaction you can create a sequence of m*3 transactions which form a three-stage [switching network](https://en.wikipedia.org/wiki/Clos_network) that permits any of m^2 final outputs to have come from any of m^2 original inputs (e.g. using three stages of 32 transactions with 32 inputs each 1024 users can be joined with a total of 96 transactions).  This allows the anonymity set to be any size, limited only by participation.

For practical reasons, ZeroLink does not attempt to incorporate such switching network into its design, instead it lets the implementor to scale up if the need ever arises.

### Chaumian CoinJoin

Chaumian CoinJoin was briefly described by Maxwell:  
> Using chaum blind signatures: The users connect and provide inputs (and change addresses) and a cryptographically-blinded version of the address they want their private coins to go to; the server signs the tokens and returns them. The users anonymously reconnect, unblind their output addresses, and return them to the server. The server can see that all the outputs were signed by it and so all the outputs had to come from valid participants. Later people reconnect and sign.  

Simplified workflow:  
1. User provides its **input** and a **blinded output** to the Tumbler.  
2. Tumbler signs the **blinded output** and gives it back to the User.
3. User unblinds the **signed blinded output** and provides the server the **signed output** through a different anonymity network identity.  
4. The Tumbler constructs the **CoinJoin transaction** and gives out to the Users for signing.

Every mix via Chaumian CoinJoin comes with a guarantee that Tumbler can neither violate anonymity, nor steal bitcoins. Furthermore Chaumian CoinJoin is by no means complex. Its simplicity allows it to be one of the most, if not the most performant on-chain mixing technique. A mixing round can be measured in seconds or minutes.  

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

ZeroLink requires such P2P anonymous protocols at mixing and at transaction broadcasting. [Tor](https://www.torproject.org/) is the most widely deployed such protocol. A ZeroLink compliant application should not use a Tor proxy to the clearnet, instead it should stay inside the Tor network and constrain its communication with [hidden services](https://www.torproject.org/docs/hidden-services.html.en). This constraint is needed to dodge various [attacks](https://en.wikipedia.org/wiki/Tor_(anonymity_network)#Weaknesses).  

Elimination of the Tor dependency should be an interest of future research.  
 
### Privacy Is Teamwork

The theoretical anonymity set of a mixing technique is misleading. If one user of the mix gets deanonymized, the real anonymity set of the rest of the users drops. For instance, if one user participates in the mix through a full node and the rest through a web wallet, the anonymity set of the full node user against the web wallet company is zero. Therefore it is not acceptable that a set of users are using a mixing technique in a flawed way.

### Transactions And Transaction Chains

![](http://i.imgur.com/8DYjaqq.png)

Any Bitcoin mixing technique must use a common denomination, otherwise simple amount analysis can re-establish the links, as Kristov Atlas did in his [CoinJoin Sudoku](http://www.coinjoinsudoku.com) analysis of Blockchain.info's [SharedCoin](https://github.com/sharedcoin/Sharedcoin). Since the service has been discontinued.
This notion leads to mixing in multiple rounds. For example if a user wants to mix eight bitcoins and the mixing denomination is one bitcoin, then it must use eight mixing rounds.  
Additionally when a Bitcoin wallet does not find enough value on an unspent transaction output (utxo), then it joins together that utxo with another utxo the wallet contains.  
If the post-mix wallet would function as a normal Bitcoin wallet too, the observer would notice post-mix transactions. Those are joining together mixed outputs. Since pre-mix wallets naturally divide and join utxos in order to fund a mixing round with the correct amount, similarly to CoinJoin Sudoku, a simple amount analysis on transactions chains, instead of transactions could re-establish links between pre-mix and post-mix wallets.  

![](http://i.imgur.com/AqnwKMr.png)

Moreover if Gregory Maxwell's [Confidential Transactions](https://elementsproject.org/elements/confidential-transactions/) are introduced to Bitcoin in the future, it could potentially solve the "common denomination issue". Such technique is Tim Ruffing's [ValueShuffle](https://eprint.iacr.org/2017/238.pdf), which is CoinShuffle with Confidential Transactions.

### Theoretical And Real Anonymity Set

Theoretical anonymity set refers to the anonymity set that is achieved by a bitcoin mixing technique within one round and does not weigh in external factors, like flawed wallet architecture or network analysis.  
Real anonymity set is when these external factors are weighted in and transaction chains are analyzed.  

### Alternative On-Chain Mixing

The Classic Tumbler mode of Ethan Heilman's [TumbleBit](https://eprint.iacr.org/2016/575.pdf) and Gregory Maxwell's [CoinSwap](https://bitcointalk.org/index.php?topic=321228.0) are not CoinJoin based, on-chain privacy techniques. They are both multiple times more expensive and slower than Chaumian CoinJoin. For example Nicolas Dorier's [NTumbleBit](https://github.com/NTumbleBit/NTumbleBit): TumbleBit Classic Tumbler implementation requires four transactions, therefore approximately four times transaction fees, CoinJoin requires only one. While NTumbleBit's Classic Tumbler requires hours to complete a round, CoinJoin minutes.  
Tim Ruffing's CoinShuffle, CoinShuffle++, ValueShuffle and Chris Belcher's and Adam Gibson's [JoinMarket](https://github.com/JoinMarket-Org/joinmarket) are CoinJoin based techniques. Ruffing's techniques were previously discussed, thus there is need not go in depth here.  
JoinMarket introduced a novel maker-taker concept, where market makers are waiting until a taker wants to execute a CoinJoin transaction and asks market-makers to provide liquidity for his CoinJoin for a small fee. A single JoinMarket style CoinJoin of course gets expensive quickly as the anonymity set grows and it achieves plausible deniability rather than unlinkability, because how the makers use their coins after the mix will noticeably differ from the takers' behaviour. In addition JoinMarket provides more complex techniques, like `patientsendpayment.py` and `tumbler.py`. Gibson's detailed analysis of `tumbler.py` can be found in his GitHub: [Analysis of tumbler privacy](https://github.com/AdamISZ/JMPrivacyAnalysis/blob/master/tumbler_privacy.md).  

Moreover when [Schnorr signatures](https://www.elementsproject.org/elements/schnorr-signatures/) are introduced to Bitcoin in the future, CoinJoin based techniques will get even more Blockchain space efficient.  
More detailed comparisons can be found in the article: [TumbleBit vs CoinJoin](https://medium.com/@nopara73/tumblebit-vs-coinjoin-15e5a7d58e3).  

Furthermore it is possible to plug CoinShuffle, CoinShuffle++ or TumbleBit's Classic Tumbler mode into ZeroLink, instead of Chaumian CoinJoin.  

### RFC2119

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" in this document are to be interpreted as described in [RFC2119](https://tools.ietf.org/html/rfc2119).

## II. Chaumian CoinJoin

### A. Simplified Protocol

Alice and Bob are the same user, however the Tumbler does not know this.  

![Chaumian CoinJoin](http://i.imgur.com/eVUVM6i.png)

#### 1. Input Registration Phase

Many Alices register their 
 - confirmed utxos as the inputs of the CoinJoin,
 - proofs, those are messages signed with the corresponding private keys,
 - their desired change outputs,
 - and blinded outputs to the Tumbler.

Tumbler checks if inputs have enough coins, are unspent, confirmed, were not registered twice and that the provided proofs are valid, then signs the blinded output.  
Alices unblind their signed and blinded outputs.

#### 2. Output Registration Phase

Bobs register their signed outputs to the Tumbler.

#### 3. Signing Phase

Tumbler builds the unsigned CoinJoin transaction and gives it to Alices for signing.  
When all the Alices signed arrive, the Tumbler combines the signatures and propagates the CoinJoin on the network.

### B. Achieving Liquidity

When a round does not have enough liquidity, that would often result in low, even zero anonymity set rounds.  
The solution is when Tumbler has reached a desired anonymity set at Input Registration phase, another Connection Confirmation phase follows. This phase is intended to sort out disconnected Alices.  
In order to identify Alices: at Input Registration phase the Tumbler must assign unique identifiers to them. Using these unique identifiers Alices can confirm their connection at Connection Confirmation phase.  
If some Alices did not confirm their registration within the input confirmation phase timeout, then the desired anonymity set is not reached, in consequence the round falls back to Input Registration phase.  

How should the desired minimum anonymity set be chosen? Manually or utilizing a dynamic algorithm:  
Choose the minimum anonymity set to three and the maximum to 300. If the previous non-fallback Input Registration phase took more than three minutes then decrement this round's desired anonymity set relative to the previous desired anonymity set, otherwise increment it.  
More sophisticated algorithms may be applied, too.

#### No need for multiple mixing rounds

If the denomination is one bitcoin and the user wants to mix eight bitcoins it must participate in eight mixing rounds. By allowing the user to register eight outputs within one round, this issue can be bypassed, resulting eight times cheaper and faster mixing. The drawbacks are weaker anonymity set, less liquidity, more complex implementation and longer mixing rounds. This improvement should be considered to be implemented when a Tumbler achieved massive liquidity. In depth discussion and specification can be found under the issue: [Bypass the need for multiple mixing rounds](https://github.com/nopara73/ZeroLink/issues/21).

### C. Optimizing Performance

When to change between phases?  
Phases can be triggered by Bitcoin blocks, for instance every time a block arrives the next phase is triggered. In order to eliminate the inconsistencies of the Bitcoin network it is a better idea to trigger a new phase at every even block.  
Nonetheless it results unnecessarily long mixing rounds.  
Another way of doing is to stick phases into timeframes. Assuming performant Tumbler and optimal utilization of the anonymity network by the clients, one minute is enough to complete every phase. While it is a more performant way to complete a tumbling round, it is still not optimal.  
Optimal performance is achieved when the Tumbler triggers the changes between phases, because it is the only actor that is aware of when a phase completes. The issue is: Tumbler can execute various timing attacks, those result in user deanonymization. To make sure the Tumbler is honest about its phases all clients must setup another, monitoring identity: Satoshi, who monitors the phases, so the Tumbler does not know who to lie to.  
In addition every phase must time out after one minute. Timeout happens when malicious or disconnected Alice is detected.

#### How long does a round take?  

The first phase: Input Registration, using the recommended dynamic anonymity set algorithm at low liquidity could take hours or days. At medium liquidity it will average to three minutes, at high liquidity it will run within a few seconds.  

If actors disconnect during Input Registration, Connection Confirmation will time out after one minute, otherwise this phase should execute quickly.  

The remaining phases, assuming no malicious actors, optimal anonymity network utilization the bottle neck is the size of the transaction being downloaded by the clients, which at high liquidity would be approximately 100k byte. Even in this case the round should execute within a couple of seconds.  

Assuming sophisticated malicious actors at Output Registration, the round aborts within two minutes, because the phase's timeout is one minute and these Alices could potentially delay their connection confirmation up to 0:59 seconds after the start of Connection Confirmation.  

Assuming worst case sophisticated malicious actors at Signing, the round aborts within three minutes, because the timeout of signing phase is one minute and they could potentially delay their connection confirmation and output registration up to 0:59 seconds after the start of Connection Confirmation and Output Registration phases.

#### Speeding Up Mixing

All Chaumian CoinJoin input MUST be Segregated Witness input. This prevents the transaction to be malleated, as a result the Tumbler can accept unconfirmed Chaumian CoinJoin change outputs from the user in the next round.

#### Accepting Zero Confirmation
If a 

### D. DoS Attack

There are various ways malicious users can abort a round and there are various ways to defend against it:

1. Banning IP addresses,  
2. Complete with subset,  
3. Closed source DoS protection,  
4. Utilization of fidelity bond,  
5. Banning the registration of provided utxos and related utxos of malicious Alice.


Due to the nature of anonymity networks, which tend to reuse IP addresses, banning IP addresses SHOULD NOT be utilized.  
The "complete-with-subset" model MAY be implemented, however it is not clear if its benefits justify its complexity. A Tumbler MAY close source its DoS protection algorytm, thus forcing attackers into reverse engineering.  
[Utilization of fidelity bonds](https://github.com/nopara73/ZeroLink/issues/6#issuecomment-321662470) SHOULD NOT be utilized. It ruins user experience and results in longer rounds.  
This document recommends a DoS defense based on the utxo registration banning technique, which makes it economically infeasible to execute DoS attacks. In addition the Tumbler operator MUST evolve the protections if the need arises.  
This protection requires the Tumbler to identify the malicious Alice's utxos it registered as inputs for the CoinJoin. The identification of malicious utxos is explained by examining all possible variations of DoS attacks.

**DoS 1: What if an Alice spends her input prematurely?**  
If it happens at Input Registration phase the Tumbler SHOULD ban the malicious Alice.  
If it happens at later phases the round falls back to input registration phase, and all the so far provided CoinJoin outputs SHOULD be banned by the Tumbler.  
Clients MUST not ever register with the same CoinJoin output twice, even if the round fails, otherwise the Tumbler could work with that information.  
**DoS 2: What if an Alice refuses to sign?**  
The same strategy applied as in DoS 1.  
**DoS 3: What if a Bob does not provide output?**  
The same strategy is applied as in DoS 1 and DoS 2. The only difference is that Alices who do not wish to be banned reveal their registered outputs in a new Blame Phase.

A ban SHOULD time out after one month.  

To find the optimal severity of utxo banning the attacker's Initial Bitcoin Requirements and Attack Costs are helpful metrics. These metrics are calculated in this document by assuming one bitcoin Tumbler denomination, $1 network transaction fees and that the attacker is willing to keep up the attack for one day.  
The most sophisticated attacker can delay the execution of a round maximum up to three minutes. Therefore there can be a minimum of `24h*(60m/3m)=`480 rounds per day an attacker must to disrupt.  
For simplicity this document assumes a malicious Alice only registered one utxo. If there are any other utxos Alice registered with, the same ban applies to them.  

#### Severity 0: No utxo banning

![](http://i.imgur.com/dVMnVLO.png)

At level zero severity the attacker can re-register and disrupt a round as many times as it wants.

Attack|Initial Bitcoin Requirements|Attack Costs
------|----------------------------|---------------
I.    |1btc                        |$0

#### Severity 1: Banning the malicious utxo

![](http://i.imgur.com/SBqVPwb.png)

In this case the most effective attack if the attacker holds 480btc. Because nobody has 480btc happened to be predivided perfectly to 1btc outputs, the attacker must first predivide them and attack with those utxos. Predividing such amount is 1 transaction with 480 outputs. A transaction output is [approximately 20%](https://bitcoin.stackexchange.com/q/1195/26859) of a transaction, therefore the cost of this attack is `480out*0.2=`$96.  

The second attack can be executed with less Initial Bitcoin Requirements. The attacker can first disrupt a round, then make a transaction, so the output of that transaction is not banned, then register that output to the next round. Of course Bitcoin transactions are not instant and a Tumbler only accepts confirmed outputs, thus assuming every Bitcoin transaction confirms within ten minutes, the attacker must have around four bitcoins to begin with. By not factoring in the predivision, the attacker must make `480-4=`476 transactions to disrupt the tumbling for a day. That costs $476.

Attack|Initial Bitcoin Requirements|Attack Costs
------|----------------------------|---------------
I.    |480btc                      |$96
II.   |4btc                        |$476

#### Severity 2: Banning the malicious utxo and all its sibling utxos

![](http://i.imgur.com/AqmaimX.png)

The first attack, where the attacker holds 480btc does not work anymore. Because of the predivision, all the utxos would be banned:

![](http://i.imgur.com/Uz8uw80.png)

Therefore what the attacker would have to do is to predivide its coins in a different way. It cannot create one big transaction, instead it creates 480 transactions, thus its attack cost is $480.  

The second attack results in exactly 480 transactions, too.  

Attack|Initial Bitcoin Requirements|Attack Costs
------|----------------------------|---------------
I.    |480btc                      |$480
II.   |4btc                        |$480

#### Severity 3,4,5,6...

To impose additional costs to the second type of attack the Tumbler can ban the outputs of the transaction that spends the malicious output.

![](http://i.imgur.com/BURPSWP.png)

In this case the attacker has to do one extra transaction to be able to use its coins for attacking again. After the predivision the attacker can disrupt four rounds, spends its banned malicious outputs, each one twice. Note, spending an unconfirmed output is valid. That results in `2*4=`8 transactions. It disrupts four more rounds, then spends eight more transactions and so on... The final transaction count will be `(480-4)*2=`952.

Attack|Initial Bitcoin Requirements|Attack Costs
------|----------------------------|---------------
I.    |480btc                      |$480
II.   |4btc                        |$952

The higher the severity is, the higher the Attack Costs are.

![](http://i.imgur.com/YFuYI8d.png)

The issue is increasing severity might result in banning honest actors out of the mix: if honest actors get their coins from malicious ones, therefore severity should be kept at level two and only to be increased if needed.  

#### Imposing additional Attack Costs to attackers with huge Initial Bitcoin Reserves

Moving the other direction on the transaction chain, towards the parents of the malicious utxo and banning them and their childs to participate in further mixes imposes additional costs to attackers with huge Initial Bitcoin Reserves. Such strategies should be used only if needed because it assumes the parent utxos and their childs are controlled by the attacker. This assumption increases the possibility of banning honest actors.

#### Lowering denomination

By calculating the metrics the Tumbler denomination of one bitcoin was assumed. Lowering this does not affect the Attack Costs, it only affects the Initial Bitcoin Requirements. 

#### Dependence on high Bitcoin transaction fees

Attack Cost calculation assumed $1 Bitcoin transaction fees. The proposed DoS defense in a zero fee environment is not sufficient.

#### Can this system be bypassed with Bitcoin exchanges/mixers or similar services?

The Attack Costs cannot be bypassed. Using such service would only impose additional costs on the attacker and introduce third party risk.

### E. Sybil Attack

![](http://i.imgur.com/oQyrzqc.png)

It is possible to deanonymize a user if every participant of the mix is the attacker, except the user. Similarly to [Xim: Sybil-Resistant Mixing for Bitcoin](https://people.cs.umass.edu/~gbiss/mixing.pdf), the cost of this attack grows as the anonymity set grows. However unlinke in Xim, this attack is only feasible if the Tumbler is the attacker. If the attacker is not the Tumbler, it would have to figure out exactly in which rounds the targeted user participates and it must make sure nobody else gets to participate in that mix.  
Furthermore executing a covert Sybil attack as a Tumbler is not evident, it depends on the protocol implementation. Overt Sybil attack as a Tumbler is always possible, however in that case the Tumbler is accountable.  

To execute this attack: when Tumbler notices an input is registered that it wants to deanonymize, it must refuse all following input registration and all the input confirmation that has already been registered and is not from the target. Refusing input registration can happen for many raeason, therefore it can be done in a covert way, however refusing input conformination cannot. It can only happen if the input has been spent, therefore malicious Tumbler can be noticed. Clients whose input confirmations are refused and they did not prematurely spent their inputs SHOULD NOT use the Tumbler anymore.  
The cost of the Sybil attack at $1 tranasction fees is `1.2 * number of sybils * $1`. If the number of sybils is 100 and the denomination is one bitcoin, the Tumbler must first predivide 100btc into 100 one btc outputs, which is about `$1*(100*0.2)`= $20, wait until the transaction confirms, then it must pay the CoinJoin fees, which is about $100, so the cost of this attack is $120 per round.  
This pattern can be noticed by the post-mix wallet. In this case the post mix wallet MAY require re-mixing the coins.  

There are various other ways to address Tumbler Sybil attacks in expense of the complexity of pre-mix wallet implementations. Defending Sybil attack should be an interest of future research.

## III. Wallet Privacy Framework

### A. Pre-Mix Wallet

A pre-mix wallet can be any Bitcoin wallet, without much privacy requirements.  
Pre-mix wallets MUST either get bitcoin addresses of the post-mix wallet directly, for instance through a local RPC API or through the sharing of the post-mix wallet's [extended public key](https://bitcoin.org/en/glossary/extended-key). In the latter case pre-mix wallets MUST NOT share the extended public key or any of its derived keys of the post-mix wallets with any third party.  
Pre-mix wallets MUST be mixing from Segregated Witness outputs. This lowers the size of the transaction, thus enabling lower transaction fees overall, allows for a higher theoretical anonymity set and enables faster mixing by not needing to wait for confirmation when the input is an output of a Chaumian CoinJoin transaction, because the transaction will not be malleated.   

Pre-mix and post-mix wallets MAY be separate wallet accounts within the same wallet. From an end user perspective the following GUI workflow illustrates how such wallet might work:  

![HiddenWalletTumbleBit1](http://i.imgur.com/xT0Ezvm.png)  
![HiddenWalletTumbleBit2](http://i.imgur.com/rdOGZKG.png)

#### Retrieving Transaction Information  

A pre-mix wallet can use a privacy breaching way to retrieve transaction and balance information, for instance it can query its address balances through a web API. In this case the web API knows about all the addresses the user possesses. However a pre-mix wallet MUST NOT use a privacy breaching way to acquire information about the childs of the extended public key, otherwise it would expose the post-mix wallet to a third party. An additional problem is that the pre-mix wallet cannot ever register the same addresses twice to a Tumbler. Therefore the pre-mix wallet must always register the next unused extended public key child, that was not registered before.  

A user to use the same extended public key in multiple pre-mix wallets is unlikely to happen, as well should be discouraged. If this is a given, a pre-mix wallet can keep records which derived keys it already registered before and never acquire their balances. This approach brings additional issues at wallet recovery.  

Another way to solve this is to have a server that tells the pre-mix wallet all the addresses those have ever been used in CoinJoin transactions. In this case the pre-mix wallet does not expose which addresses it is interested in, because it gets all the addresses a any pre-mix wallet can be interested in. Additionally a pre-mix wallet MUST keep records which derived keys it already registered before.  
This approach is reliable, it can handle proper wallet recovery and the case if multiple pre-mix wallets use the same extended public keys. Some information leak is still possible, however it is unlikely.  
Information leak happens if:  
- a malicious attacker disrupted a round that the user is participated in
- AND the user either decides to recover its wallet OR is using the same extended public keys in another pre-mix wallet right after the disrupted round  
- AND the Tumbler does not reject the already registered, but unused address  
- AND the Tumbler is malicious.

If all the above conditions are true, the Tumbler may be able to deanonymize the user.

The Tumbler MAY be the third party who serves the addresses. In this case the Tumbler could serve the already registered, but unused addresses, too.

### B. Post-Mix Wallet

The privacy requirements of the post-mix wallet are stronger, than the pre-mix wallet's.  
A post-mix wallet MUST NOT breach its users privacy and it SHOULD work in the same way as every other post-mix wallet. For example if only one wallet software is used as a post-mix wallet and it supports Replace-by-Fee (https://github.com/bitcoin/bips/blob/master/bip-0125.mediawiki) Blockchain analysis cannot come to valuable conclusions. However if different wallet softwares are used as post-mix wallets and one of them does not support it, blockchain analysis can identify which wallet software is used as a post-mix wallet.  
The first implementation of post-mix wallet will set precedents. In the future, when multiple implementations are created, it is important that these implementations are indistinguishable from the first implementation.  

**Basic Post-Mix Wallet Requirement** refers to the requirement that the wallet software must fulfil in order to avoid after-mix deanonymization, assuming the wallet software is the only wallet software that is used as a post-mix wallet of a specific mix.  
**Post-Mix Wallet Uniformity Requirement** refers to the requirement that the wallet software must fulfil in order to avoid after-mix deanonymization, assuming the wallet software is NOT the only wallet software that is used as a post-mix wallet of a specific mix.  

#### Coin Selection
|Basic Post-Mix Wallet Requirement|Post-Mix Wallet Uniformity Requirement|
|---------------------------------|--------------------------------------|
|Post-mix wallet MUST prevent joining inputs together.||

![](http://i.imgur.com/BbSM9N4.png)

If the post-mix wallet would function as a normal Bitcoin wallet, the observer would notice post-mix transactions, those are joining together mixed outputs. In this case the real anonymity set of all the users who participated in the same mixes would suffer.  
Adding coin control feature to the post-mix wallet account in the same way Bitcoin Core does encourages more conscious post-mix wallet management. Nevertheless, users would eventually still join inputs together.  
![Coin Control Feature](http://i.imgur.com/i67J7JS.png)  
It is better to prevent input joining in post-mix wallets altogether. This, of course naturally restricts the useability of the wallet. This prevents the users from making bigger denomination payments at first, then they are constrained to spend a maximum of their biggest change amount. This is expected to be violated in many ways, such as a user could keep sending out its freshly mixed coins to another wallet and join their inputs together there. This restriction however is necessary in order to narrow the gap between the theoretical and real anonymity set of all users of the mixer.  

To enhance useability of a post-mix wallet
- the wallet MAY implement coin control feature, 
- the wallet MAY offer the user to donate smaller change outputs, instead of getting them back. This could even finance the development of such wallet,  
- the wallet MAY also implement a visualized transaction history, instead of a traditional wallet history.

##### Dead Change Attack

![](http://i.imgur.com/3LtkqNA.png)

While the recommended strict coin selection properly separates mixed coins from each other, it does not protect against the [dead change attack](https://www.dash.org/forum/threads/dead-change-an-anonymity-issue.3019/). Change outputs will be used for different purposes, therefore it is possible to connect those payments together. Fortunately it does not affect the anonymity set of other users who participated in the mix, but it does affect individual privacy.  
To encourage more cautious user behavior post-mix wallet MAY implement a transaction labeling system, so users can decide manually what purcheses it does not care about if they are connected together by third party observers.  

##### Additional Anonymity Set

A post-mix wallet MAY offer to make a user's first purchase to be a regular CoinJoin transaction, without the usage of fixed denomination so additional anonymity set can be achieved. In this CoinJoin every input transaction is the same denomination, therefore an observer will not be able to tell who wanted to pay who, it can only figure out which change belongs to which active output.  

![](https://i.imgur.com/1IotuiI.png)

#### Change ScriptPubKeys
|Basic Post-Mix Wallet Requirement|Post-Mix Wallet Uniformity Requirement|
|---------------------------------|--------------------------------------|
||Post-mix wallet SHOULD always generate P2WPKH ScriptPubKeys as the change output of a built transction.|

#### Active SriptPubKeys
|Basic Post-Mix Wallet Requirement|Post-Mix Wallet Uniformity Requirement|
|---------------------------------|--------------------------------------|
||Post-mix wallet SHOULD be able to build transactions to P2PKH, P2WPKH, P2SH and P2WSH active outputs.|  

If all post-mix wallet software would only be able to send to P2PKH active outputs, except one post-mix wallet software, that supports P2WPKH active outputs, too, then Blockchain analysis can identify the outlier post-mix wallet software.  

#### Transaction Output Indexing
|Basic Post-Mix Wallet Requirement|Post-Mix Wallet Uniformity Requirement|
|---------------------------------|--------------------------------------|
||Post-mix wallet SHOULD index its built transaction outputs randomly.|

A post-mix wallet, due to its design, will only have one input and a maximum of two outputs at all times. Uniform indexing of outputs is necessary in order for multiple post-mix wallet implementations to look the same. A post-mix wallet SHOULD use random indexing of outputs.

Random indexing is not exclusively beneficial for post-mix wallet uniformity, conversely it has another privacy benefit. When a wallet software always generates the change output on the second index, observers always know which output is the change.

It must be mentioned [BIP69](https://github.com/bitcoin/bips/blob/master/bip-0069.mediawiki), Lexicographical Indexing of Outputs was created for the same purpose, however random indexing is slightly more private. If a blockchain observer wants to know if a transaction is in a wallet, using the BIP is a track, because it uses a deterministic algorithm, while random indexing leaves no tracks.

#### Fee Rate Estimation	
|Basic Post-Mix Wallet Requirement|Post-Mix Wallet Uniformity Requirement|
|---------------------------------|--------------------------------------|
|-|Post-mix wallet SHOULD utilize fee rate sanity check through the same web API that is used by all other post-mix wallet software.|

Blockchain analysis attempts to figure out which wallet a transaction was constructed with, is by examining the fee patterns. Therefore post-mix wallet implementations SHOULD use unified fee estimations.  

Bitcoin Core `estimatesmartfee` may differ node by node, based on how much information is available to the node. Usually if a Bitcoin wallet is not built on top of Bitcoin Core's RPC API, it either implements its own fee estimation algorithm or uses a public API.  

Post-mix wallet SHOULD utilize fee rate sanity check through the same web API that is used by all other post-mix wallet software.  
The first implementation of the post mix wallet will set precedent. This sanity check should range from Bitcoin Core's RPC's `estimatesmartfee 1 CONSERVATIVE` to `estimatesmartfee 1008 ECONOMICAL`.  
Post-mix wallet SHOULD be able to produce any integer satoshi/byte fee rate that falls between the sanity check. It can be done, for instance salting results with randomization or from the UI with a slider, where the steps are integer numbers.  
![](https://i.imgur.com/JDF48aM.png)
  
In order to avoid the identification of the transaction by timing attack, executed by the web api, post-mix wallets SHOULD retrieve sanity check from the common web API randomly from every 3 to 10 minutes.  

#### Fee Calculation
|Basic Post-Mix Wallet Requirement|Post-Mix Wallet Uniformity Requirement|
|---------------------------------|--------------------------------------|
|-|Post-mix wallet SHOULD calculate the final fee with `vsize` and make sure the final fee falls into sanity check.|  

If any post-mix wallet produces a fee that does not fall into the sanity check, the post-mix wallet can be identified by blockchain analysis.  

#### Replace-by-Fee
|Basic Post-Mix Wallet Requirement|Post-Mix Wallet Uniformity Requirement|
|---------------------------------|--------------------------------------|
||Post-mix wallet SHOULD prevent its users to utilize RBF.|

Replace-by-Fee, [RBF](https://bitcoin.org/en/glossary/rbf) is a often used feature. On the one hand its usage is beneficial, on the other hand the way RBF is used by a wallet software helps blockchain analysis to identify the wallet software in used.  
Creation of a common algorithmic utilization of RBF should be an interest of future research. Bram Cohen's [article](https://medium.com/@bramcohen/how-wallets-can-handle-transaction-fees-ff5d020d14fb) might be a good starting point.

#### Spending Unconfirmed Transactions

It is possible to spend the output of a transaction that did not confirm yet. Post-mix wallets MUST not do such thing. It leads some users to use it more often, than others.

#### Retrieving Transaction Information
|Basic Post-Mix Wallet Requirement|Post-Mix Wallet Uniformity Requirement|
|---------------------------------|--------------------------------------|
|Post-mix wallet MUST retrieve transaction information in a private way.||

Retrieving private transaction information from the Blockchain is the [most challenging](https://hackernoon.com/bitcoin-privacy-landscape-in-2017-zero-to-hero-guidelines-and-research-a10d30f1e034) part of implementing a wallet that aims to not breach its users' privacy. Querying the balances of a central server shares private information with that central server. Bloom filtering SPV wallets are [not a sufficiently private](https://groups.google.com/forum/#!msg/bitcoinj/Ys13qkTwcNg/9qxnhwnkeoIJ), either.  

There are four types of wallet architectures, ZeroLink classifies a private:    
1. **Full Nodes:** Since they download all the transactions the network has nobody can tell who is interested in what transactions.  
2. **Full Block Downloading SPV Wallets:** Such wallets download all transactions the network has from the creation of the wallet, consequently they do not need to wait weeks for [Initial Block Downloading](https://bitcoin.org/en/glossary/initial-block-download) and they do not store hundreds of gigabytes of Blockchain data. They throw away what they do not need. There are three implementations of such wallet, all in the testing phase: [Jonas Schnelli's PR to Bitcoin Core](https://github.com/bitcoin/bitcoin/pull/9483), Ádám Ficsór's [HiddenWallet](https://github.com/nopara73/HiddenWallet) and Stratis' [BreezeWallet](https://github.com/stratisproject/Breeze).
3. **[Transaction Filtered Full Block Downloading Wallet](https://medium.com/@nopara73/full-node-level-privacy-even-for-mobile-wallets-transaction-filtered-full-block-downloading-wallet-16ef1847c21):** Which only exists as an idea to date.  
4. **ZeroLink Specific Transaction Retrieval:** There is an easier and more user friendly way to achieve it: The post-mix wallet MAY accept deposits to be directly made to its addresses, without mixing. Since the input joining is disallowed there is no reason not to enable that. However if the post-mix wallet disables it, it can simply query all the Chaumian CoinJoin transactions and all its ZeroLink compliant children, since it is not interested in any other transaction. This would result in drastically better user experience, because it does not need to wait hours for Blockchain syncing.  
  
Furthermore, because  every time a CoinJoin transaction fails a new post-mix wallet output is registered, post-mix wallets SHOULD be monitored in huge depth. While it is not unlikely that an attacker ever tries to disrupt any round, because of the reasons detailed above, nevertheless a post-mix wallet is recommended to monitor 1000 clean addresses after the last used one. In this case a post-mix wallets would still show the right balances if the pre-mix wallet participates in disrupted rounds continuously for two days.  
Alternatively, if the Tumbler serves already registered, but unused addresses the post-mix wallet can use this to avoid monitoring huge depth.  

#### Transaction Broadcasting
|Basic Post-Mix Wallet Requirement|Post-Mix Wallet Uniformity Requirement|
|---------------------------------|--------------------------------------|
|Post-mix wallet MUST broadcast transactions in a private way.|Post-mix wallet SHOULD broadcast transactions over Tor through the same web API that is used by all other post-mix wallet software.|

As [Dandelion: Privacy-Preserving Transaction Propagation](https://github.com/gfanti/bips/blob/master/bip-dandelion.mediawiki) BIP candidate explains:
> Bitcoin transaction propagation does not hide the source of a transaction very well, especially against a “supernode” eavesdropper that forms a large number of outgoing connections to reachable nodes on the network. From the point of view of a supernode, the peer that relays the transaction *first* is the most likely to be the true source, or at least a close neighbor of the source. Various application-level behaviors of Bitcoin Core enable eavesdroppers to infer the peer-to-peer connection topology, which can further help identify the source.

Dandelion's explanation only applies to full nodes. Most wallet softwares are not constantly relaying transactions, for instance when the wallet software only connects to other nodes on the network to broadcast its transactions.  

ZeroLink classifies broadcasting transactions over an anonymity network to the Bitcoin network as private.  
Tus in order to fulfil Basic Post-Mix Wallet Requirement post-mix wallet MUST broadcast transactions in a private way.  
Post-mix wallet SHOULD change anonymity network indentity between every transaction broadcast.  
In order to fulfil the Post-Mix Wallet Uniformity Requirement post-mix wallet SHOULD broadcast transactions over Tor through the same web API that is used by all other post-mix wallet software. 
Post-mix wallet SHOULD broadcast every transaction on different Tor circuit.  

Private transaction broadcasting, especially Dandelion, should be an interest of future research.

#### Moving Money Between Post And Pre-Mix Wallets

The user MAY send transactions from pre-mix to post-mix wallet directly, because joining inputs are not allowed in post-mix wallets, therefore the coins will be separated.  
The user SHOULD NOT send transactions from post-mix to pre-mix wallet directly, because pre-mix wallets join inputs together. If an observer notices any connection between pre-mix coins and post-mix coins, it may re-estabilish a link in the CoinJoin transaction.

### C. Stealth Addresses

#### History

Stealth Addresses were described in detail by Peter Todd and the subject was assigned to [BIP63](https://github.com/genjix/bips/blob/master/bip-stealth.mediawiki), although it was never published in the BIP repository.
The concept was popularized by [Dark Wallet](https://github.com/darkwallet/darkwallet) which combined Stealth Addresses and coin mixing.
The Dark Wallet project ground to a halt and, despite a couple of attempts at relaunching it, it remains inactive to this day.

BIP47 Stealth Addresses were proposed by Justus Ranvier and described in [BIP47](https://github.com/bitcoin/bips/blob/master/bip-0047.mediawiki).

BIP47 Stealth Addresses differ from Dark Wallet Stealth Addresses in that both sides of a BIP47 payment channel handle address detection and synchronization rather than relying on any server-assisted Blockchain scanning. BIP47 provides the privacy advantages of Dark Wallet-style Stealth Addresses to Simplified Payment Verification and other light clients without necessitating the use of a trusted full node.

Following the publishing of the BIP, BIP47 Stealth Addresses were [implemented](https://github.com/Samourai-Wallet/samourai-wallet-android/tree/develop/app/src/main/java/com/samourai/wallet/bip47/rpc) in [Samourai Wallet](https://samouraiwallet.com) and have since gained traction through real usage with over 420 active channels having been created by privacy-seeking users.

It should be noted that Dark Wallet started work on [their own](https://github.com/darkwallet/darkwallet/commits/pcodes) BIP47 implementation during a short period in early 2016 when their project was momentarily revived. 

#### Chaumian CoinJoin And Stealth Addressing

##### Background

For the purposes of this proposal of combining Chaumian CoinJoin and Stealth Addressing, BIP47 could be used. However ZeroLink avoids adding complexity to pre-mix wallets, it aims to use existing production-ready code bases and librairies and, as such, does not want to introduce any significant overhead to the overall Chaumian CoinJoin workflow, therefore BIP47 is not part of the protocol. Neverthless it should be a topic of future research.  

BIP47 allows for the calculation of two address spaces between Alice and Bob. Alice can calculate the public keys of the addresses she will use to send transactions to Bob. In addition, Alice can calculate the private keys for the addresses which will receive transactions from Bob. The same is true for Bob vis-à-vis Alice.

There is no need to exchange or publish individual addresses, public keys or extended public keys before any transaction. In this way, the individual derived addresses will remain off the radar of Blockchain analytics and surveillance services in the event of data leak.  

Disadvantages of extended public key-based solutions are:

- Address reuse may occur if multiple pre-mix wallets are using the same extended public keys of one post-mix wallet and the Tumbler does not refuses already registered mix output addresses. Address spaces based on BIP47 payment codes can easily be kept synchronized because there are only two parties involved in any channel and transactions can be followed in lockstep.

- Extended public keys can leak and compromise privacy. Any party having knowledge of somebody else's extended public key will have complete knowledge of their transaction history and mixing balance. Payment codes provide no information about transaction amounts or addresses used between parties and can be openly distributed without concern of compromise to transactional privacy.

- In case of many continous round diruption by malicious actors, a new mix output address must be registered every time. This requires post-mix wallets to monitor the balances of its keys in huge depth.

Payment codes can be exchanged, distributed, and published without compromising the secrecy and privacy of any individual address generated from the same payment codes thereafter.

##### Application

Stealth Addresses generated from BIP47 payment codes can be used within the Wallet Privacy Framework described above.

Rather than relying on post-mix wallet extended public keys, pre-mix and post-mix wallets exchange payment codes and derive addresses on an "as needed" basis. Synchronization between wallets is simplified as address lookahead is greatly reduced.

Payment codes can be included in the Chaumian encrypted payload. In this way, the server will have no knowledge of which payment codes have been matched with each other. Note that even with knowledge of which payment codes are paired with each other, there is still no knowledge of derived individual addresses or transactions because the server never has any private keys required for address calculating.

Since BIP47 payment codes are used to derive compressed public keys, payments can be made to P2WPKH addresses. Chaumian CoinJoin transactions using BIP47 Stealth Addressing will not be identifiable on the Bitcoin Blockchain.  

Note that BIP47 [notification transactions](https://github.com/bitcoin/bips/blob/master/bip-0047.mediawiki#Notification_Transaction) can be ignored for this application. Notification transactions allow payment codes to be communicated encrypted over the Blockchain and be recoverable in the event of a wallet restore and the subsequent address rediscovery and synchronization. For this application, payment codes will be relayed within the Chaumian encrypted payload. If a post-mix wallet loses its own metadata containing the payment code and associated indexes the necessary information can be recalculated the next time, if ever, that the same payment code is received by the post-mix wallet.

##### Pseudonymous Repositories

BIP47 payment codes, being unique identifiers derived from the BIP44 wallet seed, MAY be served up pseudonymously from a [repository](https://paymentcode.io) or key store of some kind. Such services are being rolled out presently with an eye towards the development of pseudonymous payments, refunds, and mixing.
