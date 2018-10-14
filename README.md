# Specification for a network of embedded devices with IOTA as payment solution

## Preconditions

### Wording

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", 
and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

### CAP
In sense of the CAP theorem, this specification prefers availability and partion tolerance over consistency. 
It is designed to be eventual consistent.

### Routing

This specification prefers Content Centric Networking for network purposes, 
but can also used with a traditional TCP/IP stack or other network-layer solutions like 6LoWPAN.

### Usage of the Specifications

This Specification is UNSTABLE. It SHALL not be used for production implementations.

## Naming Convention

The following naming convention SHALL be used in the implementation. 
The naming convention within the implementation MUST be CamelCase or Underscore _.


### Node

A Node MUST be one physical device. A Node SHOULD be an embedded device.

#### Properties

###### Node Class

The value SHALL be Sensor Node or Payment Node.
Every Node MUST have a Node Class.

###### Node Type

The value SHALL be Seed Node, Worker Node or Temporary Node.
Every Node MUST have a Node Type.

###### Node ID

Each Node MUST have its own Node ID.
The value of the Node ID SHALL be unique within a Node Group.

#### Other

###### Sensor Node

A Sensor Node SHOULD be a small embedded device.
It MUST collect any data about the surrounding environment.

###### Payment Node

A Payment Node SHOULD be a small embedded device. 
A Payment Node is REQUIRED to send or/and accept IOTA payments.
It SHOULD give access to services or goods.

###### Area Width

Every Node Group MUST have a Area Width.
The Area Width MUST be the longest distance between two Nodes within a Node Group.
The Area Width MUST be a circle.

###### Node Group

A Node Group MUST be group of Sensor- or Payment Nodes. 
They MUST fulfil the same or similar purpose. 
All Nodes within a Node Group MUST be in an Area Width.

###### Node Group Member

A Node Group Member SHALL be the name of a Node within a Node Group.

###### Seed Node

A Seed Node MUST be a Node within a Node Group. 
It MUST holds the configuration for new Node Group Members. 
It is also MUST be responsible for collecting the status of the Node Group for an IOTA Gateway.

###### Worker Node

A Worker Node MUST be Node Group Member which is not a Seed Nodes.

###### Temporary Node

A Temporary Node MUST be temporary member in a node group. 
It SHALL not be a member for longer than x minutes.


###### Node Group Neighbor

All other Nodes within a Node Group SHALL be called Node Group Neighbor.

### IOTA Gateway

A IOTA Gateway SHOULD be a more powerful embedded device. 
A IOTA Gateway SHALL give a Node Group Member indirect access or direct to the tangle.

### Group Status Codes

###### HEALTHY

The Status HEALTHY MUST return by a Seed Node, if everything operates in the specified way.

###### DEAD

The Status DEAD MUST return by a Seed Node if all Node Group Members are DEAD.

###### PARTIALLY_BROKEN

The Status PARTIALLY_BROKEN MUST return by a Seed Node, if the status of, at least one, Node Group Member is BROKEN or DEAD.

###### QUORUM_BROKEN

The Status QUORUM_BROKEN MUST return by a Seed Node, if the status of a quourum count of Node Group Members is BROKEN.

###### QUORUM_DEAD

The Status QUORUM_DEAD MUST return by a Seed Node, if the status of a quourum count of Node Group Members is DEAD. 
QUORUM_DEAD MUST overrule the status QUORUM_BROKEN.


### Node Status Codes

###### HEALTHY

The Status HEALTHY MUST return by Node, if everything operates in the specified way.

###### DEAD

The Status DEAD MUST return by a Nodes Node Group Neighbors for a Node Group Member, 
if they cannot contact this Node Group Member directly or indirect anymore.

###### BROKEN

The Status BROKEN MUST return by a Node, if the Node is still available but not able to fulfil its purpose.


### IOTA Tangle Network

###### IOTA Address

An IOTA Address MUST be the ternary char presentation of an address which is used on the IOTA tangle.
Once generated and IOTA address MUST be saved persistent on a Node.

###### IOTA Transaction

An IOTA Transaction MUST be a transaction on the IOTA tangle. It SHOULD be a confirmed transaction. 
A IOTA Transaction MUST, at least, contain the following properties:
- Tag
- Message

###### IOTA Address Milestone

An IOTA Address Milestone MUST be an ID for a given IOTA Address which SHOULD be set by a IOTA Gateway. 
It SHALL NOT be set by a Node Group Member.
An IOTA Address Milestone SHALL be only contain A-Z & 0-9.
The IOTA Gateway MUST save the IOTA Address Milestones persistently.

###### Seed Key

A Seed Key is the private IOTA Seed which is used as base for generating IOTA addresses, transactions etc. 
It is also known just as seed in the IOTA specifications.

### Communication

#### IOTA Gateway

###### Health Status Request
 
A Health Status Request MUST be used by a IOTA Gateway to get the health of a Node Group.
An IOTA Gateway MUST send the Health Status Request in an interval.

###### Health Status Response

The Health Status Response MUST contain the following information:
- Status: BROKEN, HEALTHY or DEAD

###### Status Refresh Request

A Status Refresh Request MUST be used by a IOTA Gateway to get information about the Node Group.
An IOTA Gateway MUST send the Status Refresh Request in an interval.

###### Status Refresh Response

The Status Refresh Response MUST contain the following information:
- Current used addresses
- Prices of the services, sensor data or goods
- New added Member, if there were any between current status call and last one.


#### Nodes

###### IOTA Address Transactions Update Request

An IOTA Address Transactions Update Request SHALL be send by a Node which is interested in a new transaction. 
It MUST , at least, contain the following information:
- Latest Milestone

## Functional description

### Communication

Nodes SHALL communicate peer to peer within their Node Group. 
Nodes within one Node Group SHALL NOT be able to communicate with Members of another Node Group. 
Each Node Group Member MUST also be able communicate to one or more IOTA Gateway.

##### IOTA Gateway <-> IOTA Tangle

An IOTA Gateway MUST subscribe to the transaction feed of all known IOTA Addresses. 
The IOTA Gateway MUST receive all incoming transactions of these IOTA Addresses. 
The IOTA Gateway MUST cache these transactions.
The IOTA Gateway SHOULD delete the entry in the cache after a quorum of Node Group Members received the transaction.
An IOTA Gateway SHALL NOT forward these transactions directly to the Node Group. 
It MUST wait for a Address Transactions Update Request.

##### IOTA Gateway <-> Node Group Member communication

###### Health Status Requests

An IOTA Gateway MUST request the Health Status in an interval.
The IOTA Gateway SHOULD request every Node Group Member for the Health Status.
If this Member does not respond, it SHOULD forces the other Node Group Members to get the status of this Node Group Neighbor. 
The Members SHOULD respond with their current saved status of this Node Group Member. 
If, at least, one Node Group Member responses with the status Healthy, the Member SHOULD NOT marked as Dead. 
If no Node Group Member responses with a Health status, the IOTA Gateway SHOULD retry the Health Status Request. 
The retry SHOULD have a timeout and it SHOULD be tried several times before the Node Group Member will marked as Dead.

###### Status Refresh Requests

An IOTA Gateway SHOULD requests every x seconds the status of a Node Group by its Seed Node. 
These requests MUST be called Status Refresh Requests. 
It MUST response one Seed Node. 
It is OPTIONAL that more Seed Nodes responses.


#### Seed Node <-> Seed Node communication

###### Seed Node Gossip

Every Seed Node MUST request the Seed Node Gossip in an interval from the other Seed Nodes. 
It SHOULD do it every x seconds. 
The Node which requests the newest Seed Node Gossip selects a random Seed Node Neighbor. 
This Seed Node Neighbor SHOULD respond to this request. 
If it does not respond to this request, SHOULD be marked as DEAD. 
The Seed Node Gossip Response MUST contain the following information:
- Latest known transactions since the last milestone.
- Latest known milestone
- Its own status
- New added Worker Nodes (a new Worker Node SHOULD be deleted from this response after x hours.)

If a Seed Node detected an issue on its side, it MUST response with a BROKEN status within the Seed Node Gossip.

The SeedNodes SHOULD retry contacting the not responding Seed Node. They SHOULD do it several times with a timeout.
The Seed Node which is not responding MUST mark, eventually, as DEAD if it is not responding.

#### Node Group Member <-> Node Group Member communication

###### Node Group Member Gossip

Every Node Group Member MUST request the Node Group Member Gossip in an interval. 
A Node which requests the Node Group Member Gossip MUST select a Node Group Neighbor randomly. 
This Node Group Neighbor SHOULD respond to this request. 
If it does not respond, it SHOULD be marked as DEAD. 
The Node Group Member Gossip MUST contains the following information:
- Latest known transactions since the last milestone.
- Latest known milestone.
- Its own status

If the Member detected an issue on its side, it MUST response with a BROKEN status within the Node Group Member Gossip.

The Node Group Members SHOULD retry contacting the not responding Node Member. 
They SHOULD do it several times with a timeout. 
The Node Group Member which is not responding MUST mark, eventually, as DEAD if it is not responding.

#### Worker Node <-> Seed Node communication

###### Worker Node Gossip

Every Worker Node MUST request a Seed Node in an interval. 
The Seed Node MUST selected randomly. 
The Seed Node SHOULD respond. 
If the Seed Node does not respond, 
the Worker Node MUST select a new Seed Node randomly and request the new selected Seed Node.
The response of a Seed Node MUST contain the following information:
- New Worker Nodes (a new Worker Node SHOULD be deleted from the list after x hours.)


#### Node Group Member Registration

###### Add a Worker Node to an existing Node Group

If a new Worker Node gets added to an existing Node Group, it MUST request a list of all existing Seed Nodes by a IOTA Gateway.
It MUST communicate to this given Seed Nodes and MUST gives them all necessary information to operate within the Node Group. 
At least one Seed Node MUST response with all necassary information for the new added Node Group Worker Node.
At least one Seed Node MUST also response with a list of all existing Worker Node of the Node Group.


###### Creation of a new Node Group

A new Node Group MUST be added to a IOTA Gateway. A new Node Group MUST NOT be created by a Node itself. 
The first Member of a new created Node Group MUST be a Seed Node. 
The Member of the created Node Group MUST request a IOTA Gateway for initial configuration.

###### Add a new Seed Node

Like the creation of a new Node Group, a Seed Node MUST be added to a Node Group through an IOTA Gateway. 
After adding a new Seed Node to a IOTA Gateway, the new added Seed Node MUST get the initial configuration by a IOTA Gateway. 
It MUST also register itself to all of its Node Group Neighbor Seed Nodes. 
It MUST provide all necessary information to operate within the Node Group. 
The already active Seed Nodes MUST receiving the request of the new Seed Node. 
They MUST request a IOTA Gateway for the new Node Group NodeSeed list. 
They MUST check if the new Seed Node is in this list. 
If the list does not contain the new Seed Node, they MUST retry requesting the seed list. 
They SHOULD do it x times. They SHOULD do it with a timeout. 
This time SHOULD be x seconds. 
If the Seed Node is still not in the list, they SHOULD ignore all of the new Seed Nodes requests.

#### Seed Keys

A Seed Key MUST generated by the Node itself. 
It SHALL NOT share this Seed with other Nodes. 
It MUST keep this Seed Key secure in its storage. 
Seed Keys MUST be saved persistent.
It MUST also keep track of the current used IOTA Address' index.

#### Generation of IOTA Addresses

Since a Seed Key is used to generate IOTA Addresses, the IOTA Address MUST be generated on the Node.
The Node SHALL NOT share the index of the IOTA Address with other Nodes.
Generation of IOTA Addresses in chunks is OPTIONAL.


#### Receiving an IOTA Address' transactions

The Node Group Member which is interested in an incoming transaction 
MUST provide an Address Milestone. 
The response MUST include all transactions since the given Transaction Milestone. 
The response MUST contains all new available transactions since the given Address' Milestone. 
The latest transaction SHALL be marked with the latest Milestone. 
To get the IOTA Addresses' transactions, the Node SHOULD request a IOTA Gateway. 
It is OPTIONAL to also request the Node Group Neighbors. 
The Member which is waiting for a transaction MUST request the transactions in an interval. 
The interval time SHOULD be higher than 100 ms.
A Node Group Member SHOULD also be able to force its Node Group Neighbors to forword its request to any IOTA Gateway. 
The IOTA Gateway or Node Group Neighbors response 
MUST contain the latest Address Milestone and all transactions between the provided Milestone and the latest. 
If the Node Group Member was not the initiator of the request, 
it MUST forewords the information to the Node Group Member which requested the transactions and 
MUST updates its own transaction database.


#### Receiving new IOTA Addresses

The IOTA Gateway SHALL receive new IOTA Addresses by the Node Group Members.

#### Promotion of an IOTA transaction

The IOTA Gateway MUST take care of promoting an IOTA transactions created by a Node.

### Reattachment of an IOTA transaction

The IOTA Gateway MUST take care of reattaching an IOTA transaction created by a Node.

### Proof of Work

The IOTA Gateway MUST take care of the Proof of Work for an IOTA transaction created by a Node.

### Trunk- & Branch-Transaction

The IOTA Gateway MUST take care of picking the Trunk- & Branch-Transaction-Bundles.

### Access Data, Services or Goods

#### Receiving Payments as Payment Node



