---
title:  "FPS based on Unity's FPS Controller Part 1 - Crouching"
date:   2017-04-15 14:00:00 +0200
tags: FPS movement controller
---
I decided to create a first person shooter prototype, a "base" which I can use in further experiments.
<!--more-->
I took Unity's **Shooting with Raycasts** tutoral project as a basis for my sandbox. I could have implemented an FPS controller from scratch but this time I have deiced to go with Unity's out of the box version. I may change my mind later on.   
After playing around with the controls a bit I did a quick fix: the player's weapon was clipping through walls and this was easily solved by creating a Gun layer then adding the Gun object and its children to it. I created another camera and added it as a child of FirstPersonCharacter. This cam has Clear Flags set to Depth only, Culling Mask set to Gun and Depth set to a bigger value than the original main cam's. This way the weapon still clips into the wall but it seems like it doesn't. I disabled a the FOV Kick on the FPS Controller, because it only h andles one camera so there was an odd glitch when the two rendered weapons were misaligned.

![Second cam settings](/assets/images/second-cam-settings.PNG){: .align-center}
*Second cam settings*
{: .text-center}

Soon I found the next shortcoming: no crouching. So I decided to implement a crouch mechanism for the controller. I wanted to keep this logic as separate from the character controller code as possible, so I created a Crouch Controller Monobehaviour script. The crouch is now implemented as a "toggle", the player does not hold the crouch key, crouch mode can be turned on and off instead. The idea was to move the main camera attached to the FirstPersonCharacter object between two positions on the y axis. This gives the illusion of crouching, it doesn't fix collision problems however (the player doesn't become "smaller" as expected).

To make the crouching player smaller I also set the height of the Character Controller in the script. Originilly, the Character Controller had its Center in the middle of its capsule. I changed this so the center coordinates are 0, 1, 0, effectively moving the center to the "feet" of the character. Upon resizing, the center y coordinate is switched between 0.5 and 1 eliminating strange behaviour such as the triggering of the landing sound because the engine assumes that the player has landed on the ground from a higher position.

![Crouch controller in action](/assets/images/crouch-controller.gif){: .align-center}
*Crouch controller in action*
{: .text-center}
   
It was also necessary to introduce a flag - isCrouchTransitionInProgress - so the lerp isn't applied constantly to cam position. This would mess up jumping and other movement with height changes. For now, this version is working but there is still much left to do e.g. disable crouch while jumping. Here's the code of the current CrouchController class:   

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

public class CrouchController : MonoBehaviour
{
    [SerializeField]
    private float speed;
    [SerializeField]
    private float crouchingHeight;
    [SerializeField]
    private float standingHeight;
    [SerializeField]
    private float camStandingHeight;
    [SerializeField]
    private float camCrouchingHeight;
    [SerializeField]
    private bool isCrouchTransitionInProgress = false;
    [SerializeField]
    private CharacterController characterController;

    private bool isCrouching = false;

    private void FixedUpdate()
    {
        Vector3 camPosition = transform.position;
        Vector3 standCamPosition = new Vector3(camPosition.x, camStandingHeight, camPosition.z);
        Vector3 crouchCamPosition = new Vector3(camPosition.x, camCrouchingHeight, camPosition.z);

        if (Input.GetButtonDown("Crouch"))
        {
            isCrouchTransitionInProgress = true;

            if (isCrouching)
            {
                isCrouching = false;
                characterController.height = standingHeight;
                characterController.center = new Vector3(0, 1.0f, 0);
            }
            else
            {
                isCrouching = true;
                characterController.height = crouchingHeight;
                characterController.center = new Vector3(0, 0.5f, 0);
            }
        }

        if (isCrouchTransitionInProgress)
        {
            if (isCrouching)
            {
                CamLerpToPosition(camPosition, crouchCamPosition);
            }
            else
            {
                CamLerpToPosition(camPosition, standCamPosition);
            }
        }
    }

    private void CamLerpToPosition(Vector3 currentPosition, Vector3 targetPosition)
    {
        transform.position = Vector3.Lerp(currentPosition, targetPosition, Time.fixedDeltaTime * speed);

        if (Mathf.Abs(transform.position.y - targetPosition.y) < 0.01f)
        {
            isCrouchTransitionInProgress = false;
            Debug.Log("Reached " + (isCrouching ? "crouching" : "standing") + " height");
        }
    }
}

{% endhighlight %}
