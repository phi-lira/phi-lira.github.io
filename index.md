---
layout: default
title: Introduction
nav_order: 1
description: "LearnSRP is page that focuses on teaching Computer Graphics using Unity and Scriptable Render Pipelines (SRP)."
permalink: /
---

# Scriptable Render Pipeline (SRP)
{: .fs-9 }

SRP is an [open source](https://github.com/Unity-Technologies/ScriptableRenderPipeline) technology made by [Unity](https://unity.com/). Similar to how GPUs allow programmable pipeline stages, with SRP, Unity allows developers to program large parts of rendering in C#.
{: .fs-6 .fw-300 }

This page focuses on teaching Computer Graphics using Unity and SRP. All lessons and material can be downloaded in the LearnSRP github page.

[Get started now](#getting-started){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 } [LearnSRP GitHub](https://github.com/phi-lira/learnsrp){: .btn .fs-5 .mb-4 .mb-md-0 }

## Why learning about SRP

If you are a student, you can use SRP to learn about Computer Graphics rendering techniques such as forward, deferred and tile/clustered lighting.

If you are a professional you can create highly customizable production pipeline for your game or application. 


## About the author

Felipe Lira is a Graphics Programmer at Unity Technologies. He joined Unity in 2016 to work on SRP technology and he now leads the development of the [Universal Render Pipeline](https://unity.com/srp/universal-render-pipeline), a robust and optimized renderer that supports several platforms. 

---

## Getting started

## 1. Pre-requisites

In order to follow these lesssons it's assumed you have basic understanding of [C# programming language](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/) and the [Unity Engine](https://unity.com/).

## 2. Setting up your development environment

### 2.1 Create a project with Unity 2019.3.0f6 or above 

Follow [these instructions](https://docs.unity3d.com/Manual/GettingStarted.html) to install Unity and create a new 3D project template.

### 2.2 Change your project's color space to linear

Click on __Edit -> Project Settings__. Then now under __Player__ panel, select __Linear__ color space. This will give us correct lighting curve for rendering.

![Linear Color space](assets/images//linearcolorspace.JPG)

### 2.3 Install Core RP Library package 7.2.0 or above

This contains the SRP framework that will allow us to override Unity rendering.

Open the newly created Unity project. On the top menu bar, click on __Window -> Package Manager__.
A new window will open. You can now search for Core RP. Select the __7.2.0__ or above version and click on __install__. 

![Package Manager](assets/images//packagemanager.JPG)
---
**NOTE:** 
If you don't see package 7.2.0. Click on __show all versions__ under the Core RP package tab.

