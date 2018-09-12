# Running a TomoChain full node

With the recent release of TomoChain Testnet 2.0 with our PoSV consensus, you might be interested in creating a TomoChain full node and applying it to be a masternode.

To run a full node and meet the requirements to apply on our governance DApp, you have to run two services:

- The [TomoChain client](https://github.com/tomochain/tomochain), our TomoChain implementation written in _Go_.
- [Telegraf](https://github.com/influxdata/telegraf), an agent to collect performance metrics of your full node.

## General hardware notice

Our team extensively tested performances and came up with those minimal requirements for any TomoChain masternode host.

**Testnet**

- Must be facing internet directly (no NAT, public IP)
- Must have at least 2 cores
- Must have at least 8GB of RAM
- Must use an IaaS ("cloud") provider of your choice (AWS, Digital Ocean, Google Cloud, etc.).
- Storage must be SSD

**Mainnet**

- Must be facing internet directly (no NAT, public IP)
- Must have at least 8 cores
- Must have at least 32GB of RAM
- Must use an IaaS ("cloud") provider of your choice (AWS, Digital Ocean, Google Cloud, etc.)
- Storage must be SSD

The full node will serve on port `30303` udp and tcp for p2p communication with other nodes, `8545` tcp for RPC api and `8546` tcp for websocket api.
You may need to edit your firewall configuration accordingly.

If you have any other production grade environment than cloud providers at your displosal, please tell us more about it on our [Gitter](https://gitter.im/tomochain).

## tmn

We made a simple command line interface called [tmn](https://github.com/tomochain/masternode) to easily and quickly start a TomoChain masternode.
It takes care of starting the necessary docker containers with the proper settings for you.
It will really suits you if you don't already have a big infrastructure running.
Spin up a machine in your favorite cloud and get your masternode running in a few minutes!

### Prerequisites

To use tmn, you should meet this requirements in addition of the hardware ones:

- [Docker CE](https://docs.docker.com/install/)
- [Python](https://docs.python-guide.org/starting/install3/linux/) >= 3.6

### Installation

Simply install it from pip.

```
pip3 install --user tmn
```

### Update

Update it from pip.

```
pip3 install -U tmn
```

### First start

When you first start your full node with tmn, you need to give some informations.

`--name`: The name of your full node.
It should be formatted as a slug string.
Slug format authorize all letters and numbers, dashes ("-") and underscores ("\_").
You can name it to reflect your identity, company name, etc.

`--net`: The network your full node will connect to.
You can chose here to connect it to the TomoChain Testnet or Mainnet (once launched).

`--pkey`: The private key of the account that your full node will use.
A tomochain full node use an account to be uniquely identified and to receive transaction fee.

It could look like this:

```
tmn start --name [YOUR_NODE_NAME] \
    --net testnet \
    --pkey [YOUR_COINBASE_PRIVATE_KEY]
```

Once started, you should see your node on the [stats page](https://stats.testnet.tomochain.com)!

Note: it can take up to 15 minutes to properly end the first synchronization.

### Usage

You can now interact with it via the other commands:

`stop`: Stop your full node.

`start`: Start your full node if it was stopped.

`status`: The current status of your full node.

`inspect`: Display the details related to your full node.
Useful for applying your full node as a masternode.

`remove`: Completely remove your masternode, unique identity and data.
