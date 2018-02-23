# Specification for a network of nodes with IOTA as payment solution

**Status of the project:**
This repo contains rough specifications for a network of IOTA light wallet nodes.
These specifications are work in progress. So they can change heavily over time.

# Preconditions

## CAP
In sense of the CAP theorem this specification prefers availability and partion tolerance over consistency. It is designed to be eventual consistent.

## Routing

This specification prefers Content Centric Networking for routing purposes, but can also used with a traditional TCP/IP stack or other network-layer solutions like 6LoWPAN.

# Naming Convention

## Node

A Node is a embedded device which can be SensorNode or PaymentNode. Node is the generic name for both.

## NodeClass

The NodeClass can be SensorNode or PaymentNode.

## SensorNode

A SensorNode is a small embedded device which collect some data about the sourounding environment. This could be the temperature, as well as the traffic of a specific street.

## PaymentNode

A PaymentNode is a small embedded device which accepts IOTA payments and provides some service or goods. This could be a car park barrier, as well as a snack automat.

## NodeGroup

A NodeGroup is a group of Sensor- or PaymentNodes which fullfil the same purpose in a given area.

## NodeGroup Member

A NodeGroup Member is a Node within a NodeGroup.

## SeedNode

A SeedNode is a Node within a NodeGroup which holds the configuration for new NodeGroup Members. It is also responsible for collecting the status of the NodeGroup for a Gateway.

## WorkerNode

A WorkerNode are all Members of a NodeGroup which are not SeedNodes.

## NodeType

A NodeType can be SeedNode or WorkerNode.

## NodeGroup Neighbor

All other Nodes within a NodeGroup are the Nodes Neighbors.

##### Examples

**Temperature sensors:** Temperature sensors within a given area. This could be a district of a city or postcode area. These sensors provide information which are equal from the outside perspective. (temperature of a given area)

## NodeID

Each node has its own independent NodeID within a NodeGroup.

## Gateway

A Gateway is a more powerful embedded device, which gives the nodes indirect access to the tangle.


## Address Milestone

A Address Milestone is an ID for a given Iota address which will be set by a Gateway. A Gateway sets this ID on every [X]th transaction.

## NodeGroup Seed Key

A NodeGroup Seed key is a shared Iota seed. Therefore every NodeGroup Member uses the same seed on the tangle.



# Function description

## Communication

Nodes can communicate peer to peer within their NodeGroup. Each node can also communicate to any Gateway. Nodes cannot talk to other Node outside of their NodeGroup.

### Health Status Request

A Gatway requests every x seconds the health status of a NodeGroup. If one Member doesn't respond, it forces the other Members to get the status of this Member. The Members will respond with their current status of this Member. If, at least, one Member responses with the status Healthy, the Member will not marked as Dead. Otherwise the Gatway tries this process x times with a timeout of x seconds. After x times, it marks the Member as Dead.

### Status Refresh Request
A Gateway requests every x seconds the status of a NodeGroup by its SeedNode. These requests are called Status Refresh Requests.
The response of one or more SeedNode contains the following information:
- Current used addresses
- Prices of the services, sensor data or goods
- New added Member, if there were any between current status call and last one.


### Address transactions
A Gateway gets the current used addresses by one or more SeedNode within its Status Requests. A Gateway subscribes to all known addresses and gets therefore all incoming transactions. The Gateway caches these transactions, but doesn't forword them directly after they came in. The NodeGroup Member which is interested in an incoming transaction can request an addresses transactions since a specific Address Milestone. The response contains all new transactions since the given Address Milestone. The latest transaction gets the current Milestone. It can request a Gateway and/or its NodeGroup Neighbors for this information. The Member which is intersted in an incoming transaction should use an timeout and should request the addresses transactions again, if it is still waiting for it. A NodeGroup Member is also able to force its NodeGroup Neighbors to forword its request to any available Gateway. The Gateway or NodeGroup Neighbors response must contain the latest Address Milestone and all transactions between the provided Milestone and the latest. If the NodeGroup Member wasn't the initator of the request, it forwords the information to the NodeGroup Member which requested the transactions and updates his own transaction database.

### SeedNode Gossip

Every SeedNode requests the newest Gossip every x seconds from the other SeedNodes. All SeedNodes must respond to this request. The SeedNode Gossip Response contains the following information:
- Latest known transactions since the last milestone.
- Latest known milestone

If a SeedNode is not responding within x seconds, the SeedNode will marked as Dead.

### NodeGroup Gossip

Every NodeGroup Member requests every x seconds for the newest Gossip. All NodeGroup Members must respond to this request. The NodeGroup Gossip contains the following information:
- Latest known transactions since the last milestone.
- Latest known milestone.

If a Member is not responding within x seconds, the NodeGroup Member will marked a Dead.

### Node Registration

#### Add a Member to an existing NodeGroup

If a new Member gets added to an existing NodeGroup, it requests a list of all existing SeedNodes by a Gateway. It communicate to this given SeedNodes and gives them all necassary information to operate within the NodeGroup. All SeedNode responses with all necassary information for the new added NodeGroup Member, such as the NodeGroups Seed Key and active addresses. All SeedNodes must also response with a list of all existing Members of the NodeGroup.


#### Creation of a new NodeGroup

A new NodeGroup needs to be added to a Gateway. A new NodeGroup can not be created by a Node itself. The first Member of a new created NodeGroup must be a SeedNode. The Member of the created NodeGroup can request a Gateway for initial configuration. A new Iota seed key will be generated by the Member itself, not a Gateway.

#### Add a new SeedNode

Like the creation of a new NodeGroup, a SeedNode can only added to a NodeGroup through a Gateway. After adding a new SeedNode to a Gateway, the new added SeedNode must register itself to its NodeGroup Neighbor SeedNodes. It must provide all necessary information to operate within the NodeGroup. The already active SeedNodes which receiving the request of the new SeedNode will request a Gatway for the new NodeGroup NodeSeed list. They will check if the new SeedNode is valid. If the list doesn't contain the new SeedNode, they retry requesting the new SeedNode x times, with a timeout of x seconds. If the SeedNode is still not in the list, they will ignore all of the new SeedNodes requests.


### Receiving new addresses


## Access Data, Services or Goods

### Receiving Payments as PaymentNode



