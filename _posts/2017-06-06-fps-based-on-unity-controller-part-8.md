---
title:  "FPS based on Unity's FPS Controller Part 8 - Mob prototype, enemy FOV, player health, game over screen"
date:   2017-06-06 17:00:00 +0200
tags: FPS enemy AI mob
---
The time has come to create some basic enemies for the FPS prototype. The implementation includes basic AI behaviour, field of view and mobs being able to kill the player.
<!--more-->

<iframe width="560" height="315" src="https://www.youtube.com/embed/-vkrXLzFOkU?rel=0" frameborder="0" allowfullscreen></iframe><br>
First of all, let me mention the excellent tutorials from [Unity in Action](https://1bookcase.com/unity-action/) and [Sebastian Lague](https://www.youtube.com/channel/UCmtyQOKKmrMVaKuRXz02jbQ). Both helped me a lot in making the mob and FOV scripts work.

Mob prefabs are simple capsules with a **HealthManager** and a **BasicAi** script attached to them. **BasicAi** makes the enemies move and also casts a sphere in every update to check if an obstacle is blocking the way of the mob. If an obstacle is hit by the ray and is within the distance set in **obstacleRange** then a random angle is chosen within [-110, 110] which becomes the new orientation for the mob. This simple logic stops enemies from bumping into or going through obstacles.

Lesson learned: always check the number of arguments when using a layermask with raycast. I spent more than an hour debugging the code because the layermask didn't seem to work at all only to find out that the layermask was implicitly cast to a float and was considered as a distance value by the raycast method :D.

**BasicAi** also has a **ShootTargets()** coroutine. This iterates over all the visible targets registered by the FOV script, instantiates a bullet prefab and rotates it towards the target. My decision to move the center of the player's character controller to its feet now proved to be problematic because using **LookAt()** on all axes made the bullet fly towards the feet and colliding with the floor. At the moment, the bullet's y position remains intact so the bullet does not change its vertical angle. The coroutine yields for **attackCooldown** seconds before firing again.   

{% highlight c# %}

using System.Collections;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using UnityEngine;

public class FieldOfView : MonoBehaviour
{
    [SerializeField]
    private float viewRadius;
    [SerializeField]
    [Range(.0f, 360f)]
    private float viewAngle;
    [SerializeField]
    private LayerMask targetMask;
    [SerializeField]
    private LayerMask obstacleMask;

    private List<Transform> visibleTargets = new List<Transform>();

    public float ViewRadius
    {
        get
        {
            return viewRadius;
        }
    }

    public float ViewAngle
    {
        get
        {
            return viewAngle;
        }
    }

    private void Start()
    {
        StartCoroutine(FindTargetsWithDelay(0.2f));
    }

    private IEnumerator FindTargetsWithDelay(float delay)
    {
        while (true)
        {
            yield return new WaitForSeconds(delay);
            FindVisibleTargets();
        }
    }

    private void FindVisibleTargets()
    {
        visibleTargets.Clear();

        Collider[] targetsInViewRadius = Physics.OverlapSphere(transform.position, viewRadius, targetMask);

        foreach (Collider targetCollider in targetsInViewRadius)
        {
            Transform target = targetCollider.transform;
            Vector3 directionToTarget = (target.position - transform.position).normalized;

            if (Vector3.Angle(transform.forward, directionToTarget) < viewAngle / 2f)
            {
                float distanceToTarget = Vector3.Distance(transform.position, target.position);

                if (!Physics.Raycast(transform.position, directionToTarget, distanceToTarget, obstacleMask))
                {
                    visibleTargets.Add(target);
                }
            }
        }
    }

    public Vector3 DirectionFromAngle(float angleInDegrees, bool isAngleGlobal)
    {
        if (!isAngleGlobal)
        {
            angleInDegrees += transform.eulerAngles.y;
        }

        return new Vector3(Mathf.Sin(angleInDegrees * Mathf.Deg2Rad), .0f,
            Mathf.Cos(angleInDegrees * Mathf.Deg2Rad));
    }

    public ReadOnlyCollection<Transform> GetVisibleTargets()
    {
        return new ReadOnlyCollection<Transform>(visibleTargets);
    }
}

{% endhighlight %}

**Bullet** also has its own script. In **Update()** it moves the bullet with the given speed, if something else enters its trigger collider the bullet is destroyed (except when the other object is also a projectile). A reference is acquired to the target's **HealthManager** (if it has one) and its **Damage()** method is invoked with the bullet's damage.   

{% highlight c# %}

using UnityEngine;

public class Bullet : MonoBehaviour
{
    [SerializeField]
    private float speed = 10f;
    [SerializeField]
    private int damage = 1;

    private void Update()
    {
        transform.Translate(0, 0, speed * Time.deltaTime);
    }

    private void OnTriggerEnter(Collider other)
    {
        if (!other.GetComponent<Tags>().Projectile)
        {
            Destroy(gameObject);
        }

        HealthManager targetHealthManager = other.gameObject.GetComponent<HealthManager>();

        if (targetHealthManager != null)
        {
            targetHealthManager.Damage(damage);
        }
    }
}

{% endhighlight %}

I won't include the FOV script here, it is mostly based on Sebastian Lague's tutorial and also has an editor extension making field of view circles visible when an enemy is selected. Of course, the code can be found on my BitBucket as always.

To make enemies be able to hurt the player I made separate implementations of **NonPlayerHealthManager** and **PlayerHealthManager**. These two scripts both inherit from the abstract class **HealthManager**. Upon reaching a health amount of 0 or less, **PlayerHealthManager** disables all control scripts, unlocks the mouse cursor and calls the method **OnGameOver()** which has been implemented in a new script called **MainLogic** attached to a **GameManager** object.

This method enables a special game over GUI panel and a restart button which reloads the current scene. This coupling between the manager class and **PlayerHealthManager** is quite ugly so I'm going to add some messaging system to the code next time.
