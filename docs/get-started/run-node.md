# Running a TomoChain full node

With the recent release of TomoChain Testnet 2.0 with our PoSV consensus, you might be interested in creating a TomoChain full node and applying it to be a masternode.

To run a full node and meet the requirements to apply on our governance DApp, you have to run two services:

- The [TomoChain client](https://github.com/tomochain/tomochain), our TomoChain implementation written in _Go_.
- [Telegraf](https://github.com/influxdata/telegraf), an agent to collect performance metrics of your full node.

We provide different way of running a node to assure flexibility in integration depending on your current infrastructure.

## General hardware notice

Our team extensively tested performances and came up with those minimal requirements for any TomoChain masternode host:

- Must be facing internet directly (no NAT, public IP)
- Must have at least 8 cores
- Must have at least 32GB of RAM
- Must use an IaaS ("cloud") provider of your choice (AWS, Digital Ocean, Google Cloud, etc.)
- Storage must be SSD

The full node will serve on port `30303` for p2p communication with other nodes, `8545` for RPC api and `8546` for websocket api.
You may need to edit your firewall configuration accordingly.

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

## Docker images

### Prerequisites

To use tmn, you should meet this requirements in addition of the hardware ones:

- [Docker CE](https://docs.docker.com/install/)

### Usage

You can use our docker images with vanilla docker, docker-compose or any orchestrator of your choice.
You will need to run our two docker images: `tomochain/infra-tomochain` and `tomochain/telegraf`.

### Running the tomochain image

To run a full node, you need to provide those configurations through environment variables:

`IDENTITY`: The unique identity of your full node.
The identity should be of this format: `$NAME_$IDENTIFIER`.
Please use letters, numbers, dashes ("-") and underscore ("\_") only in the name.
The identifier should be a 6 character hexadecimal string.
You can generate one easily with the command `uuidgen | cut -c -6`

`PRIVATE_KEY`: The private key of the account that your full node will use.
A tomochain full node use an account to be uniquely identified and to receive rewards.

`BOOTNODES`: The bootnodes specific to the network you want to connect to.
They should be separated by commas without any whitespace. <!-- TODO: bootnode address? -->

`NETSTATS_HOST`: The netstats endpoint specific to the network you want to connect to.
For Testnet, it's `stats.testnet.tomochain.com`.

`NETSTATS_PORT`: The port of the netstats endpoint.
For Testnet, it's `443`. <!-- TODO: remove? and set 443 as default -->

It is also strongly recommended to use a named volume to store the blockchain data.
This way, the node will not need to re-download the whole blockchain on every restart.
The blockchain data is located in `/tomochain/data`.

Here is an example of running a tomochain container configured to use our Testnet:
```
docker run -p 30303:30303 \
    -v chaindata:/tomochain/data \
    -e IDENTITY=companyabcd_30D37E \
    -e BOOTNODES=enode://4d3c2cc0ce7135c1778c6f1cfda623ab44b4b6db55289543d48ecfde7d7111fd420c42174a9f2fea511a04cf6eac4ec69b4456bfaaae0e5bd236107d3172b013@52.221.28.223:30301,enode://298780104303fcdb37a84c5702ebd9ec660971629f68a933fd91f7350c54eea0e294b0857f1fd2e8dba2869fcc36b83e6de553c386cf4ff26f19672955d9f312@13.251.101.216:30301,enode://46dba3a8721c589bede3c134d755eb1a38ae7c5a4c69249b8317c55adc8d46a369f98b06514ecec4b4ff150712085176818d18f59a9e6311a52dbe68cff5b2ae@13.250.94.232:30301 \
    -e NETSTATS_HOST=stats.testnet.tomochain.com \
    -e NETSTATS_PORT=443 tomochain/infra-tomochain:latest
```

### Running the telegraf image

You also need to run the telegraf image to provide the required metrics to our governance system.

You need to provide these configuration through environment variable:

`METRICS_ENDPOINT`: The metrics endpoint specific to the network you want to connect to.
For testnet, it's `https://metrics.testnet.tomochain.com`.

You will also need to mount different repertories of your host system to let telegraf get the metrics:

`/var/run/docker.sock` -> `/var/run/docker.sock`

`/sys` -> `/rootfs/sys`

`/proc` -> `/rootfs/proc`

`/etc` -> `/rootfs/etc`

Here is an example of running a telegraf container configured to use our Testnet: <!-- TODO update images! infra-telegraf:devnet -> telegraf:latest -->
```
docker run \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    -v /sys:/rootfs/sys:ro -v /proc:/rootfs/proc:ro \
    -v /etc:/rootfs/etc:ro \
    -e METRICS_ENDPOINT=https://metrics.testnet.tomochain.com tomochain/telegraf:latest
```

## Binaries

You can also run directly the tomo binary.
Please refer to the tomochain repository [readme](https://github.com/tomochain/tomochain).
You will also need to run the telegraf docker image.
You can refer to the "Docker images" part of this document to that effect.