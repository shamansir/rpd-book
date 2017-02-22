<!-- MARK: Inlet -->

### `Inlet`

Inlet is the name for one of the input channels of the node so, when its connected to something, the data may flow through it _into_ the node processing function from all of them. Inlets are differentiated by their alias, that's why aliases of inlets should be unique inside every node, yet they can be same between two nodes. Inlet is the opposite to Outlet, which allows data to flow _out_ of the node and is described next in this section.

#### Inlet Definition

<!-- PROPLIST: Channel Definition -->

* `label`: `string`
* `default`: `any | stream`
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

Definition of the Inlet is the configuration object used to define
new Channel Type with `Rpd.channeltype` or an object with the same structure, passed to `node.addInlet` method or with `inlets` property to Node type or instance definition, intended to override or to append the Type Definition.

All the functions in the definition get Inlet instance as `this`. <!-- TODO: check -->

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

This object may contain no properties at all, or, in cases when Inlet Type or a single Inlet needs its originality, some of these properties:

<!-- NB: there are several checks performed when user connects Outlet to Inlet: allow, accept, adapt -->

##### `label`: `string`

Inlet label, usually displayed near to the inlet. Try to make it short, when possible, so it fits any style.

##### `default`: `any`

Default value for this inlet, which will be sent to it just when node is ready, no matter, has it connections or not. If it has some, values from connection will always be sent after the default value.

```javascript
Rpd.channeltype('docs/color', {
	default: 'black'
});
```

This value can be any type, but also a [Kefir Stream][kefir], so you may configure this inlet to receive several or infinite amount of values just from the start:

```javascript
Rpd.channeltype('docs/alarm', {
    // ring every 24 hours by default
    default: Kefir.interval(1000 * 60 * 60 * 24, './nokia-sound.wav')
});
```

<!-- TODO: test -->

The default value will be passed to `tune`, then `accept` and `adapt` functions before being passed to node's `process` handler. <!-- TODO: check -->

##### `hidden`: `boolean`

You may set an Inlet to be hidden from user, so it is not visible, but yet may receive any data and trigger `process` function. <!-- TODO: test -->.

One of the cases when it comes useful, is when Node has an additional control in its body, and you want to send the output of this control to `process` handler, so it decides if incoming data has higher priority than data from control(s) or merges all the data, both from inlets and control in one as the configuration to calculate the output.

For the example of such, see [Node Renderer](#rpd-noderenderer) description.

##### `cold`: `boolean`

When Inlet is _cold_, any incoming update to this Inlet is _not_ triggering the `process` function call in the Node, unlike with hot Inlets (by default) which trigger the `process` with every update. However the value is saved and passed to the next `process` call later, if some hot inlet triggered it.

```javascript
Rpd.nodetype('docs/microwave', {
    inlets: {
        'food': { type: 'docs/food' },
        'time': { type: 'docs/stopwatch', cold: true, default '1min' },
        'temperature': { type: 'docs/temperature', cold: true, default: 200 }
    },
    outlets: {
       'prepared-food': { type: 'docs/food' }
    },
    process: function(inlets) {
        // will be called only when there's some new food was received
        // i.e. time or temperature updates will be saved but won't trigger cooking
        return: {
            'prepared-food': Kefir.constant(
                prepareFood(inlets.food, inlets.temperature)
            ).delay(inlets.time)
        }
    }
});
```

<!-- TODO: test -->

##### `readonly`: `boolean`

In this case the name of the flag does _not_ mean that user is unable to change the value of the Inlet at all, user still can do it with connecting Inlet to some Outlet, but when Style allows Value Editors near to Inlets in general, this Inlet will have none.

Value Editors are small inputs usually shown when user clicks the value of the inlet and they allow to change the value without the connections. Though not every Channel Type has the Editor, or Style may disable all the Editors, so even while this option is `true` by default, there is no guarantee that there will be an Editor for a value there.

<!-- TODO: change to global rendering configuration -->

##### `allow`: `array[string]`

The list of the Outlet (Channel) Types this Inlet accepts to connect. By default every Inlet accepts only connections from the same Channel Type. When user tries to connect Outlet which type is not on the list, connection is not established and the error is fired. <!-- TODO: test -->

So, Outlet with `util/color` type may always be connected to any `util/color` Inlet, but it can not be connected to `util/nummer` Inlet in any case, unless this Inlet Type,  or this Inlet in particular, has `util/color` in `allow` list. <!-- TODO: check -->

```javascript
Rpd.channeltype('docs/time', {
    allow: [ 'util/number' ],
    adapt: function(value) {
        if (Number.isNumber(value)) {
            return new Date(value); // convert from milliseconds
        } else {
            return value;
        }
    }
});
```

By default, all of the Inlets have `core/any` in allow list, but when user overrides this list, user should include `core/any` there manually, if she wants to allow these connections. <!-- FIXME: correct tests -->

##### `accept`: `function`: `(value) → boolean`

This function allows you to skip/decline some incoming values basing on the value itself, before they come to the `process` handler.

```javascript
Rpd.channeltype('docs/byte', {
    allow: [ 'util/number' ],
    accept: function(value) {
        return (value >= 0) && (value <= 255);
    }
});
```

Actually if you _filter_ the stream of values with `tune` function, the result will be the same, but `accept` function allows you not to mess with the streams for a simple cases when you really don't need to.

Receives Inlet instance as `this`.

##### `adapt`: `function`: `(value) → value`

You may convert every incoming value to some another value or append some data to it, before it comes to the `process` handler.

```javascript
Rpd.channeltype('docs/byte', {
    allow: [ 'util/number' ],
    adapt: function(value) {
        if (value < 0) return 0;
        if (value > 255) return 255;
        return value;
    }
});
```

Actually if you _map_ the stream of values with `tune` function, the result will be the same, but `adapt` function allows you not to mess with the streams for a simple cases when you really don't need to.

Receives Inlet instance as `this`.

##### `show`: `function`: `(value) → string`

This function is called by Renderer when it shows the Inlet value near to it. By default, it just uses `.toString` over the value.

It is useful to convert complex values to some short summaries here. For example, when your Channel sends arrays as values, it is better to shorten the description just to the length of array and what type of elements are inside.

```javascript
Rpd.channeltype('docs/radians', {
    allow: [ 'util/number '],
    accept: function(value) {
        return (value >= 0) && (value <= (2 * Math.PI));
    }
    show: function(value) {
        // just an example, do not use in production
        var degrees = Math.round(value / Math.PI * 180);
        if (degrees == 360) return '2π';
        if (degrees > 270) return '3π/2 > α > 2π';
        if (degrees == 270) return '3π/2';
        if (degrees > 180) return 'π > α > 3π/2';
        if (degrees == 180) return 'π';
        if (degrees > 90) return 'π/2 > α > π';
        if (degrees == 90) return 'π/2';
        if (degrees > 0) return '0π > α > π/2';
        return '0π';
    }
});
```

<!-- TODO: test -->

Receives Inlet instance as `this`.

##### `tune`: `function`: `(values_stream) → values_stream`

With the help of `tune` function you may freely modify the incoming stream of values, delay them, filter them or even reduce them to something else. When you know the power of [Streams][kefir], you are literally the Master of this Inlet. On the other hand, when something unpredictable happens with values coming through, you may confuse the user, so if you hardly modify them, please pay attention to additionally describe or demonstrate why/how you do it, in the UI of your Node or somewhere nearby.

```javascript
Rpd.channeltype('docs/synchronized', function() {
    tune: function(stream) {
        // emit last received value exactly with a second pause
        return stream.throttle(1000);
    }
});
```

<!-- TODO: test -->

Receives Inlet instance as `this`.

##### `handle`: `object`

This object allows you to subscribe to any event this Node produces. _Key_ in this object is the event name, and _value_ is the handler. See [Events](#) section for the complete list of the events.

An example:

```javascript
```

Every handler receives Inlet instance as `this`.

----

#### `inlet.receive(value)`

Force this inlet to receive some specific value, overpassing the connections, if there are any.

Channel mechanics are involved only partly in this case, but the value is still checked if it is allowed by channel type, and if it does, then it is adapted following the channel type definition. <!-- TODO: ensure -->

When inlet is cold, it also can postpone sending the value, till other hot inlet triggers node update.

```javascript
myNode.addInlet('docs/number', 'num').receive(42);
myNode.addInlet('docs/date', 'date').receive(Date.parse('Apr 10, 2015'););
myNode.addInlet('docs/radians', 'angle').receive(Math.PI);
```

#### `inlet.stream(stream)`

Force this inlet to receive stream of values. RPD uses `Kefir` library to provide streams. Value streams provide practically infinite possibilities, you can send values with time intervals, throttle values by time, combine different streams in unlimited ways, actually everything.

You may find complex examples at [Kefir library page][kefir]. Also, usually it is quite easy to convert streams from some another Stream-based library, like RxJS, when you want to use such.

```javascript
// empty object is treated like a bang trigger for `util/bang` channel instances
myNode.addInlet('util/bang', 'bang').stream(Kefir.interval(3000, {}));

// control amount of red in the color using X position of a mouse,
// send it every time mouse position was changed
myNode.addInlet('util/color', 'color').stream(
    Kefir.fromEvents(document.body, 'mousemove')
         .map(function(event) {
             return { x: event.clientX, y: event.clientY };
         })
         .map(function(position) {
             return position.x % 255;
         })
         .map(function(rValue) {
             return { r: rValue, g: 255, b: 255 };
         })
);

// send how many milliseconds passed from the start time approx. every second
var start = new Date();
myNode.addInlet('util/time', 'passed').stream(
    Kefir.fromPoll(1000, function() { return new Date() - start; })
);
```

#### `inlet.toDefault()`

Force default value to be sent into this inlet, breaking its normal flow. It is not recommended to use it often, it is used mostly used in proper cases by RPD itself, but in case when you really need it, just know it's there.

#### `inlet.allows(outlet)`

Check if this inlet allows connections from given outlet. Usually it us done by the renderer <!-- ? --> on connection, but if you want to ensure connection will pass, you may use this method.
