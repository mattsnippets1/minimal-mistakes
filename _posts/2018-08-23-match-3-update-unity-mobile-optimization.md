---
title:  "Match 3 v1.2 update, Unity mobile optimization tips"
date:   2018-08-23 10:00:00 +0200
tags: match3 mobile optimization
---
My Match 3 game prototype (which has open-sourced code, see link below) went through a bunch of update iterations. While making it harder, better, faster, stronger I collected a list of mobile optimizations to consider (along with a bunch of links).
<!--more-->

I thought it was time to update my little portfolio games and I started with the simple [match 3 game]({{site.url}}/portfolio/2017-08-04-match3) I made a year ago. There was (is?) much to be done. As a first step I switched the project to Unity version 2018.2 which was quite painless, luckily. The following updates brought many changes in terms of visuals and optimization as well. For a comprehensive list see the [bitbucket repo](https://bitbucket.org/mattsnippets/match_3_base).

![Match 3 screenshot]({{site.url}}/assets/images/match3/match3-screenshot1-tn.jpg){: .align-center}
*Brand new Match 3 design*
{: .text-center}

The most useful part of the update was collecting lots of optimization best practices and tricks for Unity mobile. I put a "checklist" here for future reference (for myself and maybe others). Of course these practices shouldn't be applied mindlessly to every Unity mobile project but all of them might be considered as a possible optimization step depending on the nature of the project. Test and profile, results may vary. At the end of the post I also linked most of the sources of these tips, they go into more detail regarding the topic.

## Mobile optimization checklist
* Profile the game for possible bottlenecks (use real device because profiling in the editor doesn't give accurate results)
* Pool objects reused frequently because Instantiate-Destroy is expensive and leads to GC spikes
* Find out if the game is GPU or CPU bound (see Unity documentation in the links below)
* Avoid dynamic per-pixel lighting
* Avoid lots of real time shadows
* Prefer baked lights if possible
* Use lightprobes to fake dynamic lights on objects
* Mark GameObjects as static if possible
* Use Unity's mobile shaders or other optimized shaders whenever you can
* Avoid lots of transparency, alpha-tests
* Avoid high vertex count
* Avoid lots of overdraw
* Remember that by default Unity runs the mobile build on the device's native resolution, that might be too much for the hardware, resolution can be lowered from scripts
* Use sprite atlases and atlas variants selected by script based on device resolution
* Use reasonable sprite and texture sizes (preferably power of two for compression), use texture compression
* Consider using asset bundles
* Consider using IL2CPP. It has restrictions and longer build time but it's more performant most of the time
* For mobile, choose a lower setting in the Quality menu, texture size can be smaller e.g. Half Res, you may also get rid of anisotropic textures and soft particles, some AA should be fine
* Disable logging for the build when not debugging because calling the logger even without output has a lot of overhead
* Use vsync, preferably locked at 60 FPS. Most mobile devices are locked by default but if vsync is not set explicitly the game might still run faster, hogging a lot of CPU power on the device
* Audio can probably be converted to mono
* Avoid complex physical simulation
* Use the least amount of Update() calls. If something has to be called regularly but not 60 times a second then a coroutine is a better choice
* Also, if you have to update lots of GameObjects, rolling out your own update method for them is more performant than using built-in callbacks
* Don't leave empty Unity callbacks in GameObjects, they still get called and slow down the game
* Enable static and dynamic batching, use Frame Debugger to see the number of draw calls in the game. For mobile it is ideal to keep them under 20 per frame
* Avoid using GameObject.Find() and derivatives, especially in Update()
* Cache references (e.g. component references through GetComponent()) preferably in the Awake() callback of the GameObject, don't ask for the same reference multiple times
* Watch out for Unity GUI performance, setting a canvas to pixel perfect, using multiple canvases and having raycasters on lots of elements might lead to performance drops
* Use LOD if applicable

## Unity Mobile optimization articles
[Unity docs mobile optimization guide](https://docs.unity3d.com/Manual/MobileOptimizationPracticalGuide.html)   
[Unity docs graphics performance guide](https://docs.unity3d.com/Manual/OptimizingGraphicsPerformance.html)   
[Unity profiler, performance optimization tutorial](https://unity3d.com/learn/tutorials/topics/performance-optimization)   
[Unity forum post with links about mobile optimization](https://forum.unity.com/threads/3d-mobile-optimization.482292/)   
[Performance tips for Unity 2d mobile by Damian Connolly](https://divillysausages.com/2016/01/21/performance-tips-for-unity-2d-mobile/)   
[12 tricks for optimizing performance in VR apps in Unity 5 by Darshan Shankar](https://blog.bigscreenvr.com/12-performance-tricks-for-optimizing-vr-apps-in-unity-5-9849bb6aefa7)   
[Unity: Android Optimization Guide by Niels Tiercelin](https://www.gamasutra.com/blogs/NielsTiercelin/20170824/304424/Unity_Android_Optimization_Guide.php)