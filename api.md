# Contents

When you want to let user work with some existing Node Network or to load and build it from file (for which there is `io` module), you may use Network Building API, including these methods:

* [`Rpd`](#rpd)
    * [`Rpd.addPatch(name) → Patch`](#rpd-addpatch)
    * [`Rpd.addClosedPatch(name) → Patch`](#rpd-addclosedpatch)
* [`Patch`](#patch)
    * [`patch.addNode(type, [title, definition]) → Node`](#patch-addnode)
    * [`patch.removeNode(node)`](#patch-removenode)
    * [`patch.inputs(list)`](#patch-inputs)
    * [`patch.outputs(list)`](#patch-outputs)
    * [`patch.project(node)`](#patch-project)
* [`Node`](#node)
    * [`node.addInlet(type, alias[, definition]) → Inlet`](#node-addinlet)
    * [`node.addOutlet(type, alias[, definition]) → Outlet`](#node-addoutlet)
    * [`node.removeInlet(inlet)`](#node-removeinlet)
    * [`node.removeOutlet(outlet)`](#node-removeoutlet)
    * [`node.turnOn()`](#node-turnon)
    * [`node.turnOff()`](#node-turnoff)
* [`Inlet`](#inlet)
    * [`inlet.receive(value)`](#inlet-receive)
    * [`inlet.stream(stream)`](#inlet-stream)
    * [`inlet.toDefault()`](#inlet-todefault)
    * [`inlet.allows(outlet) → boolean`](#inlet-allows)
* [`Outlet`](#outlet)
    * [`outlet.connect(inlet) → Link`](#outlet-connect)
    * [`outlet.disconnect(link)`](#outlet-disconnect)
    * [`outlet.send(value)`](#outlet-send)
    * [`outlet.stream(stream)`](#outlet-stream)
* [`Link`](#link)
    * [`link.pass(value)`](#link-pass)
    * [`link.enable()`](#link-enable)
    * [`link.disable()`](#link-disable)
    * [`link.disconnect()`](#link-disconnect)

To control the rendering queue, you may use these methods:

* [`Rpd`](#rpd)
    * [`Rpd.renderNext(renderers, targets, config)`](#rpd-rendernext)
    * [`Rpd.stopRendering()`](#rpd-stoprendering)
* [`Patch`](#patch)
    * [`patch.render(renderers, targets, config)`](#patch-render)
    * [`patch.open()`](#patch-open)
    * [`patch.close()`](#patch-close)
    * [`patch.moveCanvas(x, y)`](#patch-movecanvas)
    * [`patch.resizeCanvas(width, height)`](#patch-resizecanvas)
* [`Node`](#node)
    * [`node.move(x, y)`](#node-move)    

When you want to build your own toolkit, you may decide to register your node & channel types and renderers using these methods:

* [`Rpd`](#rpd)
    * [`Rpd.nodetype(type, definition)`](#rpd-nodetype)
    * [`Rpd.channeltype(type, definition)`](#rpd-channeltype)
    * [`Rpd.noderenderer(type, alias, definition)`](#rpd-noderenderer)
    * [`Rpd.channelrenderer(type, alias, definition)`](#rpd-channelrenderer)
    * [`Rpd.nodedescription(type, description)`](#rpd-nodedescription)

<!-- * `Rpd.toolkiticon(toolkit, icon)` -->
<!-- * `Rpd.nodetypeicon(toolkit, icon)` -->

These methods will help you in creating your own styles or even renderers:

* [`Rpd`](#rpd)
    * [`Rpd.style(name, renderer, style)`](#rpd-style)
    * [`Rpd.renderer(alias, renderer)`](#rpd-renderer)

<!-- TODO: global `Rpd` object properties -->

To define node type or channel type, to configure some particular node or channel, to define node renderer or channel renderer, you'll need these Definition Objects:

* [Node Definition](#node-definition)
* [Inlet Definition](#inlet-definition)
* [Outlet Definition](#outlet-definition)
* [Node Renderer Definition](#rpd-noderenderer)
* [Channel Renderer Definition](#rpd-channelrenderer)

## Core types

* Channel
   * `core/any`
* Node
   * `core/basic`
   * `core/reference`

## `util` Toolkit

List of Nodes, which currently exist in `util` Toolkit [is located here](http://github.com/shamansir/rpd/blob/master/src/toolkit/util/toolkit.md).

## Naming rules

Probably you already noticed that naming style in API is different from method to method. I'd like to assure you that everything is under control and has a system before studying out any method. And the rules are simple:

* Static method for _registering_ Node Types, Channel Types, Renderers, Node Renderers, Channel Renderers, Styles etc.: `Rpd.completelylowercase`;
* Any other instance or static method: `instance.usualCamelCase`, preferrably one word;
* Node or Channel type name: `toolkit/word-or-two`;
* Property in a Node Definition, Channel Definition or any other Definition: strictly one word, lowercase;