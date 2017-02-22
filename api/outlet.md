<!-- MARK: Outlet -->

### `Outlet`

Outlet is the output channel of the node.

#### Outlet Definition

<!-- PROPLIST: Channel Definition -->

* `label`: `string`
* `show`: `function`: `(value) → string`
* `tune`: `function`: `(values_stream) → values_stream`
* `handle`: `object { <event>*: handler }`

<!-- /PROPLIST -->

Definition of the Inlet is the configuration object used to define
new Channel Type with `Rpd.channeltype` or an object with the same structure, passed to `node.addOutlet` method or with `outlets` property to Node type or instance definition, intended to override or to append the Type Definition.

All the functions in the definition get Inlet instance as `this`.

```javascript
Rpd.channeltype(..., <channel-definition>);
Rpd.channeltype(..., function() {
    return <channel-definition>;
});

Rpd.nodetype(..., ..., {
    inlets: {
        alias-1: <channel-definition>,
        alias-2: <channel-definition>
    },
    outlets: {
        alias-1: <channel-definition>
    },
    ...
});

var inlet = Rpd.addInlet(..., ..., <channel-definition>)
var outlet = Rpd.addOutlet(..., ..., <channel-definition>);
```

This object may contain no properties at all, or, in cases when Outlet Type or a single Outlet needs its originality, some of these properties:

##### `label`: `string`

Outlet label, usually displayed near to the outlet. Try to make it both as short and descriptive as possible, since most of they styles are expecting it to be short. Also, when style options were set not to display values, this can happen that your user won't see a label at all or see it only when she hovers over the Outlet. <!-- check `valuesOnHover` -->

##### `show`: `function`: `(value) → string`

This function is used to show user-friendly string when displaying the value of the Outlet. For example, when your Outlet receives an array of values, by default it will be shown as `[Array]` or if it's an object, as `[Object]` (since by default it just uses `.toString` method of JavaScript), not too user-friendly, isn't it?

Receives Outlet instance as `this`.

##### `tune`: `function`: `(values_stream) → values_stream`

Tuning function is very powerful and allows you to control the outgoing stream of values, modifying some, skipping some, delaying some, or applying a combination of these actions and so completely changing what happens. [Kefir streams][kefir] are what allows to do all these magical things.

But please be aware that when stream of values is heavily modified, user may feel uncomfortable, while it could not be obvious without seeing what happens inside. So try to modify it so user won't see the effect or explain what happens with the help of the UI. For example, when you filter output stream, explain why it is filtered in the body of the Node or better provide user control over filtering with new Inlet.

```javascript
Rpd.channeltype('docs/mouse-pos', {
    tune: function(stream) {
        // output mouse events with a minimum distance of 10 milliseconds
        stream.throttle(10);  
    }
});
```

Receives Outlet instance as `this`.

##### `handle`: `object`

This object allows you to subscribe to any event this Node produces. _Key_ in this object is the event name, and _value_ is the handler. See [Events](#) section for the complete list of the events.

An example:

```javascript
Rpd.channeltype('docs/just-another-outlet', {
    handle: {
        'outlet/connect': function(event) {
            console.log('connected to ', event.inlet);
        }
    }
});
```

Every handler receives Inlet instance as `this`.

----

#### `outlet.connect(inlet) → Link`

Establish a connection between this outlet and given Inlet. It is exactly the same what user does when connects some outlet to some Inlet using interface.

When connection was established, data flows through this wire perfectly, however the receiving end can decline any data on its will, for example when Outlet channel type is not matching the Inlet channel type or is not in the list if Inlet's channel types allowed to connect.

It depends on the options, but by default it is allowed to connect one Outlet to multiple Inlets, but Inlet may have only one incoming connection. So when some Inlet is already connected to an Outlet and you try to connect other Outlet to it, the previous connection should be removed in advance. <!-- TODO: control is performed only in renderer, that's not so good--> For user side of view, it is automatically performed by Renderer, when `config.inletAcceptsMultipleLinks` is set to `true`.

```javascript
var knob1 = patch.addNode('util/knob'),
    knob2 = patch.addNode('util/knob'),
    knob3 = patch.addNode('util/knob');
var color = patch.addNode('util/color');
knob1.outlets['number'].connect(color.inlets['r']);
var knob2ToGreenLink = knob2.outlets['out'].connect(color.inlets['g']);
knob3.outlets['number'].connect(color.inlets['b']);

var always42 = patch.addNode('docs/always-42', 'Always 42');
var outlet = always42.addOutlet('core/any', 'fourty-two', {
    tune: function(stream) {
        return stream.map(function() { return 42; });
    }
});
knob2ToGreenLink.disconnect();
outlet.connect(color.inlets['g']);
outlet.send(Math.PI); // will be converted to 42
```

<!-- test -->

#### `outlet.disconnect(link)`

Break the existing connection, so all the values from this outlet are no more delivered trough this link to the corresponding inlet.


#### `outlet.send(value)`

Force this outlet to send given value to all the connected inlets in other nodes, when there are any. These inlets can yet decline or modify the value basing on the channel type. (see `inlet.receive` description).

```javascript
myNode.addOutlet('docs/number', 'num').send(42);
myNode.addOutlet('docs/date', 'date').send(Date.parse('Mar 18, 2016'););
myNode.addOutlet('docs/radians', 'angle').send(Math.PI / 2);
```

#### `outlet.stream(stream)`

Force this outlet to receive the stream of values, any stream constructed with [Kefir API][kefir]. These values may be distributed over time in any way you want, and last till infinity or till the stream will end.

Yet, same as with `outlet.send`, value may be declined or modified on the receiving ends, when they exist (without interrupting the stream).

```javascript
// empty object is treated like a bang trigger for `util/bang` channel instances
myNode.addOutlet('util/bang').stream(Kefir.interval(3000, {}));

// control amount of white in the color using Y position of a mouse,
// send it every time mouse position was changed
myNode.addOutlet('util/color').stream(
    Kefir.fromEvents(document.body, 'mousemove')
         .map(function(event) {
             return { x: event.clientX, y: event.clientY };
         })
         .map(function(position) {
             return position.y % 255;
         })
         .map(function(value) {
             return { r: value, g: value, b: value };
         })
);
```

<!-- #### `outlet.toDefault()` -->
