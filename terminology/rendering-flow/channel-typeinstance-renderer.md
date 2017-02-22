## Channel Type/Instance Renderer

<img src="./assets/instance-renderer.png" width="200px" alt="Channel Instance Renderer"></img>

_Channel Type Renderer_ builds the Inlet/Outlet value representation and also may add the editor to a Channel value.

Editor is an optional control which lets user override the value in the Inlet.

_NB: For the moment, value editors are only supported in HTML Renderer_

_Channel Instance Renderer_ has exactly the same definition structure, it just overrides the Channel Type Renderer so you can render any specific Channel instance completely another way with just re-defining Type Renderer inline.