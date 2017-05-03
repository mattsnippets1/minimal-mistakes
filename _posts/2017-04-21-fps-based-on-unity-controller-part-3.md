---
title:  "FPS based on Unity's FPS Controller Part 3 - Bullet spread, fire rate revamp, target damage"
date:   2017-04-21 20:30:00 +0200
tags: FPS controller weapon
---
The latest changes to the FPS controller include bullet spread over time, more efficient fire rate calculation and new code for damaging targets.
<!--more-->
To make shooting a little more interesting, I added some bullet spread logic to the current code. A new method called **CalculateBulletSpread()** is used to bring some randomness into the direction of the shots. It essentially takes the transform.forward look rotation then rotates it to a random rotation with a max degrees delta of **currentSpread**. **currentSpread** itself is calculated by lerping between 0 and **maxBulletSpreadAngle** over the elapsed time since the fire button has been pressed down divided by the **timeUntilMaxSpreadAngle** variable. The original forward vector of the shot is multiplied by the returned Quaternion. This causes the bullet spread to reach maximum value over the time window previously set (the weapon getting more and more inaccurate while firing). Here's some code:

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

private Quaternion CalculateBulletSpread()
{
    Quaternion fireRotation = Quaternion.LookRotation(transform.forward);
    Quaternion randomRotation = Random.rotation;
    float currentSpread = Mathf.Lerp(.0f, maxBulletSpreadAngle, fireTime / timeUntilMaxSpreadAngle);
    fireRotation = Quaternion.RotateTowards(fireRotation, randomRotation, Random.Range(.0f, currentSpread));

    return fireRotation;
}

{% endhighlight %}

The previous fire rate calculation from the original Unity code used Time.time to check if the weapon can be fired again. I don't like the idea of a value constantly growing while the game is running so I chose another way to solve this problem. A bool named **readyToFire** is used now to indicate whether the gun can be fired once again. Every time the weapon is fired it is set to false and the **SetReadyToFire()** method is invoked with a latency of **fireRate**. This method then re-enables **readyToFire** and the gun van be fired in the next update.

I somewhat changed the layout of the shooting code by refactoring and reorganizing larger blocks into separate methods. Here's the current version:

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

public class RaycastShoot : MonoBehaviour
{
    [SerializeField]
    private int gunDamage = 1;
    [SerializeField]
    private float fireRate = .25f;
    [SerializeField]
    private float weaponRange = 50f;
    [SerializeField]
    private float hitForce = 100f;
    [SerializeField]
    private float maxBulletSpreadAngle = 15.0f;
    [SerializeField]
    float timeUntilMaxSpreadAngle = 1.0f;
    [SerializeField]
    private Transform gunEnd;

    [SerializeField]
    private GameObject hitMarker;

    private Camera fpsCam;
    private WaitForSeconds shotDuration = new WaitForSeconds(.05f);
    private AudioSource gunAudio;
    private LineRenderer laserLine;
    private int layerMask;
    private float fireTime;
    private bool readyToFire = true;

    public float WeaponRange
    {
        get
        {
            return weaponRange;
        }
    }

    void Start()
    {
        laserLine = GetComponent<LineRenderer>();
        gunAudio = GetComponent<AudioSource>();
        fpsCam = GetComponentInParent<Camera>();
        layerMask = ~LayerMask.GetMask("Gun", "Player");
    }

    void FixedUpdate()
    {
        if (Input.GetButton("Fire1"))
        {
            fireTime += Time.deltaTime;

            if (readyToFire)
            {
                readyToFire = false;
                Invoke("SetReadyToFire", fireRate);
                StartCoroutine(ShotEffect());
                CastRay();
            }
        }
        else
        {
            fireTime = .0f;
        }
    }

    private void CastRay()
    {
        laserLine.SetPosition(0, gunEnd.position);
        Vector3 rayOrigin = fpsCam.ViewportToWorldPoint(new Vector3(0.5f, 0.5f, .0f));
        RaycastHit hit;

        if (Physics.Raycast(rayOrigin, CalculateBulletSpread() * Vector3.forward, out hit, weaponRange, layerMask))
        {
            laserLine.SetPosition(1, hit.point);

            hit.collider.SendMessage("Damage", 1, SendMessageOptions.DontRequireReceiver);

            if (hit.rigidbody != null)
            {
                hit.rigidbody.AddForce(-hit.normal * hitForce);
            }

            Instantiate(hitMarker, hit.point, Quaternion.identity);
        }
        else
        {
            laserLine.SetPosition(1, rayOrigin + (fpsCam.transform.forward * weaponRange));
        }
    }

    private Quaternion CalculateBulletSpread()
    {
        Quaternion fireRotation = Quaternion.LookRotation(transform.forward);
        Quaternion randomRotation = Random.rotation;
        float currentSpread = Mathf.Lerp(.0f, maxBulletSpreadAngle, fireTime / timeUntilMaxSpreadAngle);
        fireRotation = Quaternion.RotateTowards(fireRotation, randomRotation, Random.Range(.0f, currentSpread));

        return fireRotation;
    }

    private IEnumerator ShotEffect()
    {
        gunAudio.Play();

        laserLine.enabled = true;
        yield return shotDuration;
        laserLine.enabled = false;
    }

    private void SetReadyToFire()
    {
        readyToFire = true;
    }
}

{% endhighlight %}

As you can see, the way of damaging a target when the raycast hits has also changed. Manipulating enemy health from the shooting script was far from elegant so I created a new **HealthManager** script. This can be attached to any gameobject making it damageable or healable. The shooting code now uses **SendMessage()** to invoke **Damage()** on the target (if it has a **HealthManager** attached).

{% highlight c# %}

public class HealthManager : MonoBehaviour
{
    [SerializeField]
    private int health;

    public void Damage(int damage)
    {
        health -= damage;

        if (health <= 0)
        {
            gameObject.SetActive(false);
        }
    }

    public void Heal(int heal)
    {
        health += heal;
    }
}

{% endhighlight %}