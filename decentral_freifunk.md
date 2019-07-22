# Decentralizing Freifunk

## Motivation

Besides being a community, Freifunk is a de-central wlan mesh network. Of 
course the traffic from each node is routed via central gateways to central 
providers using infrastructure that must be managed centrally.

Central infrastructure has advantages and drawbacks. The drawbacks are:

* the infrastructure must match what is in the firmware. This split causes problems:

  * different technologies are used. This makes it hard to maintain the network as a whole.
  * the packaging for mesh protocol must be in sync for multiple platforms. This is a nuissance

* At a scale of a few hundred nodes, the infrastructure in the data center is 
  already quite specialized. This makes it difficult to contribute individual 
  resources and effort. It is very difficult to run a community project when
  contribution is not easily possible.


## current state

### Nodes

To form a network of the current Freifunk stack, a star-like topology is 
created by using central VPN gateways. These will accept VPN connections from 
nodes and offload traffic to some exit. This could be called a centralized 
design.


### Exits

Exits are part of the freifunk network. Exits route traffic from the Freifunk 
network towards the internet. For this they expose a default route. Upstream is 
connected either via yet another VPN or via a Freifunk Provider or via direct 
exit.


## future state

### Nodes

Nodes in the future are like current, however the ability to expose a default 
route is added.

### Exits

Exits are dedicated to handling the uplink connection where traffic is 
offloaded. Exits are not part of the Freifunk network.


## Implications

* traffic may leave the Freifunk network at any node should their operator 
  chose so.
* source-specific routing must be used on all nodes

## Implementation

### Prefixd

We already have a daemon that decides which prefixes are announced. thus 
prefix-sharing should already be possible. Although it is likely broken in 
subtle ways


## Outlook

### move more exit functionality into nodes

* Ability to open gre-tunnels for exiting traffic via some provider?

### Change topology by allowing peer-to-peer VPN connections

The nodes are organized around central nodes at the moment. This is due to the batman design of the network. Now that we have routing, we could build additional VPN links and have the routing protocol figure out the best paths. For this, there are two scenarios:

* Nodes with uplink that are part of the network should be able to establish additional VPN  links to build less of a star-based system.
* Nodes with uplink that are not part of the network should be able to find other nodes that already are.
