## Node Type/Instance Renderer

<img src="./assets/instance-renderer.png" width="200px" alt="Node Instance Renderer"></img>

_Node Type Renderer_ builds the body of the Node and may update its content when some incoming update triggered it. Also, it may send values from inner controls to a hidden Inlets of the Node.

There should be a separate Node Type Renderer for each way to render a node, such as HTML, SVG and so on. By default, if Node can't render itself in the requested way, it is rendered as an empty Node, yet having all the defined Inlets and Outlets.

_Node Instance Renderer_ has exactly the same definition structure, it just overrides the Node Type Renderer so you can render any specific Node instance completely another way with just re-defining Type Renderer inline.