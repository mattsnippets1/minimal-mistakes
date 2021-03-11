---
title: "Balance"
date:   2021-03-09 10:00:00 +0200
excerpt: "3rd person shooter game set in a dark, procedurally generated forest where you fight huge spiders."
header:
  teaser: /assets/images/balance/balance-teaser.jpg
gallery:
  - url: /assets/images/balance/balance-1.jpg
    image_path: assets/images/balance/balance-1-tn.jpg
    alt: "balance screenshot 1"
  - url: /assets/images/balance/balance-2.jpg
    image_path: assets/images/balance/balance-2-tn.jpg
    alt: "balance screenshot 2"
  - url: /assets/images/balance/balance-3.jpg
    image_path: assets/images/balance/balance-3-tn.jpg
    alt: "balance screenshot 3"
  - url: /assets/images/balance/balance-4.jpg
    image_path: assets/images/balance/balance-4-tn.jpg
    alt: "balance screenshot 4"
  - url: /assets/images/balance/balance-5.jpg
    image_path: assets/images/balance/balance-5-tn.jpg
    alt: "balance screenshot 5"
  - url: /assets/images/balance/balance-6.jpg
    image_path: assets/images/balance/balance-6-tn.jpg
    alt: "balance screenshot 6"
---

An early alpha demo of Balance, a procedurally generated TPS game where you roam around a dark forest world controlling a golem, fighting huge, procedurally animated spiders and collect powerups. (The current build has no sound or music).

*Engine: Unity -- Platform: Windows*

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/cydZWDkCZdM?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

### Feature highlight:
- 3rd person character with keyboard + mouse controls and camera follow
- Procedurally generated 3D terrain chunks based on seed input
- Pooled and throttled object creation to keep a consistent framerate
- Dynamic loading and unloading of visible chunks reducing memory and CPU usage
- Inventory system with a hotbar, stackable items, option to drop the items onto the ground
- Spider enemies with inverse kinematics procedural animation and ragdolling death
- Patrol and follow AI for enemies
- Zoomable automap generated from the chunk data
- Save-load system serializing world, player and inventory state into a file
- Firing with auto-aim

The assets are either freely available or have been purchased from the asset store.

{% include gallery %}

## Balance links
[Balance Windows build]({{ site.url }}/assets/download/balance_demo_desktop_0.2.4.zip)
## Repository
[https://gitlab.com/mattsnippets/balance](https://gitlab.com/mattsnippets/balance)  
