# Sequencers

Before diving in, make sure to check out the ["Understanding Starknet: Sequencers, Provers, and Nodes"](https://book.starknet.io/chapter\_3/topology.html) chapter for a quick rundown of Starknet’s architecture.

Three main layers exist in blockchain: data availability, ordering, and execution. Sequencers have evolved within this evolving modular landscape of blockchain technology. Most L1 blockchains, like Ethereum, handle all these tasks. Initially, blockchains served as distributed virtual machines focused on organizing and executing transactions. Even roll-ups running on Ethereum today often centralize sequencing (ordering) and execution while relying on Ethereum for data availability. This is the current state of Starknet, which uses Ethereum for data availability and a centralized Sequencer for ordering and execution. However, it is possible to decentralize sequencing and execution, as Starknet is doing.

Each of these layers plays a crucial role in achieving consensus. First, the data must be available. Second, it needs to be put in a specific order. That’s the main job of a Sequencer, whether run by a single computer or a decentralized protocol. Lastly, you execute transactions in the order they’ve been sequenced. This final step, done by the Sequencer too, determines the system’s current state and keeps all connected clients on the same page.

## Introduction to Sequencers

The advent of Layer Two (L2) solutions like Roll-Ups has altered the blockchain landscape, improving scalability and efficiency. But what about transaction order? Is it still managed by the base layer (L1), or is an external system involved? Enter Sequencers. They ensure transactions are in the correct order, regardless of whether they’re managed by L1 or another system.

In essence, sequencing has two core tasks: sequencing (ordering) and executing (validation). First, it orders transactions, determining the canonical sequence of blocks for a given chain fork. It then appends new blocks to this sequence. Second, it executes these transactions, updating the system’s state based on a given function.

To clarify, we see sequencing as the act of taking a group of unordered transactions and producing an ordered block. Sequencers also confirm the resulting state of the machine. However, the approach explained here separates these tasks. While some systems handle both ordering and state validation simultaneously, we advocate for treating them as distinct steps.

![Sequencer role in the Starknet network](../img/ch03-sequencer.png)

Sequencer role in the Starknet network

## Sequencers in Starknet

Let’s delve into Sequencers by focusing on [Madara](https://github.com/keep-starknet-strange/madara) and [Kraken](https://github.com/lambdaclass/starknet\_stack/tree/main/sequencer), two high-performance Starknet Sequencers. A Sequencer must, at least, do two things: order and execute transactions.

* **Ordering**: Madara handles the sequencing process, supporting methods from simple FCFS and PGA to complex ones like Narwhall & Bullshark. It also manages the mempool, a critical data structure that holds unconfirmed transactions. Developers can choose the consensus protocol through Madara’s use of Substrate, which offers multiple built-in options.
* **Execution**: Madara lets you choose between two execution crates: [Blockifier](https://github.com/starkware-libs/blockifier/tree/main) and [Starknet\_in\_Rust](https://github.com/lambdaclass/starknet\_in\_rust). Both use the [Cairo VM](https://github.com/lambdaclass/cairo-vm) for their framework.

We also have the Kraken Sequencer as another option.

* **Ordering**: It employs Narwhall & Bullshark for mempool management. You can choose from multiple consensus methods, like Bullshark, Tendermint, or Hotstuff.
* **Execution**: Runs on Starknet\_in\_Rust. Execution can be deferred to either [Cairo Native](https://github.com/lambdaclass/cairo\_native) or [Cairo VM](https://github.com/lambdaclass/cairo-vm).

| Feature                 | [Madara](https://github.com/keep-starknet-strange/madara)                                | [Kraken](https://github.com/lambdaclass/starknet\_stack/tree/main/sequencer)                                        |
| ----------------------- | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Ordering Method**     | FCFS, PGA, Narwhall & Bullshark                                                          | Narwhall & Bullshark                                                                                                |
| **Mempool Management**  | Managed by Madara                                                                        | Managed using Narwhall & Bullshark                                                                                  |
| **Consensus Options**   | Developer’s choice through Substrate                                                     | Bullshark, Tendermint or Hotstuff                                                                                   |
| **Execution Crates**    | [Blockifier](https://github.com/starkware-libs/blockifier/tree/main), Starknet\_in\_rust | Starknet\_in\_rust                                                                                                  |
| **Execution Framework** | [Cairo VM](https://github.com/lambdaclass/cairo-vm)                                      | [Cairo Native](https://github.com/lambdaclass/cairo\_native) or [Cairo VM](https://github.com/lambdaclass/cairo-vm) |

### Understanding the Execution Layer

* [Blockifier](https://github.com/starkware-libs/blockifier/tree/main), a Rust component in Starknet Sequencers, generates state diffs and blocks. It uses [Cairo VM](https://github.com/lambdaclass/cairo-vm). Its goal is to become a full Starknet Sequencer.
* Starknet\_in\_Rust is another Rust component for Starknet that also generates state diffs and blocks. It uses [Cairo VM](https://github.com/lambdaclass/cairo-vm).
* [Cairo Native](https://github.com/lambdaclass/cairo\_native) stands out by converting Cairo’s Sierra code to MLIR. See an example [here](https://github.com/lambdaclass/cairo\_native/blob/main/examples/erc20.rs).

## The Need for Decentralized Sequencers

For more details on the Decentralization of Starknet, refer to the dedicated subchapter in this Chapter.

Proving transactions doesn’t required to be decentralized (although in the near future Starknet will operate with decentralized provers). Once the order is set, anyone can submit a proof; it’s either correct or not. However, the process that determines this order should be decentralized to maintain a blockchain’s original qualities.

In the context of Ethereum’s Layer 1 (L1), Sequencers can be likened to Ethereum validators. They are responsible for creating and broadcasting blocks. This role is divided under the concept of "Proposer-Builder Separation" (PBS) ([Hasu, 2023](https://www.youtube.com/watch?v=6xS0xMzh9Tc)). Block builders form blocks (order the transactions), while block proposers, unaware of the block’s content, choose the most profitable one. This separation prevents transaction censorship at the protocol level. Currently, most Layer 2 (L2) Sequencers, including Starknet, perform both roles, which can create issues.

The drive toward centralized Sequencers mainly stems from performance issues like high costs and poor user experience on Ethereum for both data storage and transaction ordering. The challenge is scalability: how to expand without sacrificing decentralization. Opting for centralization risks turning the blockchain monopolistic, negating its unique advantages like network-effect services without monopoly.

With centralization, blockchain loses its core principles: credible neutrality and resistance to monopolization. What’s wrong with a centralized system? It raises the risks of censorship (via transaction reordering).

A centralized validity roll-up looks like this:

* User Interaction & Selection: Users send transactions to a centralized Sequencer, which selects and orders them.
* Block Formation: The Sequencer packages these ordered transactions into a block.
* Proof & Verification: The block is sent to a proving service, which generates a proof and posts it to Layer 1 (L1) for verification.
* Verification: Once verified on L1, the transactions are considered finalized and integrated into the L1 blockchain.

![Centralized rollup](../img/ch03-centralized-rollup.png)

Centralized rollup

While centralized roll-ups can provide L1 security, they come with a significant downside: the risk of censorship. Hence, the push for decentralization in roll-ups.

## Conclusion

This chapter has dissected the role of Sequencers in the complex ecosystem of blockchain technology, focusing on Starknet’s current state and future directions. Sequencers essentially serve two main functions: ordering transactions and executing them. While these tasks may seem straightforward, they are pivotal in achieving network consensus and ensuring security.

Given the evolving modular architecture of blockchain—with distinct layers for data availability, transaction ordering, and execution—Sequencers provide a crucial link. Their role gains more significance in the context of Layer 2 solutions, where achieving scalability without sacrificing decentralization is a pressing concern.

In Starknet, Sequencers like Madara and Kraken demonstrate the potential of high-performance, customizable solutions. These Sequencers allow for a range of ordering methods and execution frameworks, proving that there’s room for innovation even within seemingly rigid structures.

The discussion on "Proposer-Builder Separation" (PBS) highlights the need for role specialization to maintain a system’s integrity and thwart transaction censorship. This becomes especially crucial when we recognize that the current model of many L2 Sequencers, Starknet included, performs both proposing and building, potentially exposing the network to vulnerabilities.

To reiterate, Sequencers aren’t just a mechanism for transaction ordering and execution; they are a linchpin in blockchain’s decentralized ethos. Whether centralized or decentralized, Sequencers must strike a delicate balance between scalability, efficiency, and the overarching principle of decentralization.

As blockchain technology continues to mature, it’s worth keeping an eye on how the role of Sequencers evolves. They hold the potential to either strengthen or weaken the unique advantages that make blockchain technology so revolutionary.

The Book is a community-driven effort created for the community.

* If you’ve learned something, or not, please take a moment to provide feedback through [this 3-question survey](https://a.sprig.com/WTRtdlh2VUlja09lfnNpZDo4MTQyYTlmMy03NzdkLTQ0NDEtOTBiZC01ZjAyNDU0ZDgxMzU=).
* If you discover any errors or have additional suggestions, don’t hesitate to open an [issue on our GitHub repository](https://github.com/starknet-edu/starknetbook/issues).
