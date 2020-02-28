---
layout: default
title:  The first triangle
nav_order: 4
parent: Getting Started
---

# The first triangle

In the previous lesson we learned how to override Unity rendering and clear the screen to black. 

In this lesson you will learn about the graphics pipeline and how to get a colorful triangle on screen. In order to learn about how to render a triangle let's take a brief look at how a GPU renders it. 

If you look at a GPU as a black box system. It takes some data (meshes, materials, textures, shaders), processes drawing commands (`drawcalls`) in several pipelined stages and output pixels into a buffer (`render target`). Then some dedicated hardware reads this buffer and present the image on the screen. This is what is called a graphics pipeline. 

In the very early days of GPUs, these pipeline stages were all fixed functions. In other words, they were not programmable. The increased demand for more content and more customization has driven the GPU vendors to expose programable stages in their pipeline. These stages are programable by the application with a specific type of program called `shader`. Graphics pipelines vary a lot from different types of GPUs. NVidia GPUs have pipelines with fewer stages that rely heavily on compute shaders. Mobile GPUs have deep pipeline stages that focus on cache locality when rendering triangles. These are known as tile based GPUs. You can read in more detail about graphics pipelins for different GPUs here, here and here.

For this lesson, we will focus on explaining some common pipeline stages used in most GPUs. For simplicity we are skipping a few fixed and programable stages. In our graphics pipeline, for each drawcall, these stages are executed:

__Vertex Shader__: Receives as input vertex attributes (position, uv, color, normal). Processes each vertex, transforming their position from a local coordinate system (object space) to a coordinate system that can be easily mapped to screen (homogeneous clipping space). The other attributes can be either processed in some way or passed as is to next stage.

__Primitive Assembly__: Creates topology information by assembling how vertices form polygons. In majority of cases, triangles are used a primitive and every three consecutive vertices forms a triangle.

__Rasterization__: Receives the processed vertex attributes and interpolates the values to generate attributes for each fragment. Optionally, this stage can know in some situations if the fragment is visible and don't dispatch any work for them in this case.

__Fragment Shader__: Receives the fragment attributes and outputs a color for each each one. This is usually where the most intense work happens. The fragment shader can use uvs to sample texture maps and use normals to compute lighting.

__Output Merger__: Receives the fragment color and applies further logic to decide how to write this color to the render target. This is where blend modes can be setup to achieve transparency.

In short, if we want to draw a triangle we have to:
1. Write a vertex and a fragment shader.
2. Optionally setup shader input data.
3. Issue a draw call command that tells the pipeline the vertex attributes and the primitive type.
4. Configure render state that tells fixed parts of the pipeline (rasterizer and output merged) to perform operations with fragments.

For simplicity in this lesson we will only cover the creation of vertex, fragment and drawcalls parts. We will introduce configuring drawcall data and render state in next lesson. 

So let's start with shaders! In Unity you can create a shader file by clicking on `Asset -> Create -> Shader -> Unlit Shader`. Sadly this is not a good shader template but it's the most simple one.

```csharp
Shader "LearnSRP/Triangle"
{
    SubShader
    {
        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "com.unity.render-pipelines.core/Common.hlsl"

            struct Attributes
            {
                float4 positionOS : POSITION;
            };

            struct Varyings
            {
                float4 positionHCS : SV_POSITION;
            };

            Attributes vert (Attributes IN)
            {
                Varyings OUT;
                OUT.positionHCS = IN.positionOS;
                return OUT;
            }

            half4 frag (Varyings IN) : SV_Target
            {
                return half4(1.0, 1.0, 1.0, 1.0);
            }
            ENDHLSL
        }
    }
}
```

Unity uses hlsl and a custom syntax called ShaderLab to make add further information to shaders. 
```csharp
Shader "LearnSRP/Triangle"
```

This line gives a name to this shader that we can search for and use to assign to materials.

```csharp
Subshader {
```

Every shader has to be in a Subshader block. We will skip talking about Subshaders for now.

```csharp
Pass {
```

Every vertex and fragment shader must be in a pass block. A shader can have multiple passes. For now we only need one pass to draw the triangle.

```csharp
HLSLPROGRAM
#pragma vertex vert
#pragma fragment frag
...
ENDHLSL
```

Here we start actual HLSL code. We tell the vertex program is implemented in a function called `vert` and the fragment program in a function called `fragment`.

```csharp
struct Attributes
{
    float4 positionOS : POSITION;
};

struct Varyings
{
    float4 positionHCS : SV_POSITION;
};
```

Here we declare that this shader is taking mesh that contains position attribute. Every shader must have at least a position attribute. The syntax used for attributes in a shader is:
`type varname : semantic`. Semantic binds to the variable what type of attribute this variable holds. There are several possible [semantics for vertex shader](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-semantics#vertex-shader-semantics).

Then we declare the `Varyings` struct to hold pixel shader semantics. The vertex shader writes to the fields in Varyings, the rasterizer interpolates them and given them as input to the fragment shader. Similarly to the vertex semantics, the fragment must have [semantics as well](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-semantics#pixel-shader-semantics).

Our vertex shader is very simple:
```csharp
Attributes vert (Attributes IN)
{
    Varyings OUT;
    OUT.positionHCS = IN.positionOS;
    return OUT;
}
```

It basically assigns the vertex position (Object Space) to the output position in clip space. We don't apply any transformation here because when we passed in the vertex positions in c# we alredy passed them in the clip space.

Finally the fragment shader output a white color to the render target. The [SV_Target semantic](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-semantics#system-value-semantics) tells it to write to whatever color buffer is bind as render target. 

```csharp
half4 frag (Varyings IN) : SV_Target
{
    return half4(1.0, 1.0, 1.0, 1.0);
}
```

Now let's switch back to our render pipeline and learn how to do a __drawcall__.

```csharp
public class CustomRenderPipeline01 : RenderPipeline
{
    Material triangleMaterial;
    Mesh triangleMesh;
```

Let's first declare a __Mesh__ to hold our triangle data and a __Material__ to hold our shader data.

In the render pipeline constructor we will create a shader and assign it to our material.

```csharp
Shader shader = Shader.Find("LearnSRP/01/TriangleShader");
triangleMaterial = CoreUtils.CreateEngineMaterial(shader);
```            

Here we use the shader name to find our shader in the Unity editor and create a material referencing it.

Now we need to create a mesh. Since our shader is expecting that we pass in the vertex information already in clip space, let's learn about what is this clip space system. 

[figure]

In short, it's a normalized space with x and y in the range of [-1, 1] and z in the range of [0, 1]. Note for OpenGL x, y, and z are in [-1, 1] range.

So let's create our vertex position to have a triangle in the center.

```csharp
Vector3[] vertices =
{
    // bottom-left corner
    new Vector3(-0.5f, -0.5f, 0.0f),

    // top corner
    new Vector3(0.0f, 0.5f, 0.0f),

    // bottom right corner
    new Vector3(0.5f, -0.5f, 0.0f),
};
```

We need to also tell how these vertices form a triangle. We do this by creating a index buffer that references our vertices. Every three consective indices in this buffer form a triangle.

```csharp
ushort[] indices = { 0, 1, 2 };
```

We now create a mesh, set the vertices and index buffer and upload submit the mesh data to GPU.
```csharp
triangleMesh = new Mesh();
triangleMesh.SetVertices(vertices);
triangleMesh.SetColors(colors);
triangleMesh.SetIndices(indices, MeshTopology.Triangles, 0);
triangleMesh.UploadMeshData(true);
```

Because we are creating some resources, we need to make sure we also dispose them. The render pipeline implements the `IDisposable` interface. We can them implement the `Dispose` to release our resources.

```csharp
protected override void Dispose(bool disposing)
{
    base.Dispose(disposing);
    if (disposing)
    {
        Material.DestroyImmediate(triangleMaterial);
        Mesh.DestroyImmediate(triangleMesh);
    }
}
```

Now that we have the mesh and shader data created all that we have to do is issue the draw call. 
In our `Render` method let's add `DrawMesh` just before executing the command buffer.

```csharp
cmd.ClearRenderTarget(true, true, Color.black);
cmd.DrawMesh(triangleMesh, Matrix4x4.identity, triangleMaterial, 0, 0);
context.ExecuteCommandBuffer(cmd);
```

If you switch back to Unity and open the __Game View__ you should see a boring white triangle on screen. Let's add some color now. 

In our render pipeline constructor, let's create a color attribute buffer.

```csharp
Color[] colors =
{
    new Color(1.0f, 0.0f, 0.0f, 1.0f),
    new Color(0.0f, 1.0f, 0.0f, 1.0f),
    new Color(0.0f, 0.0f, 1.0f, 1.0f),
};
```

and set it to our mesh like following:

```csharp
triangleMesh.SetVertices(vertices);
triangleMesh.SetColors(colors);
triangleMesh.SetIndices(indices, MeshTopology.Triangles, 0);
```

Finally we have to tell our shader to use this data:
In __Attributes__ let's add our new color buffer:
```csharp
struct Attributes
{
    float4 positionOS : POSITION;
    half4 color : COLOR;
```

We also add in __Varyings__ because we want to have the rasterizer interpolate these color values and generate a nice degrade per-pixel.

```csharp
struct Varyings
{
    float4 positionHCS : SV_POSITION;
    half4 color : TEXCOORD0;
```

Finally we change our vertex and fragment shader to output the color:

```csharp
Varyings vert (Attributes IN)
{
    Varyings OUT;
    OUT.positionHCS = IN.positionOS;
    OUT.color = IN.color;
    return OUT;
}

half4 frag (Varyings IN) : SV_Target
{
    return IN.color;
}
```

Now if you switch back to Unity you should see this colorful triangle. Why do we get smooth color degrade here?

I hope you are as excited as me to draw you first triangle. This is the very foundation work to get your feet wet on Computer Graphics, after all, it's all about triangles and pixels.

Or maybe you are still wondering. Well, this is nice but how is learning to draw a mesh useful for me? Turns out that we can easily extend this triangle to render a full screen quad and this is the very foundation work for post-processing. :)

How to render a quad? We render 2 triangles!

[image]

Let's change our vertex attributes to do it.

```csharp
Vector3[] vertices =
{
    // bottom-left corner
    new Vector3(-1.0f, -1.0f, 0.0f),

    // top corner
    new Vector3(-1.0f, 1.0f, 0.0f),

    // bottom right corner
    new Vector3(1.0f, -1.0f, 0.0f),

    // top right corner
    new Vector3(1.0f, 1.0f, 0.0f),
};

Color[] colors =
{
    new Color(1.0f, 0.0f, 0.0f, 1.0f),
    new Color(0.0f, 1.0f, 0.0f, 1.0f),
    new Color(0.0f, 0.0f, 1.0f, 1.0f),
    new Color(1.0f, 1.0f, 1.0f, 1.0f),
};

ushort[] indices = { 0, 1, 2, 2, 1, 3 };
```

And that's all we need to do.

## What's Next?
If you look at __Game View__ now your triangle should be nicely on screen. In __Scene View__ not so much. This is because we need to set some draw call data to position our triangle in the correct coordinate system based on it's position in the world and the position and projection of the camera.

We will learn how to render objects and change the vertex shader to position them correctly for each camera.

## What If I got lost?
You can check the final scripts and project in this github. Open the project and you will find all scripts and the Unity scene in the folder `02`.

## Exercise01:
When we created our material, we used `Shader.Find()` to search for a shader in project. However if we just try to build the scene to a standalone player our shader will not be included in the build. This is because Unity track the game dependencies by detecting all assets in the scenes and project we build. The shader is only reference in script and therefore is not automatically included. 

You can change this by adding a public field in the pipeline asset to hold the `triangleMaterial`. Because our pipeline asset is included in the build and it references the material, the material is now also included in the build. Including required resources for your rende render pipeline asset is a good practice. 

## Exercise02:
A very common technique to draw full screen geometry is to use a triangle instead of a quad. This is more efficient as it allows to avoid setting up vertex and index buffers for mesh. 
How to do it? You can use [CommandBuffer.DrawProcedural](https://docs.unity3d.com/ScriptReference/Rendering.CommandBuffer.DrawProcedural.html) and have a shader with [SV_VertexID semantic in HLSL as decribed in presentation](https://www.slideshare.net/DevCentralAMD/vertex-shader-tricks-bill-bilodeau)

If you get lost and need a reference implementation you can check mine here.







