<!-- MARK: Link -->

### `Link`

Link represents a single connection between Inlet and Outlet <!-- what happens when the connection was declined? -->. By default, one Outlet can be connected to several Inlets, but for Inlets it is not allowed to have more than one incoming connection. This is configurable through `config.inletAcceptsMultipleLinks`, though. `Link` instance is returned from `outlet.connect` method.

#### `link.pass(value)`

Send a value through the link. The rather logical difference with `outlet.send()` is that this action does not apply any of the Outlet value modifiers, like `tune` function. On the other hand, when value comes to the connected Inlet, it applies all the usual modifiers to the value, as if it was sent from the Outlet. <!-- TODO: check --> Also, in default configuration, Outlet may be connected to several Inlets, so in this case value is sent only through this particular link to a single connected Inlet, instead of many.

#### `link.enable()`

Enable this link, so values will flow normally through, as just after the connection. Opposite to `link.disable()`.

#### `link.disable()`

Disable the link temporarily, but the connection actually stays. Practically the same as filtering all the values going through the link.

#### `link.disconnect()`

Remove the connection between given Outlet and Inlet. For ever. Unless new one will be established.
