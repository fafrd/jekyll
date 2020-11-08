---
layout: post
title:  "Querying KeeperDAO transactions on the blockchain"
date:   2020-11-08 16:33:24 -0500
tags: ethereum keeperdao
---
KeeperDAO is a simple ethereum contract that allows other contracts to execute a [flash loan](https://finematics.com/flash-loans-explained/) as part of transaction. This allows actors, known as _keepers_, to take advantage of [arbitrage](https://en.wikipedia.org/wiki/Arbitrage) opportunities. You can read more about KeeperDAO [here](https://github.com/keeperdao/docs/wiki).

This blog post shows how to query the Ethereum blockchain to examine past arbitrage transactions utilizing KeeperDAO, so we can better understand what actors exist and how to query for smart contract events generally.

*TL;DR- [skip to full code below](#full-code-from-start-to-finish)*

# Setting up our environment

This guide will use javascript, but we could use pretty much any language to query Ethereum.

First, let's install the necessary packages:
{% highlight bash %}
npm install web3-eth node-rest-client-promise
{% endhighlight %}

And start an interactive javascript shell using node:
{% highlight bash %}
$ node
Welcome to Node.js v14.1.0.
Type ".help" for more information.
>
{% endhighlight %}

Connect to an ethereum node. You can run your own using software like [OpenEthereum](https://github.com/openethereum/openethereum) or [geth](https://github.com/ethereum/go-ethereum), or you can connect to a publicly-available node, such as one provided by [Infura](https://infura.io/product/ethereum). This example will point to a locally-running node on port 8545.
{% highlight javascript %}
var Eth = require('web3-eth');
var restClient = require('node-rest-client-promise').Client();

// Ethereum node connection
var eth = new Eth('http://localhost:8545')
{% endhighlight %}

We need to fetch the KeeperDAO contract interface (also known as the [ABI](https://solidity.readthedocs.io/en/latest/abi-spec.html)) to query for past contract interactions. The easiest way to do this is to get it from Etherscan.

You'll need to create an account on [Etherscan](https://etherscan.io/), then create a key to use their api.
{% highlight javascript %}
// Etherscan connection, to fetch contract ABI
var etherscanApiKey = 'YOUR_API_KEY' // replace this with a key from your etherscan.io account
var contractAddr = '0xEB7e15B4E38CbEE57a98204D05999C3230d36348' // KeeperDAO contract
var esurl = `http://api.etherscan.io/api?module=contract&action=getabi&address=${contractAddr}&apikey=${etherscanApiKey}`
{% endhighlight %}

# Setting up the query
Now we'll create some functions to ask the ethereum node to find all KeeperDAO contract events since block 11000000.
{% highlight javascript %}
// Get contract interface from Etherscan
async function getContractAbi() {
    const es_response = await restClient.getPromise(esurl);
    const abi = JSON.parse(es_response.data.result);
    return abi;
}

// Callback for event query
var allEvents = null;
function handle(events) {
    allEvents = events;
    console.log('Event query complete.');
}

// Query ethereum node for past contract interactions
async function eventQuery(){
    var abi = await getContractAbi();
    var contract = new eth.Contract(abi, contractAddr);

    //var start = 10418479; // first borrow?
    var start = 11000000; // oct 6. use a more recent block # if the query is taking too long
    var end = 'latest';
    return contract.getPastEvents("allEvents", { fromBlock: start, toBlock: end });
}
{% endhighlight %}

# Performing the query
Now that we're set up, we can perform the query. This will take some time to run, and will print `Event query complete` when it is complete.
{% highlight javascript %}
eventQuery().then(events => handle(events)).catch((err) => console.error(err));
{% endhighlight %}

When the query completes, the variable `allEvents` will be populated. You can take a peek at the contents by running `allEvents[0]`

# Printing the results
Wait for the previous command to complete. It'll probably take 30 seconds or so.

Then, print all the borrow events in CSV format:
{% highlight javascript %}
var counter = 0;
console.log("event,block,tx,caller")
for (const event of allEvents) {
    if (event.event == 'Borrowed') {
        console.log(counter++ + "," + event.blockNumber + "," + event.transactionHash + "," + event.returnValues[0]);
    }
}
{% endhighlight %}

Example output: [download keeperdao.csv](/assets/keeperdao.csv)

# Full code, from start to finish
{% highlight javascript %}
var Eth = require('web3-eth');
var restClient = require('node-rest-client-promise').Client();

// Ethereum node connection
var eth = new Eth('http://localhost:8545')

// Etherscan connection, to fetch contract ABI
var etherscanApiKey = 'YOUR_API_KEY' // replace this with a key from your etherscan.io account
var contractAddr = '0xEB7e15B4E38CbEE57a98204D05999C3230d36348' // KeeperDAO contract
var esurl = `http://api.etherscan.io/api?module=contract&action=getabi&address=${contractAddr}&apikey=${etherscanApiKey}`

// Get contract interface from Etherscan
async function getContractAbi() {
    const es_response = await restClient.getPromise(esurl);
    const abi = JSON.parse(es_response.data.result);
    return abi;
}

// Callback for event query
var allEvents = null;
function handle(events) {
    allEvents = events;
    console.log('Event query complete.');
}

// Query ethereum node for past contract interactions
async function eventQuery(){
    var abi = await getContractAbi();
    var contract = new eth.Contract(abi, contractAddr);

    //var start = 10418479; // first borrow?
    var start = 11000000; // oct 6. use a more recent block # if the query is taking too long
    var end = 'latest';
    return contract.getPastEvents("allEvents", { fromBlock: start, toBlock: end });
}

eventQuery().then(events => handle(events)).catch((err) => console.error(err));

// wait for eventQuery to complete...

var counter = 0;
console.log("event,block,tx,caller")
for (const event of allEvents) {
    if (event.event == 'Borrowed') {
        console.log(counter++ + "," + event.blockNumber + "," + event.transactionHash + "," + event.returnValues[0]);
    }
}
{% endhighlight %}
