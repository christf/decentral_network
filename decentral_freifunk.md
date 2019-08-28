# Decentralizing Freifunk

## Motivation

Besides being a community, Freifunk is a de-central wlan mesh network. 
At the moment, the traffic from each node is routed via central gateways to central 
providers using infrastructure that must be managed centrally.

Central infrastructure has advantages and drawbacks. The drawbacks are:

* the infrastructure must match what is in the firmware. This split causes problems:

  * different technologies are used. This makes it hard to maintain the network as a whole.
  * the packaging for mesh protocol must be in sync for multiple platforms. This is a nuisance

* At a scale of a few hundred nodes, the infrastructure in the data center is 
  already quite specialized (virtual machines are replaced by physical ones, 
  when becoming ISP router hardware is used that holds the full table, managing 
  peerings and failover is required). This makes it difficult to contribute 
  individual resources and effort. It is very difficult to run a community 
  project when contribution is not easily possible.

We already have an ipv6-only network and the ability to distribute multicast to every node on the net.

## Ideas

All of these ideas are designed for ipv6-only networks and will move 
functionality from the data center into nodes and thus requiring less and less centralized infrastructure.

### Node operators should be able to chose their node to become an Exit

Traffic may leave the Freifunk network at any node should their operator 
chose so. This would de-centralize the delivery of internet services.

We already have a daemon that decides which prefixes are announced. thus 
prefix-sharing should already be possible. Although it is likely broken in 
subtle ways and likely needs fixing. So in theory, this is already possible.
In practice it is untested and it is broken as long as one single node in the 
Freifunk network deploys a default route without a from stanza. The feature 
thus can only be tested with nodes that do not have a connection to existing 
networks.

Nodes providing this capability will be referred to as _exit-nodes_.

### Ability to open gre-tunnels for exiting traffic via some provider?

We already have gre kernel modules, configuration would be device specific. Why 
don't we just include the relevant gre modules in our firmware builds for the 
larger targets? This would be a quick win.

### Change topology by allowing peer-to-peer VPN connections

The nodes are organized around central nodes at the moment. This is due to the batman design of the network. Now that we have routing, we could build additional VPN links and have the routing protocol figure out the best paths. For this, there are two scenarios:

* Nodes with uplink that are part of the network should be able to establish
  additional VPN links to build less of a star-based system.
* Nodes with uplink that are not part of the network should be able to find
  other nodes that already are.

This eliminates a single point of failure in current Freifunk networks: the gateways.


#### Assumptions

Assuming we have unconnected wlan mesh clouds.

* For each mesh clouds we have at least one node that provides uplink capabilities. This node can and must establish VPN connections to allow connectivity between mesh clouds. These uplink-nodes will use Linkfinder to learn a viable VPN endpoint. Each uplink-enabled node will create multiple VPN connections. At least one of them should target an exit-node.
* more than 90% of the traffic inside a Freifunk network is routed towards the internet. This means the exit-nodes will see 90% of the traffic.
* All uplink-nodes should have the capability to become an exit-node. We are  assuming that only a fraction of the regular uplink-nodes will actually be exits. Instead most exits will run in a data-center somewhere, much similar to the current gateways
* The exit-nodes are tightly-meshed via VPN. When the exit-functionality breaks down to 
  whatever reason, the traffic should be re-routed via another exit with no additional hops.


#### WG-broker-Server

The wireguard broker server needs to be enhanced to be able to run on gluon to allow peer-to-peer VPN connections.

#### Linkfinder

This is the tricky bit. We need a mechanism to identify possible VPN link endpoints.

##### Option 1: Query the Map

Every uplink-node could query the map for the topology of the existing network using its WAN-interface and then use that information to determine one exit-node to connect to. After this the local mesh-cloud becomes part of the network and is able to establish more VPN links as required.

This has the drawback of establishing the central map server as single point of failure. For a system of that criticality, its software stack is more complex than what I would like.

##### Option 2: query a DHT

A DHT is used in peer to peer networks to store data. We could query this DHT for possible exit-nodes. Kadnode implements DNS-over-DHT and could be re-used.


##### Option 3: use DNS to keep a list of exit-nodes.

DNS is fairly simple and does not require additional software so it seems like 
a good fit if the following question can be answered:

_How to keep the data about exit-nodes current in a de-centralized way?_

One way to do this is to maintain the records using a git repository. This 
however would require the users wanting to install an exit-node to raise PR via 
git. It likely will not happen. Really the trigger should be setting the 
exit-node-option on the device itself.

##### Option 4: Dead-Man-Switch via http

Each node that is capable of being a VPN endpoint will query a certain URL on a 
regular basis. From the source-IP-addresses of these queries, the server will 
compile a list of existing exit-nodes. When a nodes does not send a query for a 
while, it will be removed from that list. That list could be downloaded via 
http by nodes who attempt to join the network.

##### Option 5: <your idea here>

What else could we do?


# Terminology

## Exit-Node

A Freifunk-Router that allows traffic to exit the Freifunk-Network through its WAN-interface.
There it may be routed through a VPN or exit directly.

## Uplink-Node

A Freifunk-Router that has at least one VPN connection to maintain its connectivity to the Freifunk network in addition to WIFI/LAN-mesh.

## Mesh-only-Node

A Freifunk-Router that is neither Exit- nor Uplink-node.

