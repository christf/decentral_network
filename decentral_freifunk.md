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

We already have an ipv6-only network and the ability to distribute multicast to every node on the net.

## Ideas

All of these ideas are designed for ipv6-only networks.

### move more exit functionality into nodes

#### Node operators should be able to chose their node to become an Exit

traffic may leave the Freifunk network at any node should their operator 
chose so. This would de-centralize the delivery of internet services.

We already have a daemon that decides which prefixes are announced. thus 
prefix-sharing should already be possible. Although it is likely broken in 
subtle ways and likely needs fixing. So in theory, this is already possible.
In practice it is untested and it is broken as long as one single node in the 
Freifunk network deploys a default route without a from stanza. The feature 
thus can only be tested with nodes that do not have a connection to existing 
networks.

#### Ability to open gre-tunnels for exiting traffic via some provider?

We already have gre kernel modules, configuration would be device specific. Why 
don't we just include the relevant gre modules in our firmware builds for the 
larger targets? This would be a quick win.

### Change topology by allowing peer-to-peer VPN connections

The nodes are organized around central nodes at the moment. This is due to the batman design of the network. Now that we have routing, we could build additional VPN links and have the routing protocol figure out the best paths. For this, there are two scenarios:

* Nodes with uplink that are part of the network should be able to establish additional VPN  links to build less of a star-based system.
* Nodes with uplink that are not part of the network should be able to find other nodes that already are.

This eliminates a single point of failure in current Freifunk networks.

#### WG-broker-Server

The wireguard broker server needs to be enhanced to be able to run on gluon to allow peer-to-peer VPN connections.

#### Linkfinder

We need a mechanism to identify possible VPN link endpoints.

### Load

At the moment the design of respondd is keeping us from having an efficient implementation of a babel statistics provider that is required for large networks. We'll have to come up with something else. See respondd.md for a discussion of these.


