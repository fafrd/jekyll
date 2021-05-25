---
layout: post
title:  "My work with Curio Cards"
date:   2021-05-25 15:10:01 -0700
tags: ethereum curiocards
---

Curio Cards is an early [NFT](https://ethereum.org/en/nft/#what-are-nfts) project on the Ethereum blockchain. Travis ([@travisformayor](https://twitter.com/travisformayor)), Mad Bitcoins ([@madbitcoins](https://twitter.com/MadBitcoins)) and Rhett Creighton launched the project.

## 2017

When this was launched in 2017, there was no equivalent of a _browser-based_ wallet such as MetaMask; instead, I helped by forking the popular MyEtherWallet website, changing the theme and monitoring the addresses of the 30 Curio Cards.

The card addresses were pulled in dynamically from a google sheet, allowing us to deploy new cards without needing to update the frontend. See my change: [github.com/fafrd/etherwallet/compare/3f57cd0..3d7769a](https://github.com/fafrd/etherwallet/compare/3f57cd0..3d7769a)

## 2021
### Wrapper contract and frontend

In 2021, a resurgence in the popularity of NFTs brought renewed interest in Curio Cards. Originally, Curio Cards were implemented as a basic ERC-20 contract, as they were launched before modern NFT standards. To be interoperable with NFT marketplaces such as [OpenSea](https://opensea.io/), I worked with Moon ([@moon@shitposter.club](https://shitposter.club/users/Moon)) to develop an ERC-1155 wrapper. Moon handled the Solidity implementation and I handled the frontend interface. I had familiarity with React but chose to use vanilla JS to better understand whether a framework was necessary. (It certainly would have helped!)

Website: [wrap.curio.cards](https://wrap.curio.cards)<br />
Repository: [github.com/fafrd/curio-wrapper-site](https://github.com/fafrd/curio-wrapper-site)

### Weighted voting methodology

At the time of development, there was another wrapper contract put out by a different development team. Unfortunately the OpenSea listing for this other wrapper placed a fee on all trades directed to the devs. To differentiate our wrapper contract we created a governance poll to ask the community how to implement the fee structure.

Voting weight was determined by the number of Curio Cards held, with each card weighted according to its relative scarcity- holding a small number of rare cards would have a similar voting power as holding a large number of common cards. Voting was done through [Snapshot](https://snapshot.org/). I created a new snapshot [_voting strategy_](https://docs.snapshot.org/strategies) to calculate each user's voting power. This was my first foray into Typescript.

Community vote: [governance.curio.cards](https://governance.curio.cards/#/curiovoters.eth/proposal/QmU61U2bux8io9VrVKGd9pmSfn7v5WDQ1o9EooSGe3qYhu)<br />
Voting strategy implementation: [github.com/snapshot-labs/snapshot.js/pull/144](https://github.com/snapshot-labs/snapshot.js/pull/144/files)

### Leaderboard website

To gamify collection of Curio Cards and incentivize use of the wrapper, I created a leaderboard website to show off the top card holders. The website was written as a simple React page.

Website: [leaderboard.curio.cards](https://leaderboard.curio.cards)<br />
Repository: [github.com/fafrd/curio-collector-leaderboard](https://github.com/fafrd/curio-collector-leaderboard)

The interesting part was the API I needed. The ERC-1155 contract doesn't provide a reverse mapping to determine who holds the cards, so I needed a way to index past Curio Card transfers. I ended up writing an index as a [Graph Protocol](https://thegraph.com/) _subgraph_. This is a piece of typescript code that, for each Ethereum block, finds Curio Card transfers and creates an index of who holds which cards. This can then be sorted by the number of unique cards held, providing an easy backend API for the leaderboard.

Subgraph repository: [github.com/fafrd/curio-cards-subgraph](https://github.com/fafrd/curio-cards-subgraph)

### 0x limit order exchange fork

Before the wrapper was completed, we still wanted a way to facilitate trading the ERC-20 Curio Cards. Liquidity pools didn't make sense here, so I forked a 0x starter kit frontend to allow limit orders.

Website: [buycuriocards.com](https://buycuriocards.com/#/erc20)<br />
Fork changes: [github.com/fafrd/curio-exchange/compare/44272c9..b70bb61](https://github.com/fafrd/curio-exchange/compare/44272c9..b70bb61)
