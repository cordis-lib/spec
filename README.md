# The spec

This repository documents the Cordis infrastructure layout - allowing anyone to implement their own version in whatever language they wish.

## Dependencies and tooling

Cordis is set of standardized micro-services that communicate with each other for various tasks.

Cordis should run on top of [RabbitMQ](https://www.rabbitmq.com/) for sending/receiving data.

Preferably, implementations should encourage containerization as the method of deployment.

## Services

Each service should be configurable via enviornment variables.

An implementation should provide the following micro-services:

### Gateway

Connecting your RabbitMQ instance to the Discord Gateway.
___

#### Configuration

The following enviornmental variables should be taken:
- CORDIS_AUTH - The Discord bot token - REQUIRED
- CORDIS_AMQP_HOST - Where the RabbitMQ instance is hosted - REQUIRED
- CORDIS_DEBUG - A toggle for enabling debug logs - default to false
- CORDIS_SHARD_COUNT - How many shards this cluster should spawn - default to Discord's recommended shard count
- CORDIS_STARTING_SHARD - The first shard that this cluster should spawn - default to 0
- CORDIS_TOTAL_SHARD_COUNT - How many shards you intend to have across all clusters - default to Discord's recommended shard count
- CORDIS_REDIS_URL - URL for Redis. If provided, your implementation should use Redis for storage of rate limiting and gateway information
- CORDIS_WS_COMPRESS - Wether or not WebSocket compression should be used - default to true
- CORDIS_WS_ENCODING - What encoding to use for the gateway connection ("json"/"etf") - default to "etf"
- CORDIS_WS_OPEN_TIMEOUT
- CORDIS_WS_HELLO_TIMEOUT
- CORDIS_WS_READY_TIMEOUT
- CORDIS_WS_GUILD_TIMEOUT
- CORDIS_WS_RECONNECT_TIMEOUT
- CORDIS_WS_LARGE_THRESHOLD - https://discord.com/developers/docs/topics/gateway#identify-identify-structure
- CORDIS_WS_INTENTS - Comma seperated strings reflecting what intents should be used. Should also support and expand: "all", meaning all intents, "privileged", meaning all privileged intents, "nonPrivileged", meaning all non-privleged intents.
___

#### Behaviour

First and foremost, your implementation should ensure a successful connection to the RabbitMQ server before continuing.

You should now establish a connection to Discord's Gateway and start processing the packets that come through.

Your implementation should *absolutely*, under no circumstances, *implicitly* cache anything.

You implemention should *absolutely*, under no circumstances, make any HTTP requests but the required `/gateway/bot` call.

#### Outgoing data

No packets but opcode 0 (Dispatch) should make their way out of the gateway service.

You should output Discord data as it comes in - no mutations.

You should publish those packets to an exchange named `gateway`, using the packet's `t` property as the routing key.

#### Incoming data & communicating with the gateway

You should consume the `gateway_commands` fanout exchange by binding it to a randomly generated queue and send the data coming through *as is* to every shard under your cluster.

To summarize, your user's workers should be able to publish a packet to `gateway_commands`, and it should make it to Discord from every single shard.
