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

The following enviornmental variables should be used:
- AUTH - The Discord bot token - REQUIRED
- AMQP_HOST - Where the RabbitMQ instance is hosted - REQUIRED
- Everything else is left to user dis
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
