---
title:  "Experimenting with bloom"
date:   2018-07-17 20:00:00 +0200
tags: bloom postprocessing graphics corpsemob
---
I have decided to explore the possibilities of the Unity post processing stack and experiment a bit.
<!--more-->

I have been planning for some time to try Unity's post processing stack which had been made available on the Asset Store quite a long time ago. Most of the effects are better suited for 3D games or really over the top but bloom seemed like and interesting choice.

There is no actual lighting used in Corpse Mob right now so I though bloom would be ideal for some "cheating" to make the player believe that some objects are actually emitting light in the scene. If some of you are not familiar with bloom: if you enable HDR in your game the effect of strong lights can be imitated with this effect. Colors are blurred and mixed with the adjacent pixels creating a "halo" which is interpreted as light by our eyes.

Setting up the stack was easy, it is recommended to use Linear color space and enable HDR on the camera which applies the post process effects. You might also disable anti-aliasing in the quality settings because the PP stack has its own anti-aliasing effect.

So things were working out of the box but enabling bloom applied it to the whole scene. That shouldn't be surprising hence the name post processing: the effects are applied on the rendered image from the cameras. However my goal was to apply bloom to certain objects not all of them. I could have used multiple cameras and render textures to separate the bloomed and non-bloomed objects (at least some people mentioned it as a possible solution) but I decided to go with an emissive shader instead.

With a high enough threshold value (e.g. 1.1) set for bloom it is only applied on objects with high emission value. I grabbed a cool 2D shader pack which supports emission (among other effects): [traggett's UnitySpriteShaders](https://github.com/traggett/UnitySpriteShaders). Unfortunately it didn't seem to work. After trying out some different shaders came the facepalm moment: although I enabled HDR on my camera I forgot to enable HDR in the Graphics settings of the project.

![HDR check in Graphics settings]({{site.url}}/assets/images/hdrcheck.png){: .align-center}
*Don't forget to enable this!*
{: .text-center}

Now emissive materials work properly and I have created quite nice effects using them.
![Bloom screenshot 1]({{site.url}}/assets/images/bloom1.png){: .align-center}


![Bloom screenshot 2]({{site.url}}/assets/images/bloom2.png){: .align-center}


![Bloom screenshot 3]({{site.url}}/assets/images/bloom3.png){: .align-center}