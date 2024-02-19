---
title: "Unity UI Tips: Draw calls"
date: 2024-02-19 00:00:00 -03:00
excerpt: "There are a few ways to optimize your game's UI in Unity, but today we'll focus on the draw calls."
classes: wide
header: 
  teaser: "/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/candy-crush.webp"
categories:
  - blog
tags:
  - unity
  - c#
  - ui
  - quick-tips
---

There are a few ways to optimize your game's UI in Unity, but today we'll focus on the [draw calls](https://docs.unity3d.com/Manual/optimizing-draw-calls.html).

As defined in the docs, "a draw call tells the graphics API what to draw and how to draw it".
What we want here is to group more stuff in a single draw call, minimizing render state changes, which are expensive. To minimize those state changes, you need to develop your scene with a few rules in mind. 

I'll explain them in a bit, but before we start, let's take a look at the lovely [Frame Debugger](https://docs.unity3d.com/Manual/frame-debugger-window.html).

## Frame Debugger
I highly recommend reading the documentation if you're unfamiliar with it, although it's straightforward to use.
We'll focus on basically three things here:
1. Draw call count
2. Draw call selection
3. Textures

I've marked them in the image below:

[![small image](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-03.png)](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-03.png)
<figcaption>The batch break section looks very interesting, but unfortunately it doesn't work for UI.</figcaption><br>
[![small image](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-02.png)](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-02.png)
<figcaption>Notice how the game view changes when you select a different draw call.</figcaption><br>

And now that we're properly equipped, let's move forward to the batching rules.

# Batching rules for UI

## Sprite Atlas
Unity can only batch draw calls with sprites that belong to the same texture. A [Sprite Atlas](https://docs.unity3d.com/2022.1/Documentation/Manual/SpriteAtlasV2.html) is a big texture containing a bunch of sprites. Sprites that aren't assigned to any atlas won't be batched, except with themselves.

While Sprite Atlas are pretty straightforward, you'll need to be smart about their organization, especially in platforms with tight memory constraints, for instance, 2048x2048 is usually the safe maximum size for Android devices. This means that you can't simply dump everything in one single atlas, most of the time. 

With the Frame Debugger you can check the "Textures" section and see if it's the same texture as the previous/next draw call.

[![small image](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-atlas-01.png)](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-atlas-01.png)
[![small image](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-atlas-02.png)](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-atlas-02.png)
<figcaption>Those elements have different textures, hence the two draw calls.</figcaption><br>

## Materials
Similarly to textures, elements with different materials cannot be batched together. With that in mind, it's good to keep the material count low and sometimes it might make sense to see if you can reach the same effect without changing materials.

## Canvas
Only elements that belong to the same canvas can be batched. This one is a bit more complex because sometimes it's actually [good to break things into multiple canvases](https://unity.com/how-to/unity-ui-optimization-tips), so before dumping everything into the same canvas to keep your draw call count low, try to profile both scenarios and see which one performs better.

## Masking
Whenever you add a [Mask component](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-Mask.html) to your UI, it generates a new draw call for the element being masked.

But they're not always bad, you can also use the [RectMask2D](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-RectMask2D.html) to reduce your draw calls. With it you can limit the amount of stuff being rendered to the masked area. This is especially useful with [ScrollRect](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-ScrollRect.html). 

Check it out:
[![small image](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-mask-01.png)](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-mask-01.png)
<figcaption>A Mask component was applied to the "Viewport" element.</figcaption>

[![small image](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-mask-02.png)](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-mask-02.png)
<figcaption>A RectMask2D component was applied to the "Viewport" element.</figcaption><br>

You can see that RectMask2D is actually rendering less vertices (24 vs 48), that means that none of those invisible "HeartContainer" are being rendered. It also needed one less draw call than the Mask component variant.

## Extras

### Different Z axis values
There's pretty much no reason to move screen space and overlay UIs on the Z axis. If you need to adjust the rendering order, you can either change the hierarchy or the [Canvas Sorting Order]({% post_url 2024-01-29-unity-ui-tips-canvas-sorting-order %}). Besides not really working, moving UI elements on the Z axis can also break your draw call batches, so just don't do it.

### Bad positioning or invalid height and width values
Your UI elements need to wrap around all of their child elements, otherwise it can possibly break batches. This is very common when you're dealing with layout groups and for some reason the height or width values are stealthily changed to 0.

Besides the draw call issues, it's also harder to understand how the UI is structured when things are badly dimensioned. As a general rule, avoid having child elements positioned out of its parent bounds.

### Rethinking hierarchies
Sometimes you might have some complex UI scene with many elements, let's take this image as an example:

{% include figure image_path="/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/candy-crush.webp" alt="Candy Crush" caption="From [Candy Crush](https://www.king.com/game/candycrush)."%}

Think for a while about how you would structure the board.

You might be tempted to create a prefab that contains the whole board entry and everything that goes with it, background, the piece, glow effects, text effects, and all that stuff, but this is probably going to be pretty bad regarding draw calls. You would need the background to be in the same sprite atlas as all the board pieces and all of the effects, besides, the text would certainly break some batches.

A better approach here would be to think in layers: have a layer for the background, a layer for the board pieces and one (or more) for the effects. This allows elements within each layer to be rendered together, reducing the number of draw calls required.
This might be a bit harder to code depending on how things are structured, but it can give a huge performance boost. So, if you're having excessive draw calls, try to explore new ways of defining your hierarchies.

## Conclusion
Try to follow good practices to avoid excessive draw calls, but keep in mind that premature optimizations should be avoided. Before going all in, make sure that you actually have a performance issue. Your UI might already be good enough!

While fewer draw calls are a good performance indicator, it's far from being the only one, so measure and compare everything before committing to any optimization.
