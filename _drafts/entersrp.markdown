---
layout: post
title:  "Enter SRP"
date:   2020-02-16 19:43:31 +0100
categories: 
---
# Enter SRP

When I joined Unity a little bit more than 3 years ago there was a huge excitement in Unity about new rendering pipelines. SRP (Scriptable Rendering Pipeline) became the buzz word in the graphic foundation team. The ability to describe rendering code in C# land was exciting to me. From one perspective this would allow developers to customize Unity in a level not possible before, on the other hand it would allow students to learn Computer Graphics techniques at a high level. The past three years I've been developing the Universal Render Pipeline on top of the SRP library. Now it's time to start teaching Computer Graphics using SRP.

So what is this SRP and why is that important if you are a student or professional in the area?
In short, SRP allows to describe at a high-level rendering code in Unity. Just like couple of decades ago GPUs allowed programmable pipeline stages, with SRP, Unity is allowing developers to program large parts of rendering in C#. A render pipeline implements a strategy that culls objects and lights and then perform a series of render steps (compute or render passes) that composite a final frame. In [this article](http://www.adriancourreges.com/blog/2016/09/09/doom-2016-graphics-study/) you can see an example of a high level breakdown of the render pipeline of DOOM (2016).

![Doom](assets/doom.jpg)

If you are a student, you can use this to learn Computer Graphics rendering tehcniques such as forward, deferred and tile/clustered lighting. If you are a professional you can create highly customizable production pipeline for your game or application.

## 1. Getting Started - Setting up your development environment

OK. So hopefully that got you excited! Let's show you how to get Unity configured to start using SRP. 

### 1.1 Create a project with Unity 2019.3.0f6 or above

Follow [these instructions](https://docs.unity3d.com/Manual/GettingStarted.html) to install Unity and create a new 3D project template.

### 1.2 Install Core RP Library 7.2.0 or above package

Open the newly created Unity project. On the top menu bar, click on __Window -> Package Manager__.
A new window will open. You can now search for Core RP. Select the __7.2.0__ or above version and click on __install__. 

![Package Manager](assets/packagemanager.JPG)
---
**NOTE:** 
If you don't see package 7.2.0. Click on __show all versions__ under the Core RP package tab.

---

### 1.3 Change your project's color space to linear

Click on __Edit -> Project Settings__. Then now under __Player__ panel, select __Linear__ color space. This will give us correct lighting curve for rendering.

![Linear Color space](assets/linearcolorspace.JPG)

## SRP Overview

From a top down approach. A custom render pipeline contains high-level rendering code. A typical render pipeline will implement a strategy to cull objects and lights and execute a combination of drawing commands, compute and post-processing to composite the final image.

The interface between these high-level commands and Unity is given by the SRP Core Library, a very thin API layer that handles the render pipeline and shader library abstraction. 

The core Unity graphics backend contains several systems that can handle large groups of objects and batch, sort and set per-object GPU data. The core Unity is heavily jobified and prepare work for the GPU on the render thread, optionally dispatching additional worker threads.

The core also contains large system such as Global Illumination and Terrain.

Let's take a closer look at what are the core components of SRP Core Library and how to start creating our own render pipeline.

![SRP](assets/SRP.JPG)

__Render Pipeline Asset__: The pipeline asset is a factory class reponsible for creating the render pipeline instances. The render pipeline asset holds any resources that might be needed for the renderer pipeline instance such as materials, shaders, meshes, and so on.

__Render Pipeline__: A render pipeline must implements a render callback that describes how Unity how to render a frame.

__Scriptable Render Context__: The render context is the interface between the C# user land and the core Unity graphics. From a render context you can perform CPU culling and enqueue commands to be executed by the GPU.

__Shader Library__: Core RP Library contains a minimalistic Shader Library written in HLSL. This contains helper functions to do matrix transforms, texture sampling macros, BDRFs, packing/unpacking, among other things. when creating a custom render pipeline you will most likely need to extend this minimalistic shader library with your own. 

So for this lesson, let's get started by creating a `CustomRenderPipelineAsset01` and `CustomRenderPipeline01`. Here's how you do it:

```
using UnityEngine;
using UnityEngine.Rendering;

[CreateAssetMenu]
public class CustomRenderPipelineAsset01 : RenderPipelineAsset
{
    protected override RenderPipeline CreatePipeline()
    {
        return new CustomRenderPipeline01(this);
    }
}

public class CustomRenderPipeline01 : RenderPipeline
{
    public CustomRenderPipeline01(CustomRenderPipelineAsset01 asset)
    {

    } 

    protected override void Render(ScriptableRenderContext context, Camera[] cameras)
    {

    }
}
```

For now our `CustomRenderPipelineAsset01` is very straight forward. We subclass from `RenderPipelineAsset` and implement the `CreatePipeline` method that creates the render pipeline. `CreatePipeline` is called by Unity to create the pipeline instance before rendering if necessary.

The __[CreateAssetMenu]__ is property that tells Uniyt to create a menu in __Assets -> Create__. 

The `CustomRenderPipeline01` implements the `Render` method that is called by Unity to render a frame. This is our entry point to Unity custom rendering.

Now let's make our render pipeline clear the screen to _black_.

```
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

The above piece of code creates a command buffer to set the render target and clear color to black and depth. `BuiltinRenderTextureType.CameraTarget` is a handle to the framebuffer.

TODO: Explain in hidden note about framebuffer.

We execute the command buffer by calling `context.ExecuteCommandBuffer(cmd)`. This makes a copy of `cmd` and enqueues it to be executed. It's very important to know that at this point it nothing has been executed yet. All commands are recorded in the context. It's when we call `context.Submit()` then the actual execution happends. You can call `Submit` once or multiple times in a frame. It depends on the amount of work you have to do. Calling `Submit` can help to have a better parallelism of __main__ and __render thread__.

In Unity, now you can click on `Assets -> Create -> CustomRenderPipelineAsset01`. 
![Create](assets/create.png)

---
**NOTE:** 
If you don't see the menu, double check you don't have any compiler errors in your scripts.

---


This will create a pipeline asset in your project folder. Assign the asset in Graphics Settings to tell Unity to use your custom render pipeline. 

![Graphics Settings](assets/graphicssettings.JPG)

Your game and scene view should should look just be a black image now. There you are! You've create your own first (and very effient) renderer.

You will notice that now your game and scene view, and even previews are black. This is because when you override Unity with a custom render pipeline you are not only overriding the game but every camera type that is used to render in Unity. Scene, Game, Preview, even reflection probes will be replaced by your custom rendering.

## What's next?
For the next lesson we will learn about how to create shaders and draw your first triangle.

## What If I got lost?
You can check the final scripts and project in this github. Open the project and you will find all scripts and the Unity scene in the folder `01`.



