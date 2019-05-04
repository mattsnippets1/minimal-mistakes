---
title:  "FPS based on Unity's FPS Controller Part 6 - Ledge climbing, damage fix, Tags class"
date:   2017-05-10 22:00:00 +0200
tags: FPS controller movement
---
My next goal was the implementation of ledge climbing to make movement around the level more fluid. I made some fixes to previous code and created a tag management class as well.
<!--more-->

<iframe width="560" height="315" src="https://www.youtube.com/embed/4au-1-S08GI?rel=0" frameborder="0" allowfullscreen></iframe><br>
In the past few years ledge grabbing and climbing is a must in most of the FPS games so I wanted to create something similar in my prototype. There are many ways to implement climbing in a first person shooter. I had decided to do it via these steps:

1. Check if conditions are valid for climbing
2. Disable character controller
3. Switch to a separate "cinematic camera"
4. Animate cinematic camera
5. Move player to the cinematic cam's new position
6. Enable character controller and enable main camera again

Climbing logic is divided between two classes: **ClimbController** attached to **FPSController** and **CinematicCamManager** attached to the new cinematic camera. By default the to other cams - main camera and gun camera - have a bigger depth than cinematic cam (which is also disabled when the player is not climbing).

**ClimbController** checks for valid climbing conditions every time the Character Controller collides with something else. To be able to climb the character controller has to be in the air, colliding with its sides and hitting an object which has a Climbable tag. The collision angle has to be within 30 degrees to the left or right from the current forward vector of the player. To disable climbing if the movement of the camera is obstructed (e.g. something above the head of the player), an overlap box - slightly translated towards the forward vector and translated above the player's head - is used to detect that nothing intersects the path of climbing and the player's head level is above the edge of climbable object. The following image shows a case of climbing obstruction - I used a gizmo to display the overlay box in red.

![Invalid climbing scenario - box above player's head]({{site.url}}/assets/images/invalid-climb.PNG){: .align-center}
*Invalid climbing scenario - box above player's head*
{: .text-center}

Here's the code of **ClimbController**:   

{% highlight c# %}

using UnityEngine;

public class ClimbController : MonoBehaviour
{   
    [SerializeField]
    private LayerMask layerMask;
    [SerializeField]
    private CinematicCamManager cinematicCamManager;
    [SerializeField]
    private float overlapBoxYTranslation;
    [SerializeField]
    private float overlapBoxForwardTranslation;

    private CharacterController characterController;
    private Transform fpsControllerTransform;

    private void Start()
    {
        characterController = GetComponent<CharacterController>();
        fpsControllerTransform = transform;
    }

    private void OnDrawGizmosSelected()
    {        
        if (fpsControllerTransform != null)
        {
            Gizmos.color = Color.red;
            Gizmos.matrix = fpsControllerTransform.localToWorldMatrix;
            Gizmos.DrawWireCube(new Vector3(.0f, overlapBoxYTranslation,
                overlapBoxForwardTranslation), new Vector3(2.0f, 2.0f, 2.0f));
        }
    }

    private void OnControllerColliderHit(ControllerColliderHit hit)
    {
        if (
            IsCollidingWithClimbable(hit) &&
            Vector3.Angle(-hit.normal, fpsControllerTransform.forward) <= 30.0f &&
            !IsClimbingObstructed()
           )
        {       
            cinematicCamManager.TriggerClimbUpAnimation();
        }
    }

    private bool IsCollidingWithClimbable(ControllerColliderHit hit)
    {
        Tags tags = hit.gameObject.GetComponent<Tags>();

        return !characterController.isGrounded &&
            characterController.collisionFlags == CollisionFlags.CollidedSides &&
            tags.Climbable;        
    }

    private bool IsClimbingObstructed()
    {
        Collider[] results = new Collider[10];

        return Physics.OverlapBoxNonAlloc(new Vector3(fpsControllerTransform.position.x,
            fpsControllerTransform.position.y + overlapBoxYTranslation, fpsControllerTransform.position.z) +
            fpsControllerTransform.forward * overlapBoxForwardTranslation,
            Vector3.one, results, fpsControllerTransform.rotation, layerMask) != 0;
    }
}

{% endhighlight %}

If the conditions are all right, **CinematicCamManager**'s **TriggerClimbUpAnimation()** method is invoked. This resets the main cam rotation, disables the gun camera (otherwise some awkward gun movements and camera angles may occur during the climb animation), disables the character controller and enables the cinematic cam. The animation called "ClimbUp" is triggered after that. This essentially tilts the cam left, right and forward while it's also moving up and forward creating the illusion of climbing up on the side of the object.

The animation has an event which invokes **MoveCharacterToCinematicPosition()**. This moves the controller to the cam's position (surprise), disables cinematic cam and enables gun cam then enables the controller itself.   

{% highlight c# %}

using System.Collections;
using UnityEngine;
using UnityStandardAssets.Characters.FirstPerson;

public class CinematicCamManager : MonoBehaviour
{
    [SerializeField]
    private CharacterController characterController;
    [SerializeField]
    private FirstPersonController firstPersonController;    
    [SerializeField]
    private Camera gunCamera;

    private Camera cinematicCamera;
    private Animator cinematicCamAnimator;

    private void Start()
    {
        cinematicCamera = GetComponent<Camera>();
        cinematicCamAnimator = GetComponent<Animator>();
    }

    public void TriggerClimbUpAnimation()
    {
        firstPersonController.ResetCameraRotation();
        gunCamera.enabled = false;
        StartCoroutine(InitClimbUpAnimation());
    }

    public void MoveCharacterToCinematicPosition()
    {        
        characterController.transform.position =
            new Vector3(transform.position.x,
            transform.position.y - characterController.height,
            transform.position.z);

        cinematicCamera.enabled = false;
        cinematicCamera.depth = 0;

        gunCamera.enabled = true;

        StartCoroutine(WaitBeforeCharacterControllerEnable());
        firstPersonController.enabled = true;
    }

    private IEnumerator WaitBeforeCharacterControllerEnable()
    {
        yield return new WaitForSeconds(0.2f);
        characterController.enabled = true;
    }

    private IEnumerator InitClimbUpAnimation()
    {
        yield return new WaitForEndOfFrame();

        characterController.enabled = false;
        firstPersonController.enabled = false;
        cinematicCamera.enabled = true;
        cinematicCamera.depth = 3;
        cinematicCamAnimator.SetTrigger("ClimbUp");
    }
}

{% endhighlight %}

Some minor changes on the code included replacing **SendMessage()** in **Weapon** with an explicit call to target **HealthManager**'s **Damage()** method (this might be more efficient) and the creation of a **Tags** class which is essentially a collection of public bools (currently only one - climbable) to enable tagging of GameObjects. I am aware that Unity has its own tagging system but it only lets you use one tag per object and I find that limiting.
