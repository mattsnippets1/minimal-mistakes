---
title:  "FPS based on Unity's FPS Controller Part 2 - Crouching continued + shooting tweaks"
date:   2017-04-16 20:30:00 +0200
tags: FPS movement controller weapon
---
I continued the implementation of the FPS controller with some fixes to crouch mechanics.
<!--more-->
One of the biggest problems was the possibility of crouching into obstacles while standing directly under them. Luckily, this could be fixed by applying a SphereCast on the player. The origo of the spherecast is the Character Controller's position, the direction is Vector3.up, length is a little more than the Character Controller's height and radius is equal to the controller's radius. Spherecast is necessary instead of raycast because raycast has no "thickness", it shoots from the center of the Character Controller and causes problems if the player is only slightly under the edge of an obstacle.

To disallow crouching while jumping, I introduced another check in the CrouchController code. If the Character Controller's **isGrounded** property is false (meaning the character is in the air), jumping is not allowed. To fix the glitches originating from jumping while the crouch lerp is in progress (this led to the player jumping through the floor and other anomalies) the **isGrounded** property is once more checked in CrouchController's **FixedUpdate()** before invoking the lerp function. If the character is not grounded, then lerping is essentially ignored in that frame update.

I also tweaked the shooting mechanics a bit. In the previous implementation the player could occasionally hit himself by accident (e.g. while backpedaling). This was fixed with the use of a LayerMask. It could have been done by exposing a LayerMask variable in the editor and checking the required layers but I decided to go with the old-fashioned way and created the binary negation of the bitmask containing the **Player** and **Gun** layers (both layers are user defined):

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

int layerMask = ~LayerMask.GetMask("Gun", "Player");

{% endhighlight %}

I also added a hit marker object in the form of a small static red sphere which is instantiated at the point of raycast hit. This way debugging hit locations is much easier:

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

Instantiate(hitMarker, hit.point, Quaternion.identity);   

{% endhighlight %}   


![Hit marker spheres after some shots](/assets/images/red-hit-markers.PNG){: .align-center}
*Hit marker spheres after some shots*
{: .text-center}

One more little modification was replacing **GetButtonDown()** with **GetButton()**. This way the fire script is invoked constantly while the fire button is held down, imitating an automatic weapon.   
Next time I'll continue with some bullet spreading code.