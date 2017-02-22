## Inlet

<img src="./assets/inlets.png" width="160px" alt="Inlets"></img>

_Inlet_ is a socket of a Node designed to receive incoming data. Depending on configuration, it may accept only one connection from an Outlet or any number of connections from several Outlets.

Inlet may have a default value, so it sends it to the Node when the latter is added to a Patch.

Inlet may be hidden, so it won't be visible for user, but may receive data from inputs located in the body of the Node.

Inlet may be “cold”, in contrary to “hot” by default, so it won't trigger the modifying process, but save the last value that came to this Inlet.

Inlets may accept or deny values depending on their type. Type may specify a function which will transform data before it will be sent to the Node.