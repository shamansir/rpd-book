## Network

<img src="./assets/rpd-network.png" width="600px" alt="Network"></img>

_Network_ defines a system of Patches. At this level Patch may be considered as a complex procedure with several inputs and outputs and a Network is a program that uses these procedures.

Say, you define some complex functionality of a 3D vertex shader with a Patch. It has its inputs such as 3D position, color and texture coordinate of a vertex and outputs as 2D coordinate and modified color. Then you may create another “root” Patch, which allows user to apply combinations of different vertex shaders to an imported object and so re-use the Shader Patch several times in different configurations. These two Patches (first one used several times and second one used once) form a simple Network.

<!-- TODO: diagram of a vertex shader network -->

So, some Node in one Patch may represent the inputs and outputs of another Patch's copy, running in the same Network.

