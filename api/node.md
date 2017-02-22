<!-- MARK: Node -->

### `Node`

Node represents the thing we call procedure in programming: it receives data through its inputs (inlets), does something using that data and returns either the same data, modified or not, or completely different data in response using outputs (outlets). But from this point, it goes beyond, since it may visualize the process inside its body or add some complex visual controls for additional inputs. On the other hand, it may stay in a boring state and have no inputs, no outputs and even no content at all. Everything depends only on yours decision.

#### Node Definition

<!-- PROPLIST: Node Definition -->

* `title`: `string`
* `inlets`: `object { <alias>*: inlet_definition }`
* `outlets`: `object { <alias>*: outlet_definition }`
* `prepare`: `function`: `(inlets, outlets) → nothing`
* `process`: `function`: `(inlets_values, prev_inlets_values) → outlets_values`
* `tune`: `function`: `(updates_stream) → updates_stream`
* `handle`: `object { <event>*: handler }`

<!-- /PROPLIST -->

Definition of the Node is the configuration object used to define
new Node Type with `Rpd.nodetype` or an object with the same structure, passed to `patch.addNode` method, intended to override or to append the Type Definition.

```javascript
Rpd.nodetype(..., <node-definition>);
Rpd.nodetype(..., function() {
    return <node-definition>;
});

patch.addNode(..., ..., <node-definition>);
```

This object may contain no properties at all, or, in cases when Node Type or a single Node needs its originality, some of the following properties:

##### `title`: `string`

Node title, usually displayed on the top of the node, defaults to node type if not specified or empty.

##### `inlets`: `object`

An object, containing a list of inlets this node has, _key_ is inlet label and _value_ is definition. For example, two number-typed inlets with labels `'a'` and `'b'`:

```javascript
Rpd.nodetype('docs/example', {
    'inlets': { 'a': { type: 'util/number' },
                'b': { type: 'util/number' } }
});
```

There are much more properties of the inlets available, see [Inlet Definition](#inlet-definition) for a full list of them.

##### `outlets`: `object`

An object, containing a list of outlets this node has, _key_ is outlet label and _value_ is definition. For example, two number-typed outlets with labels `'a'` and `'b'`:

```javascript
Rpd.nodetype('docs/example', {
    'outlets': { 'a': { type: 'util/number' },
                 'b': { type: 'util/number' } }
});
```

There are much more properties of the outlets available, see [Outlet Definition](#outlet-definition) for a full list of them.

##### `prepare`: `function`: `(inlets, outlets) → nothing`

When new node instance is created, it is filled with inlets and outlets from corresponding Type Definition. Then node is triggered as ready to perform processes. When you want to configure its inlets or outlets before any processing, you may use this `prepare` function for that.

NB: `prepare` function is called only when node has `process` handler.

Receives Node instance as `this`.

<!-- IN PROGRESS -->

##### `process`: `function`: `(inlets_values, prev_inlets_values) → outlets_values`

The `process` handler is the main function, the really important one for the Node Type definition. This function is triggered on every update appeared on any of the inlets and converts the data received through inlets to the data which is required to be sent to outlets. For example, `util` node, designed to multiply two numbers and send the result out, has a definition like this:

```javascript
Rpd.nodetype('util/*', {
    inlets: { 'a': { type: 'util/number' },
              'b': { type: 'util/number' } },
    outlets: { 'result': { type: 'util/number' } },
    process: function(inlets) {
        return { 'result': (inlets.a || 0) * (inlets.b || 0) };
    }
});
```

Though it is not obligatory to process all the inlets or to send data to every outlet—in some cases this function not cares about input or output at all. When Node Type defines no `process` function or it wasn't defined in `patch.addNode` method, node makes actually nothing.

Another important thing to notice is that you may return [Kefir Stream][kefir] as an outlet value, so the values from this stream will be sent to the outlet just when they are triggered. In this case, however, you should make the stream finite or stop this stream manually, or else streams for one inlet will merge with every next call to `process` function. For the real life example, see `util/metro` node definition in `src/toolkit/util/toolkit.js` file.

Sometimes you may want to trigger `process` function manually with some new data, but you don't want to send it through user inlet. Adding hidden inlet for your internal data is a common trick often used even in Toolkits provided with RPD distribution:

```javascript
Rpd.channeltype('docs/bang', {
    adapt: function(value) {
        return (typeof value !== 'undefined') ? {} : null;
    }
});

Rpd.nodetype('docs/bang', {
    inlets: { 'trigger': { type: 'docs/bang', hidden: true } },
    outlets: { 'bang': { type: 'docs/bang' } },
    process: function(inlets) {
        return inlets.trigger ? { 'bang': {} } : {};
    }
});
```

Usually when Node has some controller or input inside of its body, values from this controller are sent to a corresponding, existing, Inlet of this Node, so they come to `process` handler joined with other incoming data. But sometimes you may want to make input-connected Inlet hidden and leave it's pair visible, so user won't be surprised with new values coming to Inlets not from connected Outlets, but from nowhere.

In this case it could be important to know which Inlet received the value first and which received its own value later. For example, when Node has some input in a body, its updated value is usually sent to hidden Inlet, but also some visible Inlet in the same node is provided to the user, so she'll able to use it, when she wants to override this value from another Node.

For this reason we should know, which value came first, from user or from the controller inside, so to rewrite controller value only in the first case. For example `util/timestamped` Channel Type wraps any incoming value with timestamp. Let's implement a similar functionality which will will help us to solve the problem in this case:

```javascript
Rpd.channeltype('docs/number-timestamped', {
    allow: [ 'util/number' ],
    adapt: function(value) {
        return {
            time: new Date(),
            value: value
        }
    },
    show: function(v) { return v.value }
});

function getMostRecentValue(fromOtherNode, fromNodeBody) {
    if (!fromNodeBody) { return fromOtherNode.value; }
    else if (!fromOtherNode) { return fromNodeBody.value; }
    else {
        return (fromNodeBody.time > fromOtherNode.time)
            ? fromNodeBody.value : fromOtherNode.value;
    }
}

Rpd.nodetype('docs/inlet-or-body', {
    inlets: { 'from-other-node': { type: 'docs/number-timestamped' },
              'from-node-body': { type: 'docs/number-timestamped',
                                  hidden: true } },
    outlets: { 'recent': { type: 'util/number' } },
    process: function(inlets) {
        return {
            recent: getMostRecentValue(inlets['from-other-node'],
                                       inlets['from-node-body'])
        };
    }
});

Rpd.nodetyperenderer('docs/inlet-or-body', 'html', function() {
    var input;
    return {
        first: function(bodyElm) {
            input = document.createElement('input');
            input.type = 'number';
            bodyElm.appendChild(input);
            return {
                'from-node-body': Kefir.fromEvents(input, 'change')
                                       .map(function(event) {
                                           return event.target.value;
                                       })
            }
        },
        always: function(bodyElm, inlets) {
            if (inlets['from-other-node'] &&
                (!inlets['from-node-body'] ||
                 ( inlets['from-other-node'].time >  
                   inlets['from-node-body'].time ))) {
              input.value = inlets['from-other-node'].value;
            }  
        }
    };
});
```

As another option, you may add timestamps to Inlets' values using their own `tune` function, or using the `tune` function of the Node, which is described just below and by chance there's an example which shows how to do it.

Receives Node instance as `this`.

##### `tune`: `function`: `(updates_stream) → updates_stream`

This function allows you to tune all the updates from the inlets, so, for example, you may skip every second update from specific inlet, or every second update in general. Or you may multiply every new numeric value by 10. It gets the [Kefir Stream][kefir] which represents all the updates from the node inlets merged. When you return the same stream you received from this function, it changes nothing in the process.

Each update in `updates_stream` stream is the object in a form `{ inlet, value }`, where `inlet` is `Inlet` instance which received the update and `value` is the new value received. You should return the same structure from this function, but you are free to substitute values or even inlets.

Some examples:

```javascript
Rpd.nodetype('docs/delay', {
    inlets: { 'this': { type: 'core/any' },
              'that': { type: 'core/any' } },
    outlets: { 'delayed': { type: 'core/any' } },
    tune: function(updates) {
        return updates.delay(3000); // delays all updates for three seconds
    },
    process: function(inlets) {
        return { delayed: inlets['this'] || inlets['that'] };
    }
});
```

```javascript
Rpd.nodetype('docs/timestamp-example', {
    inlets: { 'in': { type: 'util/number' } },
    outlets: { 'out': { type: 'util/number' } },
    tune: function(updates) {
        return updates.map(function(update) {
            var updateCopy = Object.assign({}, update);
            updateCopy.value = { value: update.value,
                                 time: new Date() }
            return updateCopy;
        })
    },
    process: function(inlets) {
        console.log(inlets.in.time);
        return { out: inlets.in.value };
    }
});
```

Receives Node instance as `this`.

##### `handle`: `object`

This object allows you to subscribe to any event this Node produces. _Key_ in this object is the event name, and _value_ is the handler. See [Events](./events.html) section for the complete list of the events.

An example:

```javascript
Rpd.nodetype('docs/handle-events', {
    inlets: { 'in': { type: 'util/number' } },
    outlets: { 'out': { type: 'util/number' } },
    handle: {
        'node/move': function(event) {
            console.log(event);
        },
        'node/add-inlet': function(event) {
            console.log(event);
        },
        'inlet/update': function(event) {
            console.log(event);
        }
    }
});
```

----

#### `node.addInlet(type, alias, [definition]) → Inlet`

Add the input channel to this node, so it will be able to receive data and pass this data inside the node. You need to specify the type of this channel, so the system will know which way to process your data before passing it inside, or even decline connections from other types of channels. `core/any` is the system type which accepts connections from outlets of any type, so probably for start you'd want to use it. Later, though, it could be better to change it to something more specific, i.e. decide that it accepts only numbers or colors values. This will allow you to control the way data in this channel is displayed or even add [custom _value editor_](#rpd-channelrenderer) to this channel.

The second argument, `alias`, is the label of this channel displayed to user and also may be used to access this inlet from inside the node, so it's recommended to make it one-word and start from lowercase letter, like the key names you normally use for JavaScript objects. There is another form of this method, `addInlet(type, alias, label, [definition])`, using which you may specify user-friendly name to display in UI with `label` attribute, and still use short programmer-friendly `alias` to access this inlet from the code.

Last argument, `definition`, is optional, and allows you to override the options inherited from type description for this particular instance. This object is described in details in the [Inlet](#Inlet) <!-- or Rpd.channeltype? --> section below.

```javascript
Rpd.channeltype('docs/topping', {});
Rpd.channeltype('docs/cone', {});
Rpd.channeltype('docs/taste', {});
Rpd.channeltype('docs/size', {
    show: function(value) { return '#' + value }
});
Rpd.channeltype('docs/yoghurt', {});

var frozenYoghurtFactoryNode = patch.addNode('core/basic', 'Frozen Yoghurt', {
    process: function(inlets) { return { yoghurt: inlets } };
});
var toppingInlet = frozenYoghurtFactoryNode.addInlet('docs/topping', 'topping');
var coneInlet = frozenYoghurtFactoryNode.addInlet('docs/cone', 'cone');
var tasteInlet = frozenYoghurtFactoryNode.addInlet('docs/taste', 'taste');
var sizeInlet = frozenYoghurtFactoryNode.addInlet('docs/size', 'size', {
    allow: [ 'util/number' ],
    accept: function(size) { return (size > 0) && (size <= 4); },
    adapt: function(value) { return Math.floor(value); }
});
var yoghurtOutlet = frozenYoghurtFactoryNode.addOutlet('docs/yoghurt', 'yoghurt');

var knob = patch.addNode('util/knob');
knob.inlets['min'].receive(1);
knob.inlets['max'].receive(4);
knob.outlets['number'].connect(sizeInlet);
```

By default, inlets accept connection only from one outlet, so when user connects some other outlet to this inlet, the previous connection, if it existed, is immediately and automatically removed. Though, you can pass an option to the renderer named `inletsAcceptMultipleLinks` and set it to `true`, so multiple connections will be available to user and inlets will receive values from all the outlets connected in order they were fired. <!-- FIXME: check if it is really so and consider #336 -->

You can discover the complete list of the properties which could be used in this definition if you follow to [Inlet Definition](#inlet-definition) section.

#### `node.addOutlet(type, alias, [definition]) → Outlet`

Add the output channel to this node, so it will be able to send data to the inlets of other nodes, when connected to them. Same way as for `addInlet` method described above and following the same reasons, you need to specify the type of the channel, which can be `core/any` while you do experiments and is recommended to be changed to something more specific later, unless this channel was really intended to accept anything.

Also, you need to specify `alias`, to be able to access this outlet from the code using this `alias`. It is recommended to be short, preferably one-word and to start from lowercase letter. If you want to show user something more eye-candy, you may use another form of this method, `addOutlet(type, alias, label, [definition])`.

Last argument, `definition`, is optional, and allows you to override the options inherited from type description for this particular instance. This object is described in details in the [Outlet](#Outlet) <!-- or Rpd.channeltype? --> section below.

```javascript
Rpd.channeltype('docs/meat-type', {});
Rpd.channeltype('docs/rice', {
    adapt: function(choice) {
        if (choice == 1) return 'plain';
        if (choice == 2) return 'mexican';
    }
});
Rpd.channeltype('docs/guacamole', {});
Rpd.channeltype('docs/cheese', {});
Rpd.channeltype('docs/to-go', {});
Rpd.channeltype('docs/spicy', {
    show: function(value) { return '#' + value }
});
Rpd.channeltype('docs/burrito', {});

var burritoFactoryNode = patch.addNode('core/basic', 'Burrito', {
    process: function(inlets) { return { burrito: inlets } }
});
burritoFactoryNode.addInlet('docs/meat-type', 'meat');
burritoFactoryNode.addInlet('docs/rice', 'rice');
burritoFactoryNode.addInlet('docs/guacamole', 'guacamole');
burritoFactoryNode.addInlet('docs/cheese', 'cheese');
burritoFactoryNode.addInlet('docs/to-go', 'to-go');
var spicyInlet = burritoFactoryNode.addInlet('docs/spicy', 'spicy', {
    allow: [ 'util/number' ],
    accept: function(spicy) { return (spicy > 0) && (spicy <= 4); },
    adapt: function(value) { return Math.floor(value); }
});
var burritoOutlet = burritoFactoryNode.addOutlet('docs/burrito', 'burrito', {
    show: function(burrito) {
        var wrapped = Array.isArray(burrito);
        if (wrapped) {
            return '[ Burrito ' + (burrito[0].guacamole ? '+$1.80' : '') + ' ]';
        } else {
            return 'Burrito' + (burrito.guacamole ? '+$1.80' : '')
        }
    },
    tune: function(stream) {
        return stream.map(function(burrito) {
            if (burrito['to-go']) {
                return wrap(burrito);
            } else {
                return burrito;
            }
        });
    }     
});

function wrap(burrito) { return [ burrito ]; }

var knob = patch.addNode('util/knob');
knob.inlets['min'].receive(1);
knob.inlets['max'].receive(4);
knob.outlets['number'].connect(spicyInlet);
```

You can discover the complete list of the properties which could be used in this definition if you follow to [Outlet Definition](#outlet-definition) section.

#### `node.removeInlet(inlet)`

Remove specified inlet from the node. Node stops receiving any updates sent to this inlet and so removes this inlet from its data flow.

```javascript
// works with the example from `node.addInlet(...)` method
frozenYoghurtFactoryNode.removeInlet(toppingInlet);
```

#### `node.removeOutlet(outlet)`

Remove specified outlet from the node. Node stops sending any values passed to this outlet and so removes this outlet from its data flow.

```javascript
// works with the example for `node.addOutlet(...)` method
burritoFactoryNode.removeOutlet(burritoOutlet);
```

#### `node.move(x, y)`

Move this Node to the specified position relatively to the top left corner of the canvas of the Patch it belongs to. `x` and `y` are just numbers, while they could be treated differently by every renderer.

#### `node.turnOn()`

Turn this Node on, so it processes all the incoming updates and sends values further, if it has inputs and outputs. By default Nodes are always turned on.

#### `node.turnOff()`

Turn this Node off, so it stops all the processing. This method is useful when your Node has a lot of connections and you don't want to disconnect or disable them one by one, but to quickly turn them off at once and to have the ability to turn them back on, same way, all at once. <!-- TODO: text -->

<!-- Or, could happen, you may decide to provide user with this nice ability to turn everything off and on, for example when user clicks something located in the body of a node. -->
<!-- TODO: Make an issue for this, to be a bulb in the node header -->