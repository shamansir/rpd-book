<!-- MARK: Patch -->

### `Patch`

Patch contains a set of nodes and could be rendered on its own _canvas_, which is an invisible boundind box where this patch is drawn.

Nodes are connected with links going from outputs of one node to inputs of another. This way data dynamically flows through the patch.

<!-- schematic picture of a patch -->

#### `patch.render(renderers, targets, config)`

Render this Patch with given Renderers, to given target elements, with provided configuration.

`renderers` is one or the list of the Renderer aliases, like `'html'` or `'svg'`, which come out of the box and render Patch to HTML  or SVG elements, correspondingly.

`targets` is one or the list of elements where patch should be rendered. When you specify two targets, patch will be mirrored in both targets and changes in one target will be reflected in another and vice versa.

`conf` is the rendering configuration, described in details in [Network section](./network.html#rendering-configuration). It allows you to select Style of the Nodes and configure a lot of other useful things.

```html
<body>
    <div id="target-1"></div>
    <div id="target-2"></div>
    <div id="target-3"></div>
</body>
```

```javascript
var patchOne = Rpd.addPatch('FirstPatch')
                  .render('svg', [ 'target-1', 'target-2' ], {
                      style: 'compact-v',
                      linkForm: 'curve'
                  });
var patchTwo = Rpd.addPatch('SecondPatch')
                  .render([ 'html', 'svg' ], 'target-3', {
                      style: 'quartz',
                      linkForm: 'line'
                  });                  
```

NB: Note that _closed_ Patches are not rendered immediately, unlike _opened_ ones. To get more details on _opening_ and _closing_ Patches, see [Rpd.addClosedPatch()](#rpd-addclosedpatch) description.

#### `patch.addNode(type, title, [definition]) → Node`

Add a node, which represents any process over some inputs (inlets) and sends result of the process to its outputs (outlets). A node can have no inputs or no outputs at all, or even both, so in the latter case this node is called self-sufficient.

The type of the node is some previously registered type, for example, `core/basic`. Usually it has the form `toolkit/short-name`. You may use a prepared one from the [toolkits](TODO) or easily create your own types for the nodes with [`Rpd.nodetype`](TODO).

You may specify a custom title, if you want, or the engine will fall back to the type name.

The third argument, `definition` is a bit tricky one, but just a bit. It's optional, so usually you may omit it without any compunction. This argument is the object which actually has exactly the same structure as the object used for `Rpd.nodetype`. It helps you to override the type definition for this particular node instance, when you want. <!-- Test it merges definitions, not overrides everything -->

NB: When you override inlets and outlets this way, you may later access them using `node.inlets[alias]` and `node.outlets[alias]` shortcuts, same way as when you defined them with `Rpd.nodetype`. The inlets and outlets added later with `node.addInlet` and `node.addOutlet` methods are not accessible with this shortcuts, that is, I hope, rather logical.

You can discover the complete list of the properties which could be used in this definition if you follow to [Node Definition](#node-definition) section. Also, note that when you need, you may pass a function to this method, it is very useful when you need to share some objects between definitions, so both examples are valid:

```javascript
patch.addNode('docs/foo', 'Foo', {
    inlets: ...,
    outlets: ...,
    process: ...
});

patch.addNode('docs/foo', 'Foo', function() {
    var someSharedVariable;
    var nodeInstance = this;
    return {
        inlets: ...,
        outlets: ...,
        process: ...
    };
});
```

#### `patch.removeNode(node)`

Remove the previously added node, just pass the one you need no more.

#### `patch.open()`

Opening the Patch triggers it to be put into the rendering flow, so it listens for all the following actions and renders them accordingly. If Patch yet has no canvas to be drawn onto, engine adds this canvas to the root element before.

All the Patches are opened by default, unless they were added with [`Rpd.addClosedPatch`](#rpd-addclosedpatch) method.

Opening and closing Patches helps when you have a complex network and you want to isolate some parts of it by moving them in the background. So, you may add the patches you want to hide with[`Rpd.addClosedPatch`](#rpd-addclosedpatch) and open them later (or not open them at all). Also, you may create a special Node which refers to some closed Patch, passes data inside and then takes the processed data in return. Then, if you want, you may add a button to this node, which, in its turn, opens this, currently closed, Patch. This approach is described in details together with the [`patch.project(node)`](#patch-project) method below.

#### `patch.close()`

Closing the Patch means that the canvas of this Patch is hidden and moved to the background, so user sees no process happening there. Currently the rendering still goes there, yet staying invisible, but in the future versions it meant to be cached and reduced to the latest changes before opening instead.

#### `patch.project(node)`

Make given node to visually represent current patch<!-- projectOn, projectTo, referenceWith ?-->. It is expected, but not required, for this node to be located in another patch. <!-- TODO: test -->

It helps a lot when you have some complex network with a single large patch and so you probably want to group some nodes and reference them in another patch, while making invisible what happens inside. By _projecting_ a patch into the node, with the help of [`patch.inputs`](#patch-inputs) and [`patch.outputs`](#patch-outputs), you can "pack" any part of the network in one single node, and, optionally, let user to take a look inside or even edit, reconnect or rearrange the internals.

```javascript
Rpd.renderNext('html', document.body, {
    fullPage: true;
});

// Prepare Root patch

var rootPatch = Rpd.addPatch('Root');

var genANode = rootPatch.addNode('core/basic', 'Generate A');
var genAOutlet = genANode.addOutlet('util/number', 'A');
genAOutlet.send(3);

var genBNode = rootPatch.addNode('core/basic', 'Generate B');
var genBOutlet = genA.addOutlet('util/number', 'B');
genBOutlet.send(1);

var rootSumOfThreeNode = rootPatch.addNode('util/sum-of-three', 'foo')
rootSumOfThreeNode.addInlet('util/number', 'D');
rootSumOfThreeNode.addInlet('util/number', 'E');
rootSumOfThreeNode.addOutlet('util/number', 'F');

// Prepare Procedure Patch

var sumPatch = Rpd.addClosedPatch('Sum Procedure');

var sumOfThree1Node = sumPatch.addNode('util/sum-of-three', 'Sum1');
var in1AInlet = sumOfThree1Node.inlets['a'];
var in1BInlet = sumOfThree1Node.inlets['b'];
var sum1Outlet = sumOfThree1Node.outlets['sum'];

var sumOfThree2Node = sumPatch.addNode('util/sum-of-three', 'Sum2');
var in2AInlet = sumOfThree2Node.inlets['a'];
var sum2Outlet = sumOfThree2Node.outlets['sum'];

sum1Outlet.connect(in2AInlet);

sumPatch.inputs([ in1AInlet, in1BInlet ]);
sumPatch.outputs([ sum1Outlet, sum2Outlet ]);

var projectionNode = rootPatch.addNode('core/reference', '[Sum Patch]');
sumPatch.project(projectionNode);

outAOutlet.connect(projectionNode.inlets['a']);
outBOutlet.connect(rootSumOfThreeNode.inlets['a']);
outBOutlet.connect(rootSumOfThreeNode.inlets['b']);
rootSumOfThreeNode.outlets['sum'].connect(projectionNode.inlets['b']);
```

#### `patch.inputs(inlets)`

Specify which inlets, no matter from one or different nodes, are the global inputs of this patch. See description of the [`patch.project`](#patch-project) for details.

#### `patch.outputs(outlets)`

Specify which outlets, no matter from one or different nodes, are the global outputs of this patch. See description of the [`patch.project`](#patch-project) for details.

#### `patch.moveCanvas(x, y)`

Move the canvas of the patch to given position, treated relatively to the root element's top left corner. Both parameters are just numbers, treated as pixels for `html` rendering, and as units, for `svg` rendering.

#### `patch.resizeCanvas(width, height)`

Resize the canvas of the patch. This means all the visuals belonging to this patch and happened to be outside of given bounds, become hidden. Both parameters are just numbers, treated as pixels for `html` rendering, and as units, for `svg` rendering.