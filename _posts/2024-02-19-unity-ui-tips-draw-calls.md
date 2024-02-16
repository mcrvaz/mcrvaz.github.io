---
title: "Unity UI Tips: Draw calls"
date: 2024-02-15 00:00:00 -03:00
classes: wide
categories:
  - blog
tags:
  - unity
  - c#
  - ui
  - quick-tips
gallery-2:
  - url: /assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-atlas-01.png
    image_path: /assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-atlas-01.png
    alt: "Frame debugger with two draw calls from different textures."
  - url: /assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-atlas-02.png
    image_path: /assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-atlas-02.png
    alt: "Frame debugger with two draw calls from different textures."
---

There are a few ways to optimize your game's UI in Unity, but today we'll focus on the [draw calls](https://docs.unity3d.com/Manual/optimizing-draw-calls.html).

To keep your draw calls to a minimum, you need to be aware of a few rules for batching. 
But before we start, let's take a look at the glorious [Frame Debugger](https://docs.unity3d.com/Manual/frame-debugger-window.html).

## Frame Debugger
I really recommend you to read the documentation if you're unfamiliar with it, although it's simple to use. 
We'll focus on basically three things here:
1. Draw call count (less is better)
2. Draw call selection
3. Textures

I've marked them in the image below:

[![small image](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-03.png)](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-03.png)
<figcaption>The batch break section looks very interesting, but unfortunately it doesn't work for UI.</figcaption><br>
[![small image](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-02.png)](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-02.png)
<figcaption>Here you can see how the game view changes when you select a different draw call.</figcaption><br>

And now that we're properly equipped, let's move forward to the batching rules.

## Sprite Atlas
Unity can only batch draw calls with sprites that belongs to the same texture. A [Sprite Atlas](https://docs.unity3d.com/2022.1/Documentation/Manual/SpriteAtlasV2.html) is a big texture containing a bunch of sprites. Sprites that aren't assigned to any atlas won't be batched, except with themselves.

While this is pretty straightforward, you'll need to be smart about your atlases organization, specially in platforms with tight memory constraints, for instance, 2048x2048 is usually the safe maximum size for Android devices. This means that you can't simply dump everything in one single atlas most of the time. 

With the Frame Debugger you can check the "Textures" section and see if it's the same texture than the previous/next draw call.

[![small image](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-atlas-01.png)](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-atlas-01.png)
[![small image](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-atlas-02.png)](/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/drawcall-atlas-02.png)
<figcaption>Those elements have different textures, hence the two draw calls.</figcaption><br>

Hierarchy also matters here, consider this situation:

// TODO pic

Even though X and Z belongs to the same atlas, Y doesn't. This way we end up with 3 different draw calls, but we can reorganize it and cut it down to 2 draw calls.

// TODO pic

It's not always easy or even possible to reorganize hierarchies, but big improvements can be found here.

## Materials
Same as with textures, elements with different materials can't be batched together. With that in mind, it's good to keep the material count low and sometimes it might make sense to see if you can reach the same effect without changing materials.

// TODO pic

## Canvas
Only elements that belongs to the same canvas can be batched. This one is a bit more complex because sometimes it's actually [good to break things into multiple canvases](https://unity.com/how-to/unity-ui-optimization-tips), so before dumping everything into the same canvas to keep your draw call count low, try to profile both scenarios and see which one performs better.

// TODO pic

## Masking
Whenever you add a [Mask component](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-Mask.html) to your UI, it generates a new draw call for the element being masked. 

// TODO pic

You can also use the [RectMask2D](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-RectMask2D.html) to reduce your draw calls. With it you can limit the amount of stuff being rendered to the masked area. This is specially usefull with [ScrollRect](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-ScrollRect.html).

// TODO pic

## Extras

### Different Z axis values
There's pretty much no reason to move screen space and overlay UIs on the Z axis. If you need to adjust the rendering order, you can either change the hierachy or the [Canvas Sorting Order]({% post_url 2024-01-29-unity-ui-tips-canvas-sorting-order %}). Besides not really working, it can also break your draw call batches, so just don't do it.

### Invalid height and width values
Sometimes your UI elements end up with zero height and width values, specially when dealing with layout groups, and this can possibly break batches with child elements.
It's a good practice to always leave your UI elements with reasonable dimension values, at least enough size to wrap its contents.

### Rethinking hierarchies
Sometimes you might have some complex UI scene with many elements, let's take this image as an example:

{% include figure image_path="/assets/images/posts/2024-02-19-unity-ui-tips-draw-calls/candy-crush.webp" alt="Candy Crush" caption="From [Candy Crush](https://www.king.com/game/candycrush)."%}

Think for a while about how you would structure the board.

You might be tempted to create a prefab that contains the whole board entry and everything that goes with it, background, the piece, glow effects, text effects, and all that stuff, but this is pretty bad regarding draw calls. You would need the background to be in the same sprite atlas as all the board pieces and all of the effects, besides, the text would certainly break some batches.

A better approach here would be to think in layers: have a layer for the background, a layer for the board pieces and one (or more) for the effects. This way all of those layers would be drawn with very few draw calls.
This might be a bit harder to code depending on how things are structured, but it can give a huge performance boost. So, if you're having excessive draw calls, try to explore new ways of defining your hierarchies.

## Conclusion
Follow good practices to avoid excessive draw calls, but don't go too crazy on it, after all, "premature optimization is the root of all evil". Before going all in, make sure that you actually have a performance issue. Sometimes your UI is already good enough!

While fewer draw calls are a good performance indicator, it's far from being the only one, so measure and compare things before commiting to any optimization.
