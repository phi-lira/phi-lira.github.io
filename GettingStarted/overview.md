---
layout: default
title:  Overview
nav_order: 3
description: 
parent: Getting Started
---

## SRP Overview

A scriptable render pipeline implements a callback written in C# that tells Unity how to render a frame. 

A render pipeline implements a strategy that culls objects and lights and then perform a series of render steps (compute or render passes) that composite a final image. 

In [this article](http://www.adriancourreges.com/blog/2016/09/09/doom-2016-graphics-study/) you can see an example of a high level breakdown of the render pipeline of DOOM (2016).

![Doom]({{ site.url }}/assets/images/doom.jpg)

The interface between these high-level commands and Unity is given by the SRP Core Library, a very thin API layer that handles the render pipeline and shader library abstraction. 

The core Unity graphics backend contains several systems that can handle large groups of objects and batch, sort and set per-object GPU data. The core Unity is heavily jobified and prepare work for the GPU on the render thread, optionally dispatching additional worker threads.

The core also contains large system such as Global Illumination and Terrain.

![SRP]({{ site.url }}/assets/images//SRP.JPG)

Let's take a closer look at what are the core components of SRP Core Library and how to start creating our own render pipeline.

__Render Pipeline Asset__: The pipeline asset is a factory class reponsible for creating the render pipeline. The asset holds any resources that might be needed for the renderer pipeline such as materials, shaders, meshes, and so on. It also contains callbacks that will customize the Unity editor to create materials and shaders that are suitable for the render pipeline.

__Render Pipeline__: A scriptable render pipeline implements a callback written in C# that tells Unity how to render a frame. When overriding Unity with a custom render pipeline all cameras are now rendered using this render pipeline. This includes not only the game cameras but also the Scene View, Preview and even reflection probe cameras.

__Scriptable Render Context__: The render context is the interface between C# the core Unity graphics. From a render context you can perform culling and enqueue commands to be executed by the GPU.

__Shader Library__: Core RP Library contains a minimalistic shader library written in HLSL. This contains helper functions to do matrix transforms, texture sampling macros, lighting evaluation functions, packing/unpacking functions, among other things. When creating a custom render pipeline you will most likely need to extend this minimalistic shader library with your own. 

Let's get started by creating a `CustomRenderPipelineAsset01` and `CustomRenderPipeline01`. Here's how you do it:

```csharp
using UnityEngine;
using UnityEngine.Rendering;

[CreateAssetMenu]
public class CustomRenderPipelineAsset : RenderPipelineAsset
{
    protected override RenderPipeline CreatePipeline()
    {
        return new CustomRenderPipeline(this);
    }
}
```

```csharp
public class CustomRenderPipeline : RenderPipeline
{
    public CustomRenderPipeline01(CustomRenderPipelineAsset01 asset)
    {

    } 

    protected override void Render(ScriptableRenderContext context, Camera[] cameras)
    {

    }
}
```

For now our `CustomRenderPipelineAsset` is very straight forward. We subclass from `RenderPipelineAsset` and implement the `CreatePipeline` method that creates the render pipeline. `CreatePipeline` is called by Unity to create the pipeline instance before rendering if necessary.

The __[CreateAssetMenu]__ is property that tells Uniyt to create a menu in __Assets -> Create__. 

The `CustomRenderPipeline` implements the `Render` callnack that is called by Unity to render cameras. This callback is called once for all game cameras and then again for each Scene View, Preview and Reflection Probe camera. 

Now let's make our render pipeline clear the screen to _black_.

```csharp
 protected override void Render(ScriptableRenderContext context, Camera[] cameras)
 {
    CommandBuffer cmd = CommandBufferPool.Get();
    cmd.SetRenderTarget(BuiltinRenderTextureType.CameraTarget);
    cmd.ClearRenderTarget(true, true, Color.black);
    context.ExecuteCommandBuffer(cmd);
    CommandBufferPool.Release(cmd);

    context.Submit();
 }
```

First, we get a command buffer from the `CommandBufferPool`. Command buffers are the main mechanism we will use to schedule work to be executed by the GPU. Then we call `SetRenderTarget` to set the pipeline render target. `BuiltinRenderTextureType.CameraTarget` is a built-in Unity handle to the framebuffer or to the camera target texture.

TODO: Explain in hidden note about framebuffer.

We clear the render target to black. The first and second parameters tell Unity to clear the color and/or depth buffers.

Finally we execute the command buffer by calling `context.ExecuteCommandBuffer(cmd)`. This makes a copy of `cmd` and enqueues it to be executed. It's very important to know that at this point, nothing has been submitted to the GPU yet. All commands are recorded in the render context. It's when we call `context.Submit()` that the actual execution happens. 

NOTE: Add info about __main thread__ and __render thread__.

You can call `Submit` once or multiple times in a frame. It depends on the amount of workload you have. Calling `Submit` early in the pipeline can help to have a better parallelism of __main thread__ and __render thread__.

Switch back to Unity and wait until scripts are compiled.Now you can click on `Assets -> Create -> CustomRenderPipelineAsset01`. 
![Create]({{ site.url }}/assets/images/create.png)

---
**NOTE:** 
If you don't see the menu, double check you don't have any compiler errors in your scripts.

---


This will create a pipeline asset in your project folder. Assign the asset in Graphics Settings to tell Unity to use your custom render pipeline. 

![Graphics Settings]({{ site.url }}/assets/images//graphicssettings.JPG)

Your game and scene view should should look just be a black image now. 

You will notice that now your game and scene view, and even previews are black. This is because when you override Unity with a custom render pipeline you are not only overriding just the game camera but every camera type that is used to render in Unity. Scene, Game, Preview, even reflection probes will be replaced by your custom rendering.

There you are! You've create your own first (and very effient) renderer.

## What If I got lost?
You can check the final scripts and project in this [github](https://github.com/phi-lira/learnsrp). Open the project and you will find all scripts and the Unity scene in the folder `01`.

## Exercise
In the `Render` callback we receive a list of cameras.
Change our render pipeline to loop through all cameras and clear the screen with the camera background color.

## What's next?
For the next lesson we will learn about how to create shaders and draw your first triangle.

