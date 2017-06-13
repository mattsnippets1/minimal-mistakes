---
title:  "FPS based on Unity's FPS Controller Part 9 - Messaging system, textures, code comments"
date:   2017-06-13 18:00:00 +0200
tags: FPS messaging decoupling texturing
---
By introducing a messaging system into the code I have eliminated some of the ugly coupling problems and component references. I also created a scene with some rudimentary textures and added more comments to code.
<!--more-->

I could have implemented my own messaging system or used C# events but I just went with using [CSharpMessenger Extended](http://wiki.unity3d.com/index.php?title=CSharpMessenger_Extended) from Unify Community. It can send generic messages with the possibility of multiple parameters and returns also. The centralized messaging helped me in removing the references in game over, climbing, ladder climbing logic and in the **CinematicCamManager** class. It might generate some overhead but the code has become more organized and manageable.

Comments have been added to the more obscure parts of the code to clarify the functionality (and as a reminder for myself when I return to work on these scripts after some time). The uncommented code is mostly self-explanatory IMO.

After all of this scripting it was time for some eye candy so I downloaded some materials from the Asset Store to use in the test level (I created a textured copy of the scene, to be exact). It's still some really basic texturing - the models are really simple, have no special UVs just the same texture tiled over their surfaces - but it's something.

![Textured scene screenshot]({{site.url}}/assets/images/textured-test-scene.jpg){: .align-center}
*Textured scene screenshot*
{: .text-center}

I have added a particle effect to the scene to replace the red hit marker orbs used previously at the hit locations. They have a limited lifetime so the game does not get flooded with tons of gameobjects while shooting.