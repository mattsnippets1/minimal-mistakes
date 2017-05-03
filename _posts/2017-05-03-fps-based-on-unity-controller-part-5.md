---
title:  "FPS based on Unity's FPS Controller Part 5 - Test level, crouching fix, controller/gamepad support"
date:   2017-05-03 11:00:00 +0200
tags: FPS controller movement
---
I made a test level for further movement tweaking, fixed a crouching bug and added code for Xbox controller support.
<!--more-->
Until now, the simple test level from the original Unity tutorial was enough for my needs but now I felt I need some more possibilities for tweaking the character controller. I downloaded [ProBuilder Basic](https://www.assetstore.unity3d.com/en/#!/content/11919) from the asset store and put together a sandbox level with stairs, boxes, doors etc. of various sizes. I can only recommend ProBuilder as it made creating the scene much easier for me.

![Test map scene]({{site.url}}/assets/images/test-level-probuilder.PNG){: .align-center}
*Test map scene*
{: .text-center}

While moving around and trying jumping, crouching, slope angles and stuff I noticed a serious issue: I forgot to make my crouch compatible with standing on top of objects so the character would crouch into a box or stairs if it was standing on top of them. To fix this issue, I had to modify the calculation of the camera's standing and crouching positions in **CrouchController**. Now the character controller's current y position is added to the required camera height like this:

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

using UnityEngine;

Vector3 standCamPosition = new Vector3(camPosition.x, characterController.transform.position.y +
    camStandingHeight, camPosition.z);
Vector3 crouchCamPosition = new Vector3(camPosition.x, characterController.transform.position.y +
    camCrouchingHeight, camPosition.z);

{% endhighlight %}

The spherecast used to prevent standing up under obstacles also caused problems. The sphere collided with the gameobject under the player's feet so he could not stand up after crouching on top of something. I fixed this by casting the sphere from the center of the crouching controller and casting it with the limit of a maximum distance that allows standing up under objects which have a slightly higher y position than the player's height.

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

using UnityEngine;

if (isCrouching)
{
    return Physics.SphereCastNonAlloc(new Vector3(characterController.transform.position.x,
        characterController.transform.position.y + characterController.radius, characterController.transform.position.z),
        characterController.radius, Vector3.up, new RaycastHit[2], standingHeight - 2 * characterController.radius) == 1;
}

{% endhighlight %}

Another undertaking was fixing controller support. I have only tested the code with an Xbox 360 controller on Windows so it might not be functional on other OS/with another controller (I'm sure that the solution is similar in those cases, though). In the **FirstPersonController** class code the script for running still contained left shift as the only running button. I had to change that to **GetButton("Run")**. This way all the input buttons defined as "Run" would trigger running. I set the left shoulder button as the controller's running input. Jump and crouch could be set up in a similar fashion so the character now jumps with the Y and crouches with the B controller buttons.   

I also changed the mouse look logic, adding some extra code to **MouseLook** class. The game now checks for a connected joystick every time **LookRotation()** method is invoked and if it finds one then the controller axes are used instead of the mouse to rotate the camera. Here's the current code, **xAxis** and **yAxis** are declared along with the other class fields at the top of the code.

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

using UnityEngine;

public void LookRotation(Transform character, Transform camera)
{
    string[] jNames = Input.GetJoystickNames();

    if (jNames.Length > 0 && jNames[0] != "")
    {
        xAxis = "Joystick Look X";
        yAxis = "Joystick Look Y";
    }
    else
    {
        xAxis = "Mouse X";
        yAxis = "Mouse Y";
    }

    float yRot = CrossPlatformInputManager.GetAxis(xAxis) * XSensitivity;
    float xRot = CrossPlatformInputManager.GetAxis(yAxis) * YSensitivity;

    m_CharacterTargetRot *= Quaternion.Euler(0f, yRot, 0f);
    m_CameraTargetRot *= Quaternion.Euler(-xRot, 0f, 0f);

    if (clampVerticalRotation)
        m_CameraTargetRot = ClampRotationAroundXAxis(m_CameraTargetRot);

    if (smooth)
    {
        character.localRotation = Quaternion.Slerp(character.localRotation, m_CharacterTargetRot,
            smoothTime * Time.deltaTime);
        camera.localRotation = Quaternion.Slerp(camera.localRotation, m_CameraTargetRot,
            smoothTime * Time.deltaTime);
    }
    else
    {
        character.localRotation = m_CharacterTargetRot;
        camera.localRotation = m_CameraTargetRot;
    }

    //UpdateCursorLock();
}

{% endhighlight %}

I wanted to use the right trigger of the controller as the primary fire button and the D-Pad up and down buttons for weapon switching. These are however not handled as buttons, they are joystick axes. The problem with this that there is no "GetButtonDown()" method defined for axes meaning it triggers the bound action constantly while being pushed. That was problematic with the semi-auto weapons and weapon switching so I decided to write some code implementing **GetButtonDown()**-like behaviour for axes. The goal here is to only trigger actions when the state of the axis has changed compared to the previous update.

I created a static class: **JoystickAxisDataManager** which has a private dictionary containing all the required axis names and a float value stored with them. To use this class the static method **AxisGetButtonDown()** has to be called with the argument of the axis name. If the dictionary has no key corresponding to the axis name, then the class queries the current value for the axis, stores it in the dictionary then returns it to the caller.

From now on, each time the get button down is invoked again it compares the current value to the value stored in the dictionary. If it's different it returns the new value and stores it in the dictionary. If it's the stame, that means "no change" so it simply returns 0. It is important to only call **AxisGetButtonDown()** once per update so if the return value is required in multiple parts of the code it is best to store it in a variable and work with that in the update.

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

using UnityEngine;

using System.Collections.Generic;
using UnityEngine;

public static class JoystickAxisDataManager
{
    private static Dictionary<string, float> axisValues = new Dictionary<string, float>();

    public static float AxisGetButtonDown(string axisName)
    {
        float currentAxisValue = Input.GetAxisRaw(axisName);

        if (axisValues.ContainsKey(axisName))
        {
            if (axisValues[axisName] != currentAxisValue)
            {
                axisValues[axisName] = currentAxisValue;
                return currentAxisValue;
            }
            else
            {
                return .0f;
            }
        }
        else
        {
            axisValues[axisName] = currentAxisValue;
            return currentAxisValue;
        }
    }
}

{% endhighlight %}

The controller code is quite messy in its current state and it's divided between many classes so I might clean that up in the future.