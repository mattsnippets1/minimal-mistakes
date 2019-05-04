---
title:  "FPS based on Unity's FPS Controller Part 7 - Ladders, mobile controls, crouch fix"
date:   2017-05-21 19:00:00 +0200
tags: FPS controller movement mobile
---
Another essential feature for FPS controls is the possibility  of vertical movement - climbing ladders for example. This was my next milestone along with adding mobile input to the game.
<!--more-->

<iframe width="560" height="315" src="https://www.youtube.com/embed/LrAbcGYrn2I?rel=0" frameborder="0" allowfullscreen></iframe><br>
The concept for ladder climbing was to disable regular movement controls while the player is "attached" to a ladder and only allow movement on the y axis. Sticking to a ladder is achieved via walking into it while facing the ladder surface or jumping onto the ladder. Climbing down a ladder is also possible by walking backwards into the its collider.

The class **LadderClimbController** has most of the logic for vertical climbing. In **Update()** it checks whether the player is leaving the ladder by jumping or climbing off or - on the contrary - climbing is currently off but the player collides with a ladder, he's facing the right direction and is moving on the vertical axis.

While ladder climbing is enabled, **FirstPersonController**'s movement is turned off and the character only moves on the y axis, the movement vector multiplied by a climbing speed value. Here's the code of **LadderClimbController**:   

{% highlight c# %}

using UnityEngine;
using UnityStandardAssets.Characters.FirstPerson;
using UnityStandardAssets.CrossPlatformInput;

public class LadderClimbController : MonoBehaviour
{
    [SerializeField]
    private float climbingSpeed;

    private bool isClimbingLadder = false;
    private bool isCollidingWithLadder = false;
    private CharacterController characterController;
    private FirstPersonController firstPersonController;
    private Transform ladderTransform;

    private void Start()
    {
        characterController = GetComponent<CharacterController>();
        firstPersonController = GetComponent<FirstPersonController>();
    }

    private void Update()
    {
        if (isClimbingLadder)
        {
            if (!isCollidingWithLadder || CrossPlatformInputManager.GetButtonDown("Jump"))
            {
                ToggleLadderClimbing(false);
            }
        }
        else if (isCollidingWithLadder)
        {
            if (Vector3.Dot(ladderTransform.forward, transform.forward) >= 0.9f &&
                (CrossPlatformInputManager.GetAxis("Vertical") > .0f || !characterController.isGrounded))
            {
                ToggleLadderClimbing(true);
            }
        }
    }

    private void FixedUpdate()
    {
        if (isClimbingLadder)
        {
            transform.Translate(Vector3.up * CrossPlatformInputManager.GetAxis("Vertical") *
                climbingSpeed * Time.deltaTime);
        }
    }

    private void ToggleLadderClimbing(bool isEnabled)
    {
        isClimbingLadder = isEnabled;
        firstPersonController.ToggleMovement(!isEnabled);
    }

    private void OnTriggerEnter(Collider other)
    {
        if (other.GetComponent<Tags>().Ladder)
        {
            isCollidingWithLadder = true;
            ladderTransform = other.transform;
        }
    }

    private void OnTriggerExit(Collider other)
    {
        if (other.GetComponent<Tags>().Ladder)
        {
            isCollidingWithLadder = false;
        }
    }
}

{% endhighlight %}

**FirstPersonController** also got a new private boolean for tracking whether movement is currently disabled. The value of this switch is set by the public method **ToggleMovement()**.

I'm not convinced that mobile devices are suitable for playing FPS games but I was curious if Unity's **CrossPlatformInput** classes are of any use. For a mobile input prototype I went with a movement joystick on the left and a touchpad area controlling the cam on the right. Firing is triggered via a separate fire button instead of touching the screen anywhere (which registers as a "left mouse button click" by default), I may implement double tapping as the fire command in the future. There are additional buttons for run toggle, jump, crouch toggle and weapon switching. Here's a screenshot of the interface - nothing fancy, but it's working at least.

![Mobile controls screenshot]({{site.url}}/assets/images/mobile-controls-screenshot.png){: .align-center}
*Mobile controls screenshot*
{: .text-center}

Most of the functionality could be achieved with **CrossPlatformInput**'s code. However, I found the touchpad nearly unusable so I used another touchpad implementation from [CN Controls](https://www.assetstore.unity3d.com/en/#!/content/15233) adding the possibility to change the sensitivity of the control. In most parts of the code I had to replace **Input** with **CrossPlatformInputManager**. I also added a run toggle binding so running can be turned on and off with a switch witch is much more comfortable on mobile. Unfortunately there are some framerate drops on mobile, this might be related to shader settings but I have to look into it some more.

Also, there was a stupid bug in **CrouchController**: I put the input logic into **FixedUpdate()** instead of **Update()**. Because **FixedUpdate()** is not invoked in every frame sometimes the keypresses weren't registered correctly.
