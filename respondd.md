Option 2 has been implemented and is raised as PR towards the gluon project


# Respondd vs babel vs efficiency
 
The current form of the plugin concept doesn't allow long running socket connections because from the plugin there is no access to the event loop. 
The current babel plugin works, however it induces quite some load on the node when deployed on large networks this is because for its function it'll have to 
connect to babeld, dump everything babeld knows, filter through what is required and 
rework that to the json response structure.

babeld supports a monitor command which means that changes are communicated whenever thy happen in babeld. monitor also allows subscribing to specific events 
only.

## Option 1: Redesign respondd

It should be possible to keep state across plugin invocations however when 
opening a socket and processing its data whenever the plugin executes. While 
reducing CPU load a lot, during high message load, data might be lost when 
buffers are full but not read from.

To fix this, the plugin must be able to register a handler in the main loop of 
respondd and respondd must be able to monitor multiple file descriptors as part 
of that event loop.

## Option 2: Multi-threaded plugin

Register a new thread in a __attribute__(constructed) function in the plugin 
(see example in libplatformhelper). This thread could then implement it's own 
event loop, monitor the babel socket and maintain the data structures that will 
be used to reply to any queries respondd might receive.
