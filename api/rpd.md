<!-- MARK: Rpd -->

### `Rpd`

The `Rpd` namespace is a single entry point for your _patch network_, independently on the place where every patch should be rendered. It provides you with the ability to append new patches to your own network and <!-- scurpolously --> control the overall rendering process.

Every patch lays over its own _canvas_, several canvases may be attached to the same _target element_, this will be covered in details below.

> It's important to notice that the whole API is based on processing event streams, same way as Reactive Programming concepts work. All the created instances are immutable, they only react on user actions, with no modifications to the initial state. It guarantees the safety and ability to reverse any operation and also allows you to create streams of data of any complexity, intended to flow between the nodes.

<!-- schematic picture of a network -->

Directly in `Rpd.` namespace there are methods designed to help you register new Node Types, new Channel Types, Node Renderers and Channel Renderers. They help you to build and reuse any kinds of Nodes and Channels, actually there is no limit of what you can do using these. Every method is described below or in some section nearby.

When you find that you use registration methods from `Rpd.*` namedpace too often,  please consider extracting these parts to a separate [Toolkit](./toolkit.html).

<!-- From this point and below, let's consider some example to illustrate the practical usage of the described methods. Say, we want to draw the Solar System in static (not that RPD is unable to do it in dynamic, but it's better to consider simple examples at start, isn't it?). We won't do it step-by-step like tutorials do, rather we'll say which method fits particular situation better. For these needs, for every API method there will be a section marked as _Example_. If you really want, the complete code of this example is accessible [here] --> <!-- TODO -->

<!-- schematic picture of an example -->

#### `Rpd.renderNext(renderers, targets, config)`

Render all the Patches which follow this statement with specified Renderer (or many), to specified target (or many), with specified configuration (just one is enough).

`renderer` is an alias of a Renderer, registered with `Rpd.renderer` before. RPD comes with `'html'` and `'svg'` Renderers. Not every Style may render with both, so there's a table in [Setup](../setup.html) section clarifying which Style supports which Renderer.

`target` is any DOM (i.e. HTML or SVG) Element to be a root for the next Patches to render into.

`config` is the configuration object used both by Style and Renderer.

The process is described in details in [Network](../network.html#rendering-configuration) section.

#### `Rpd.stopRendering()`

Stop all the rendering processes, running for the moment. Even if other RPD methods will be called until next call to `Rpd.renderNext` or `patch.render`, they won't show anything in UI.

#### `Rpd.addPatch([title], [definition]) → Patch`

Adds new Patch to the Network. Patch is a container for a set of nodes and connections between them. Every Patch added this way is _opened_ by default, which means that it is rendered right away, and reacts immediately to every following change. You may set a patch title here and, also optionally, define handlers for the [events happening inside](./events.md#Patch), this way:

#### `Rpd.addClosedPatch(title, [definition]) → Patch`

Adds new Patch to the Network almost the same way as `addPatch` above, but this patch is _closed_ when you add it, so you need to explicitly call its [`open()`](#patch-open) method when you want this patch to render.

Patch may exist in two conditions: _opened_ — when you, as a user, observe all the events happening inside, new nodes appear, links connect, data flows, and everything is visually in motion, and _closed_ — when you, as a user, see nothing, but all the rendering yet happens somewhere in background. When you switch some Patch from closed state to an opened one, it shows everything happened before in the target you assigned, and vice versa.

It becomes useful when you have a Network of Patches and you want to show some while hiding others. In another words, you have some dependent Patch you don't want to be displayed until requested. This type of patches I'd recommend to call _Procedure Patch_, which is, unlike the _Root Patch_, treated as secondary.

All the Patches are opened by default. So to add initially closed patch, use this exact method (`Rpd.addClosedPatch`) or use [`patch.close()`](#patch-close), when you want to close it later.

<!-- IN PROGRESS -->

#### `Rpd.nodetype(type, definition)`

<!-- PROPLIST: Node Definition -->

* `title`: `string`
* `inlets`: `object { <alias>*: inlet_definition }`
* `outlets`: `object { <alias>*: outlet_definition }`
* `prepare`: `function`: `(inlets, outlets) → nothing`
* `process`: `function`: `(inlets_values, prev_inlets_values) → outlets_values`
* `tune`: `function`: `(updates_stream) → updates_stream`
* `handle`: `object { <event>*: handler }`

<!-- /PROPLIST -->

Register a new type of the nodes, so you, or the user, may easily create instances of this type with the help of `patch.addNode` or using some other interface.

So you may define once that all the nodes of your type have two inlets and one outlet, which channel types they have, and how the node processes data, and then create 300 instances of this node type, when you really want.

NB: Please note that user may in any case extend the instance with own definition using `patch.addNode(type, definition)` method, add or remove inlets/outles and modify properties. You should not care too much about that or even you need not care at all, but in some rare cases that could be important.

The new `type` name should be in the form `toolkit/typename`. For example, there could be nodes with the types `util/bang`, `util/color`, `blender/render`, `animatron/player`, `processing/color`, `processing/sketch` etc. Prefer one word for the type name when possible, or join several words with dash symbol `-`, when it's really not.

Then goes the `definition`, which is described in details in [Node Definition](#node-definition) section. Just note that when you need, you may pass a function to this method, it is very useful when you need to share some objects between definitions, so both examples are valid:

```javascript
Rpd.nodetype('docs/foo', {
    title: ...,
    inlets: ...,
    outlets: ...,
    process: ...
});

Rpd.nodetype('docs/foo', function() {
    var someSharedVariable;
    var nodeInstance = this;
    return {
        title: ...,
        inlets: ...,
        outlets: ...,
        process: ...
    };
});
```

Note: When you need information on creating your own toolkits, head safely to the [Toolkits](./toolkits.html) section.

#### `Rpd.nodedescription(type, description)`

Any node type can have a literary textual description of what this node does in details. Normally renderer shows it in the node list, next to corresponding node type, when available, and also when user hovers over the title of the node.

```javascript
Rpd.nodedescription('docs/foo', 'Used as the example for documentation');
```

#### `Rpd.channeltype(type, definition)`

<!-- PROPLIST: Channel Definition -->

* `label`: `string`
* `default`: `any`
* `hidden`: `boolean`
* `cold`: `boolean`
* `readonly`: `boolean`
* `allow`: `array[string]`
* `accept`: `function`: `(value) → boolean`
* `adapt`: `function`: `(value) → value`
* `show`: `function`: `(value) → string`
* `tune`: `function`: `(values_stream) → values_stream`
* `handle`: `object { <event>*: handler }`

<!-- /PROPLIST -->

Register a new type of a Channel, so you, or the user, may easily create instances of this type with the help of `node.addInlet` or `node.addOutlet`, or using some other interface.

This helps you to define a Channel properties once and use them everywhere. Some of the channels in Util Toolkit are: `util/number`, which handles number values, `util/boolean` which handles boolean values, `util/color` which handles color values or even `util/bang` which handles Bang signal used to trigger some actions.

NB: Please note that user may in any case extend the Channel with own definition using `node.addInlet(type, alias, definition)` or `node.addOutlet(type, alias, definition)` forms of the methods, change the handlers or options. You should not care too much about that or even you need not care at all, but in some rare cases that could be important.

The new `type` name should be in the form `toolkit/typename`. For example, there could be Channels with the types `util/bang`, `util/color`, `tibre/wave`, `processing/color`, `processing/shape`, `animatron/tween` etc. Prefer one word for the type name when possible, or join several words with dash symbol `-`, when it's really not.

Then goes the definition, which is described in details in [Inlet Definition](#inlet-definition) section. Channel Definition is used both for Inlets and Outlets, but Outlets [lack of several options](#outlet-definition) which are used only for the Inlets. In the case when Outlet was created with a Channel or overriden Channel Definition, that contains some option belonging only to Inlets, this Outlet just doesn't takes these particular options in consideration.

Just note that when you need, you may pass a function to this method, it is very useful when you need to share some objects between definitions, so both examples are valid:

```javascript
Rpd.channeltype('docs/foo', {
    allow: ...,
    accept: ...,
    adapt: ...,
    show: ...
});

Rpd.channeltype('docs/foo', function() {
    var someSharedVariable;
    var channelInstance = this;
    return {
        allow: ...,
        accept: ...,
        adapt: ...,
        show: ...
    };
});
```

Note: When you need more details, head safely to the [Toolkits](./toolkits.html) section, which is the tutorial for writing your very own toolkit.

#### `Rpd.noderenderer(type, rendererAlias, definition)`

<!-- PROPLIST: Node Renderer -->

* `size`: `object { width, height }`
* `first`: `function(bodyElm) [→ object { <inlet>*: { default, valueOut } }]`
* `always`: `function(bodyElm, inlets, outlets)`

<!-- /PROPLIST -->

Define new Node Renderer for particular Node Type. When you want to have and reuse some Node which is more complex to render than just empty body with inlets or outlets, this method is what you need.

It allows you to put in the Node body whatever you want and improve user experience in every possible way. Using mostly only Node Types and corresponding Renderers for them, you may create the analogues of Pure Data, VVVV, Blender Material composer, or whichever node system comes to your mind.

The only limits you have are the limits of HTML or SVG, but there are both nowadays also almost limitless.

The Toolkits for the [Examples](../examples.html) section and the ones located at `src/toolkit` are all powered by `Rpd.nodetype`, `Rpd.channeltype`, `Rpd.noderenderer` and `Rpd.channelrenderer`, but `Rpd.noderenderer` is what makes them so powerful, since you may include, for example, control of any complexity, HTML5 Canvas, or Processing Sketch, or even something WebGL-driven into the node body. The important thing is how to deal with Toolkit architecture.

But let's turn from advertisement back to API.

`type` is the type of the node you want to define renderer for.

`rendererAlias` is a name of a Renderer which should already be registered in the system under this alias. Out of the box, there are `'html'` and `'svg'` renderers provided. Though you should ensure [to include Renderer](./setup.html) into your version of RPD before using one of them. Both of them support HTML and SVG DOM Elements, but for latter one the Node body is itself an SVG Element, so you if you want to add HTML Elements there, you need put them into `<foreignelement />` tag before, in the `definition`.

May receive both object or function, returning the object, as `definition`. Structure of this object is described below. When it's a function, it receives Node instance as `this`. <!-- check -->

Any property in definition is optional.

```javascript
Rpd.noderenderer('docs/foo', 'html', {
    size: ...,
    first: ...,
    always: ...
});

Rpd.noderenderer('docs/foo', 'html', function() {
    var someSharedVariable;
    var nodeInstance = this;
    return {
        size: ...,
        first: ...,
        always: ...
    };
});
```

<!-- TODO: also may appear in path.addNode(..., ..., <node-definition>, <render-definition>) -->

<!-- TODO: it is also possible to override `render` in `patch.addNode` and `node.addInlet/node.addOutlet`, check -->

Note: When you need more details, head safely to the [Toolkits](./toolkits.html) section, which is the tutorial for writing your very own toolkit.

##### `size` : `object`

Restrict the size of the Node body to some particular size. It is the object in the form `{ width: 100, height: 200 }`. You may omit width or height property, if you want it to be automatically calculated. When you omit the `size` option completely, the Node body is assigned automatically by Style, in some cases it's enough, in other cases you may need more space for body controls, for example.

Pay attention that the same Node body size may look better for one Style and worse for another. If you want to support several Styles (it's for sure not obligatory), it's better to check if your Node looks well for all of them.

<!-- ##### `prepare` : `function(patchNode, currentPatchNode)` -->

##### `first` : `function(bodyElm) [→ object]`

This handler is called once, just before the Node is ready to process incoming data and when all the Inlets and Outlets defined in [Type Definition](#node-definition) are already attached the Node.

Use this handler to prepare the Node body, i.e. append required DOM (or whichever, it depends on the Renderer) elements there. When you use the form of the function to define `Rpd.noderenderer`, you may safely save these elements in the closure to use them in `always` handler.

```javascript
Rpd.noderenderer('docs/recipe', 'html', {
    first: function(bodyElm) {
        var recipeText = '...';
        var spanElement = document.createElement('span');
        spanElement.innerText = recipeText;
        bodyElm.appendChild(spanElement);
    }
});

Rpd.noderenderer('docs/color', 'html', function() {
    var colorElement;
    return {
        first: function(bodyElm) {
            var colorElement = document.createElement('span');
            colorElement.style.backgroundColor = 'transparent';
            bodyElm.appendChild(colorElement);
        },
        always: function(inlets) {
            colorElement.style.backgroundColor = '(' +  inlets.r + ',' + inlets.g + ',' inlets.b + ')';
        }
    };
});
```

This function may optionally return the object which allows to attach default values or streams of the values to the existing inlets. <!-- #354 --> It is very useful when you want to have some complex control (or several ones) in the Node body, so you add control there and pass its changes [Stream][kefir] (for example, `'change'` event) to an existing hidden inlet.

Don't be afraid, usually you'll need Kefir Streams only to transfer events to the Inlet, so it will be just `return Kefir.fromEvents(myControl, 'change');` here... Or may be you'll find useful to also `.map` values to something. And `.throttle` them in some cases... Streams could appear very useful!

So, first, for every inlet returned, you may specify `'default'` property, which could be a function returning a default value (so you will be able to initialize your control with this value in this function) or just some value.

And, second, you may specify `'valueOut'` [Stream][kefir], which should emit new value when you want to update inlet value. Usually it is ok to pass `'change'` events Stream from your control there.

What `bodyElm` is, depends on the Renderer you use for rendering. For example, for `'html'` Renderer it is HTML Element and for `'svg'` renderer it is SVG Element, correspondingly.

NB: It is highly recommended not to change `bodyElm` attributes or especially remove it from the flow. In most cases adding DOM children to it will satisfy all your needs. It is not the strict law, however — don't feel like someone prevents you — you're grown-ups, you know when you may break some ~~rules~~ HTML Elements.

```javascript
// sends random number to a hidden inlet immediately after a link inside the Node body was clicked

Rpd.nodetype('docs/random-on-click', {
    inlets: {
        'click': { type: 'core/any', hidden: true }
    },
    outlets: {
        'random': { type: 'util/number' }
    },
    process: function(inlets) {
        if (inlets.click) return { random: Math.random() };
    }
});

Rpd.noderenderer('docs/random-on-click', 'html', {
    first: function(bodyElm) {
        var clickElm = document.createElement('a');
        clickElm.href = '#';
        clickElm.innerText = 'Click Me!';
        bodyElm.appendChild(clickElm);
        return {
            'click': Kefir.fromEvents(clickElm, 'click')
        }
    }
});
```

NB: The `valueOut` and `default` functionality is discussable, please follow [Issue #354](https://github.com/shamansir/rpd/issues/354) if you want to keep track on changes, if they come, or feel free to add comments if you have any suggestions on how to improve it.

Receives Node instance as `this`.

##### `always` : `function(bodyElm, inlets, outlets)`

This function is called on every inlet update, next to the Node `process` handler (described in [Node Definition](#node-definition)), when the latter was defined.

So you may apply/render all the new updates immediately after the moment they happened. `inlets` object contains new Inlets values, `outlets` object contains
current Outlets values, including those returned from the `process` handler.

```javascript
// see `docs/random-on-click` Node type definition in previous example,
// this is a slightly modified version which also displays the generated
// random number inside node body
Rpd.noderenderer('docs/random-on-click', 'html', function() {
    var numberElm;
    return {
        first: function(bodyElm) {

            var clickElm = document.createElement('a');
            clickElm.href = '#';
            clickElm.innerText = 'Click Me!';
            bodyElm.appendChild(clickElm);

            numberElm = document.createElement('span');
            numberElm.innerText = '<?>';
            bodyElm.appendChild(numberElm);

            return {
                'click': Kefir.fromEvents(clickElm, 'click')
            };
        },
        always: function(bodyElm, inlets, outlets) {
            numberElm.innerText = outlets.random;
        }
    };
});
```

#### `Rpd.channelrenderer(type, rendererAlias, definition)`

<!-- PROPLIST: Channel Renderer -->

* `show`: `function(target, value, repr)`
* `edit`: `function(target, inlet, valueIn) [→ change_stream]`

<!-- /PROPLIST -->

Register a Renderer for a Channel Type.

This allows you to render values which appear near to Inlets and Outlets of particular Channel Type not only as String, but in any kind of visual presentation. For example, you may display a color value as a color box filled with this color, instead of boring variants like `#883456` or `[Some Color]`, near to any Inlet or Outlet having your own `my/color` Channel Type:

```javascript
Rpd.channetype('docs/color', {});

Rpd.channelrenderer('docs/color', 'html', {
    show: function(target, value) {
        var colorElm = document.createElement('span');
        colorElm.style.height = '30px';
        colorElm.style.width = '30px';
        colorElm.style.backgroundColor = 'rgb(' + value.r + ',' + value.g + value.b + ');';
        target.appendChild(colorElm);
    }
});
```

This method may receive either object following the structure described below, or function which returns object of same structure. It is helpful when you need to share some data to reuse in all methods using a closure.

`type` is an alias of a Channel Type, you render the values for.

`rendererAlias` is the alias of a registered Renderer, like `'html'` or `'svg'`, both of which come out of the box.

`definition` is the object that describes how the Channel Renderer should behave in different situations, its possible properties are covered below.

You also may pass a function which returns such object instead, it will help you to store shared variables in the closure. When you do so, this function receives Channel instance as `this`.

```javascript
Rpd.channelrenderer('docs/foo', 'html', {
    show: ...,
    edit: ...
});

Rpd.channelrenderer('docs/foo', 'html', function() {
    var someSharedVariable;
    var nodeInstance = this;
    return {
        show: ...,
        edit: ...
    };
});
```

<!-- TODO: also may appear in node.addInlet(..., ..., <channel-definition>, <channel-render-definition>), node.addOutlet(..., ..., <channel-definition>, <channel-render-definition>) -->

Note: When you need more details, head safely to the [Toolkits](./toolkits.html) section, which is the tutorial for writing your very own toolkit.

<!-- ##### `prepare` : `function()` -->

##### `show` : `function(target, value, repr)`

This function may convert new received value to some renderable element. For example, you may define `my/vector` Channel type which renders as the direction this vector points to, in SVG:

```javascript
Rpd.channeltype('docs/vector', {});

var SVG_XMLNS = 'http://www.w3.org/2000/svg';
var radius = 7;
Rpd.channelrenderer('docs/vector', 'svg', {
    show: function(target, value) {
        var circle = document.createElementNS(SVG_XMLNS, 'circle');
        circle.setAttributeNS(null, 'cx', 0);
        circle.setAttributeNS(null, 'cy', 0);
        circle.setAttributeNS(null, 'r', radius);
        circle.setAttributeNS(null, 'fill', 'white');
        circle.setAttributeNS(null, 'stroke', 'black')
        circle.setAttributeNS(null, 'strokeWidth', 1);
        var line = document.createElementNS(SVG_XMLNS, 'line');
        line.setAttributeNS(null, 'x1', 0);
        line.setAttributeNS(null, 'y1', 0);
        line.setAttributeNS(null, 'x2', Math.cos(value.angle) * radius);
        line.setAttributeNS(null, 'y2', Math.sin(value.angle) * radius * -1);
        line.setAttributeNS(null, 'stroke', 'black');
        line.setAttributeNS(null, 'strokeWidth', 1);
        target.appendChild(circle);
        target.appendChild(line);
    }
});
```

<!-- TODO: do this -->

`target` is the element (HTML Element for `'html'` Renderer, SVG Element for `'svg'` Renderer and so on) where you should put your own element, the one representing the value, into.

`value` is the most fresh value this Channel received.

`repr` is the string representation returned from [Channel Definition](#channel-definition) `show` method, if it was defined or just `.toString()` call on the value, when it wasn't.

NB: Node Types names and Channel Types named may intersect since Node can also represent a single thing which can also be passed through a Channel.

##### `edit` : `function(target, inlet, valueIn) [→ change_stream]`

If you want to let user edit the value not (or _not only_) in the Node body, but also when she clicks the value near to the Inlet, you may use this method to provide her that. <!-- Though it also depends on [the rendering process configuration](./network.html#rendering-configuration), there is an option to disable value editors named ... TODO ? -->. This method is called only for Inlets, not for Outlets, since only input values are allowed to be changed without connections.

`target` is the element (HTML Element for `'html'` Renderer, SVG Element for `'svg'` Renderer and so on) where you should put your own element, the one representing the value, into.

`inlet` is the Inlet where editor was attached.

`valueIn` is the [stream][kefir] of incoming values, so you may update the editor with new values. It is also useful to filter values when editor has user focus, so she won't distract from the process of changing the value.

The function should return a [stream][kefir] of outgoing values, so every time user selects or confirms some value, it should be passed to this stream.

```javascript
Rpd.channelrenderer('docs/color', 'html', {
    show: function(target, value) {
        // see `show` code above
    },
    edit: function(target, inlet, valueIn) {
        var input = document.createElement('input');
        input.type = 'text';
        valueIn.onValue(function(value) {
            input.value = colorToText(value);
        });
        target.appendChild(input);
        return Kefir.fromEvents(input, 'change').map(textToColor);
    }
});
```

#### `Rpd.renderer(alias, definition)`

Renderer Definition is completely moved to [Style Section](../style.html#writing-your-own-renderer), since it doesn't relate to Building Patches.

#### `Rpd.style(alias, rendererAlias, definition)`

Style Definition is completely moved to [Style Section](./style.html#writing-your-own-style), since it doesn't relate to Building Patches.
