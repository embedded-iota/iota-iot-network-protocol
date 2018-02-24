# Specification for a network of embedded devices with IOTA as payment solution

## Status of the project
This repo contains rough specifications for a network of IOTA light wallet nodes.
These specifications are work in progress. So they can change heavily over time.

## Preconditions

### CAP
In sense of the CAP theorem, this specification prefers availability and partion tolerance over consistency. It is designed to be eventual consistent.

### Routing

This specification prefers Content Centric Networking for network purposes, but can also used with a traditional TCP/IP stack or other network-layer solutions like 6LoWPAN.

### Wording

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

## Naming Convention

The following naming convention MUST be used in the implementation. It is OPTIONAL to replace empty strings in names with underscores _

### Node

A Node SHOULD be an embedded device which MUST be SensorNode or PaymentNode. Node is the generic name for both.

#### NodeClass

The NodeClass MUST be a property on a Node and MUST be SensorNode or PaymentNode.

#### SensorNode

A SensorNode SHOULD be a small embedded device which MUST collect some data about the sourounding environment.

#### PaymentNode

A PaymentNode SHOULD be a small embedded device. It MUST accepts IOTA payments and MUST provide some service or access to goods.

#### NodeGroup

A NodeGroup MUST be group of Sensor- or PaymentNodes. They MUST fullfil the same or similar purpose. They MUST be within physical limited area.

#### NodeGroup Member

A NodeGroup Member MUST be a Node within a NodeGroup.

#### SeedNode

A SeedNode MUST be a Node within a NodeGroup. It MUST holds the configuration for new NodeGroup Members. It is also MUST be responsible for collecting the status of the NodeGroup for a Gateway.

#### WorkerNode

A WorkerNode MUST be NodeGroup Member which is not a SeedNodes.

#### NodeType

A NodeType MUST be SeedNode or WorkerNode.

#### NodeGroup Neighbor

All other Nodes MUST be called NodeGroup Neighbor.


#### NodeID

Each Node MUST have its own independent NodeID within a NodeGroup.

### Gateway

A Gateway SHOULD be a more powerful embedded device. It MUST gives a NodeGroup Member indirect access to the tangle.


### IotaAddress

An IotaAddress MUST be the address which is used on the Iota tangle.

#### IotaAddress Milestone

A IotaAddress Milestone MUST be an ID for a given Iota Address which will be set by a Gateway.

#### NodeGroup SeedKey

A NodeGroup SeedKey MUST be a shared Iota seed. Therefore every NodeGroup Member MUST use the same seed on the tangle.

### Communication

#### Gateway

##### Health Status Request

A Health Status Request MUST be a regular request which a Gateway sends. It MUST be used to get the health of a NodeGroup.

##### Status Refresh Request

A Status Refresh Request MUST be a regular request which a Gateway sends. It MUST be used to get some information about the NodeGroup.



## Functional description

### Communication

Nodes MUST communicate peer to peer within their NodeGroup. Each NodeGroup Member MUST also able communicate to any Gateway. Nodes MUST not talk to other Node outside of their NodeGroup.

#### Gateway <-> NodeGroup communication

##### Health Status Request

A Gatway SHOULD requests every x seconds the health status of a NodeGroup. If one Member doesn't respond, it MUST forces the other NodeGroup Members to get the status of this NodeGroup Neighbor. The Members MUST respond with their current saved status of this Member. If, at least, one Member responses with the status Healthy, the Member MUST NOT marked as Dead. Otherwise the Gatway SHOULD tries this process x times with a timeout of x seconds. After x times, it SHOULD marks the Member as Dead.

##### Status Refresh Request
A Gateway SHOULD requests every x seconds the status of a NodeGroup by its SeedNode. These requests MUST be called Status Refresh Requests. It MUST response one SeedNode. It is OPTIONAL that more SeedNodes responses.
The response MUST contain the following information:
- Current used addresses
- Prices of the services, sensor data or goods
- New added Member, if there were any between current status call and last one.

##### NodeGroup Member Registration

###### Add a WorkerNode to an existing NodeGroup

If a new WorkerNode gets added to an existing NodeGroup, it MUST request a list of all existing SeedNodes by a Gateway. It MUST communicate to this given SeedNodes and MUST gives them all necassary information to operate within the NodeGroup. All SeedNode MUST response with all necassary information for the new added NodeGroup WorkerNode, such as the NodeGroups Seed Key and active addresses. All SeedNodes MUST also response with a list of all existing WorkerNode of the NodeGroup.


###### Creation of a new NodeGroup

A new NodeGroup MUST be added to a Gateway. A new NodeGroup MUST NOT be created by a Node itself. The first Member of a new created NodeGroup MUST be a SeedNode. The Member of the created NodeGroup MUST request a Gateway for initial configuration. A new Iota Seed Key MUST be generated by the Member itself.

###### Add a new SeedNode

Like the creation of a new NodeGroup, a SeedNode MUST be added to a NodeGroup through a Gateway. After adding a new SeedNode to a Gateway, the new added SeedNode MUST get the initial configuration by a Gatway. It MUST also register itself to all of its NodeGroup Neighbor SeedNodes. It MUST provide all necessary information to operate within the NodeGroup. The already active SeedNodes MUST receiving the request of the new SeedNode. They MUST request a Gatway for the new NodeGroup NodeSeed list. They MUST check if the new SeedNode is in this list. If the list doesn't contain the new SeedNode, they MUST retry requesting the seed list. They SHOULD do it x times. They MUST do it with a timeout. This time SHOULD be x seconds. If the SeedNode is still not in the list, they SHOULD ignore all of the new SeedNodes requests.


#### Address transactions
A Gateway MUST get the current used addresses by the NodeGroup's SeedNode within its Status Requests. A Gateway MUST  subscribes to all known addresses. The Gatway MUST receive all of the incoming transactions to these IotaAddresses. The Gateway MUST cache these transactions. The Gateway MUST NOT forword them directly after they came in. The NodeGroup Member which is interested in an incoming transaction SHOULD request an addresses transactions since a specific Address Milestone. The response MUST contains all new transactions since the given Address Milestone. The latest transaction MUST get the current Milestone. It SHOULD request a Gateway and/or its NodeGroup Neighbors for this information. The Member which is intersted in an incoming transaction SHOULD use an timeout and SHOULD request the addresses transactions again, if it is still waiting for it. A NodeGroup Member SHOULD be also able to force its NodeGroup Neighbors to forword its request to any available Gateway. The Gateway or NodeGroup Neighbors response MUST contain the latest Address Milestone and all transactions between the provided Milestone and the latest. If the NodeGroup Member wasn't the initator of the request, it SHOULD forwords the information to the NodeGroup Member which requested the transactions and MUST updates his own transaction database.

#### SeedNode <-> SeedNode communication

##### SeedNode Gossip

Every SeedNode MUST request the SeedNode Gossip on a regular base from the other SeedNodes. It SHOULD do it every x seconds All SeedNodes SHOULD respond to this request. The SeedNode Gossip Response MUST contain the following information:
- Latest known transactions since the last milestone.
- Latest known milestone

If a SeedNode is not responding within x seconds, the SeedNode SHOULD be marked as Dead.

#### NodeGroup Member <-> NodeGroup Member communication

##### NodeGroup Member Gossip

Every NodeGroup Member MUST request the NodeGroup Member Gossip on a regular base. All NodeGroup Members SHOULD respond to this request. The NodeGroup Member Gossip MUST contains the following information:
- Latest known transactions since the last milestone.
- Latest known milestone.

If a Member is not responding within x seconds, the NodeGroup Member SHOULD be marked a Dead.



#### Receiving new addresses


### Access Data, Services or Goods

#### Receiving Payments as PaymentNode



