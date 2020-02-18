---
layout: post
title:  "Enter SRP"
date:   2020-02-16 19:43:31 +0100
categories: 
---
# Enter SRP

When I joined Unity a little bit more than 3 years ago there was a huge excitement in Unity about new rendering pipelines. SRP (Scriptable Rendering Pipeline) became the buzz word in the graphic foundation team. The ability to describe rendering code in C# land was exciting to me. From one perspective this would allow developers to customize Unity in a level not possible before, on the other hand it would allow students to learn Computer Graphics techniques at a high level. The past three years I've been developing the Universal Render Pipeline on top of the SRP library. Now it's time to start teaching Computer Graphics using SRP.

So what is this SRP and why is that important if you are a student or professional in the area?
In short, SRP allows to describe at a high level rendering code in Unity. Just like couple of decades ago GPUs allowed programmable pipeline stages, with SRP, Unity is allowing developers to program large parts of rendering in C#. A render pipeline implements a strategy that culls objects and lights and then perform a series of render steps (compute or render passes) that composite a final frame. In [this article](http://www.adriancourreges.com/blog/2016/09/09/doom-2016-graphics-study/) you can see an example of a high level breakdown of the render pipeline of DOOM (2016).

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

## 
{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
