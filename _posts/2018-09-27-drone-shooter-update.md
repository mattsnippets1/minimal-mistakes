---
title:  "Drone Shooter FPS 1.2 update"
date:   2018-09-27 18:00:00 +0200
tags: FPS AI graphics
---
It was time to update another oldie from the portfolio: my drone shooting FPS prototype.
<!--more-->

This FPS demo was a basic prototype developed from the FPS controller experiments - my first posts on this blog. It had some basic movement and shooting action along with some AI for the enemy drones, player health and ammo management - the usual stuff found in such games.

I decided to make it a little bit more cohesive on the design side altering graphics, lighting, weapon design and audio to give it a stronger sci-fi vibe.

I also wanted to experiment with some stealth action possibilities that's the main reason for the toned down lighting and the inclusion of searchlights capable of detecting a player (being detected has no consequences yet).

![Drone Shooter screenshot]({{site.url}}/assets/images/drone/drone-4-tn.jpg){: .align-center}
*Screenshot from the revamped Drone Shooter*
{: .text-center}

I won't go into technical details now but there were many lessons learned during the update process and I might post about some of the more interesting ones later on. Also, the codebase could use some refactoring and restructuring, that's the next thing I'm planning to sort out soon.   

[Portfolio page]({{site.url}}/portfolio/02-drone/)   
[Bitbucket repo](https://bitbucket.org/mattsnippets/fps_base)

## Update highlights
* Added Unity Post-processing stack and some PP effects to scene
* Changed the skybox and most of the textures
* Tweaked lighting to use realtime GI and light probes
* Added automatic searchlights which detect player within their light cones and follow the target until line of sight is broken
* Replaced weapons with more "blaster-like" models, also added volumetric laser projectiles and new weapon sounds
* Added weapon animations: recoil, weapon switch etc.
* Added shot decals
* Added background music
* Fixed some bugs
* Various optimizations