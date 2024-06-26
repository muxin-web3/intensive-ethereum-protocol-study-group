# Ray
Hi, I am ray, I know a bit about ethereum, and I am trying to become an expert in it.

- Twitter: https://twitter.com/rayjun0412
- tg: https://t.me/rayjun0412

## Notes

### 2024.4.3

Trying to understand POS more deeply

### 2024.4.5

*以太坊的 PoS 是什么，是如何工作的*

以太坊通过 The Merge 升级，将共识机制从工作量证明（PoW）转成了权益证明（PoS）。 如果想参与到以太坊的网络，需要质押 32 ETH（这个数额有可能会调整）成为 Validator。

Validator 负责处理交易、创建新的区块，验证现有的区块，以太坊的 PoS 协议是 Gasper，来自 Casper （权益证明算法）和LMD GHOST（分叉选择算法）的结合。

如果想要成为网络的 Validator，需要先到一个智能合约中质押 ETH，然后可以参与到网络新区块的创建、区块的验证，并在其中留下自己的签名，其他人从签名中获取地址，通过这个智能合约中的来检验 Validator 的身份，*Validator 的签名就是 Proof of Stake*。

如果 Validator 作恶，那么罚没质押的资金（不同的作恶情节有不同额度的罚没），如果被罚没之后资金量少于 16 ETH，那么就会被提出网络。。

区块链的共识其实就是让链的高度可以持续增加。有两种方式：
- 最长链模型
- BFT

而以太坊的 Casper 协议结合了这两种方式，在后续，将尝试用最简单的语言将 PoS 的机制解释清楚。

PoS 比 PoW 更节省能源，也更适合以太坊长远的规划。但是我觉得 PoS 算法太复杂了，没有 PoW 那么优雅，这是一个技术上的妥协，而且我认为 PoS 最大缺陷在于 permissionless 上比 PoW 要差，如果是 PoW 机制，任何人可以在任意时间加入网络开始挖矿，但是对于 PoS 来说，需要先质押 ETH， 然后才能参与网络，而质押 ETH 是一笔交易，需要先完成执行，然后上链。所以如果有很多人参与网络或者退出网络，通常需要有一个排队的过程。

### 2024.4.6
最长链模型是一个很经典的共识方法，由于网络的原因，链可能会出现分叉，每一个区块的创建者都可以在不同的分叉上构建新的区块，这其实是一个投票的过程，最终能够成为最长链的分叉将成为新的共识。

<img src="./img/ray/longestchain.png" height="50%" width="50%" />

但是在以太坊中的最长链模型稍微有一些不一样，看下面的这种情况，产生了两个分叉，上面一个分叉在同一条链上，新产生了两个区块，下面的分叉有三个区块，但是三个区块都是新的分叉，如果按照传统的分叉模型，那么上面的分叉才是最长链。但是在以太坊中，得到更多 Validator 投票的分叉才会被认为是最长链 会认为下面的分叉得到的票数多，会把下面的分叉当做是最长链，新的区块会被添加到其中一个分叉上。

<img src="./img/ray/longestchain2.png" height="50%" width="50%" />

最长链的产生过程需要 Validator 投票，当前以太坊的新区块产生时间是 12s，是一个 slot，每 32 个 slot 是一个 epoch。每一个 epoch 可以看作是一轮 BFT 投票。BFT 投票通常是在验证者之间进行两轮投票，如果一方在两轮之后获得 2/3 的投票，那么就获得了最终的投票。

<img src="./img/ray/bft.png" height="50%" width="50%" />

在第一轮中，验证者承诺投票，在第二轮中实际进行投票。这个投票必须进行两轮，如果只有一轮，攻击者会同时做出不透的投票，造成网络的分裂。在以太坊的投票中，第一轮投票之后，每个人都做出了投票，并且攻击者同时给双方投票，导致双方都超过了 67% 的投票，那么这些攻击者就将会因为双重投票而罚没质押的 ETH，如果低于 16 ETH，就会被踢出网络。

<img src="./img/ray/vote.png" height="50%" width="50%" />

还有一种情况是验证者不参与投票，导致验证者的投票率不足，这种情况下，验证者也会因为不活跃而被罚没 ETH。

<img src="./img/ray/vote2.png)" height="50%" width="50%" />

在极端的情况下，如果有超过 1/3 的节点下线，那么以太坊可以继续是用 GHOST 算法继续运行，而不会让网络崩溃。

ref: https://www.youtube.com/watch?v=5gfNUVmX3Es


### 2024.4.7

当前以太坊网络中的质押者接近 100 万，如果这些验证者同时对区块进行随机签名投票，那么网络中就会产生大量数据，整个网络可能会变得混乱以及不稳定。所以需要按照一定的规则将这些验证者组织起来。

每一个区块对应一个 slot，一个 slot 代表 12s，32 个 slot 组成一个 epoch。

在每个 epoch 中，所有在只能合约中质押的验证者都会被随机分成 committee，每个 committee 都会负责 epoch 中的一个 slot。一个 committee 会划分成 128 个子网，每个子网中至少 128 个 Validator，当前以太坊有近 100 万的 Validator，每个 slot 的 validator 超过了 3 万。每个 epoch 会提前两个 epoch 周期（13 分钟）分配完成，Validator 有充足的时间找到分配给他们的 slot。

Validator 的分配是基于一个 randao 的分配算法，rando 依赖 validator 的私钥签名来产生随机数，这里会是用 BLS 技术将所有的签名汇集到一起，最后产生一个随机数。只要至少有一个诚实的 validator，那么最后产生的结果就是随机的。

committee 中会随机一个 validator 来创建一个区块，这个 Validator 称之为 block proposer，其他的 validator 来证明这个区块是最新的区块。每个 epoch 中，每个 validator 只能证明一个区块，否则会被罚没 ETH。

Validator 的投票很重要，如果使用的是 PoW 协议，区块是很难被伪造的，直接是用最长链的原则就可以解决共识分叉的问题。转用 PoS 之后，情况就会变得复杂一些，因为区块很容易被伪造，那么最长链的判断依据就是 Validator 的投票。如果某个时间段内的 block proposer 没有创建新的区块，这个 committee 中其他 validator 只投票支持前一个区块。

在如此庞大的网络中传递 Validator 的签名依然是一个很大的负担，所以 以太坊是用 BLS 技术来减少签名的数据量。
