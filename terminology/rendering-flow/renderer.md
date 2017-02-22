## Renderer

<img src="./assets/renderer.png" width="100px" alt="Renderer"></img>

_Renderer_ is a system which determines the way current Patch model is rendered. For now, there are two Renderers: HTML and SVG, they render Patches to HTML or SVG tags correspondingly. For instance, HTML Renderer renders Link connections as `span` blocks with CSS borders and SVG Renderers just draws SVG `line` tags for the same purpose.

Also, Renderer determines where new node will be placed if position was not specified.