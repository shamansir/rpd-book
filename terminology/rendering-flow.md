## Rendering Flow

<img src="./assets/render-flow.png" width="800px" alt="Render Flow"></img>

This sub-section is actually not about a specific Term and intended to quickly clarify how the things described below work together.

When you need to render a Patch, you need to know three things: which Renderer you want to use, which Style you prefer visually and the Target where you want everything to be injected. Style is optional, though, since `quartz` is the default one to be used, but sad things happen, and you may dislike it. So, to satisfy every taste, several Styles are included in RPD distribution. You actually may use same configuration to render several Patches, but let's leave it to [API docs](./api.html) for now.

The engine of the rendering process is Actions. They are fired on every change of the value, when new Connection between Outlet and Inlet is established, or instead someone removed the Connection, or someone moved one Node etc. To the Rendering System there is no matter who did this Action, User or API call.

> If you know what [Flux](https://facebook.github.io/flux/) or [Elm](http://elm-lang.org/) is, it is almost the same Action concept you've met there; if you don't know about both, just replace Action term with Event term here and below.

So, for example you chose `'svg'` Renderer to render your Patch. What happens then? On every user or logical Action, this Renderer is notified. Then this Renderer decides which (preferably, the tiniest one) part of the Network it should re-render. If Node or Channel (Inlet or Outlet) is required to be updated, it searches for the corresponding Node Renderer or Channel Renderer, the one assigned to this Renderer. Node/Channel Renderer can render particular type of the Node/Channel (and then it's called Node/Channel Type Renderer) or it may be overridden for the specific instance of the Node/Channel (and then it's called Node/Channel Instance Renderer). If something else was updated, Renderer re-renders this part itself, without delegation. No, actually, it sometimes passes some updates to Style, just Node/Channel Renderer step is skipped in this case.

In its turn, Node/Channel Renderer renders the body of the Node (`'svg'` Renderer uses SVG elements for that) or a value of the Channel (same, with SVG elements). Then the result of the rendering is injected into a structure, provided by selected Style.

And the final piece of DOM (or whatever) is injected into the Target, into a proper place. Target for `'html'` Renderer should be some HTML element like `<div />`, for `'svg'` Renderer it should be some SVG element, preferably `<svg>` or `<g />`, and so on.

That's it, now users sees the whole thing in dynamic.

Now, let's describe the same process from participants' points of view.