---
title:  "FPS based on Unity's FPS Controller Part 4 - Weapon inheritance, weapon switching, shot sounds"
date:   2017-04-27 20:30:00 +0200
tags: FPS controller weapon
---
Different weapons are now available through inheritance, switching between guns is also possible.
<!--more-->

<iframe width="560" height="315" src="https://www.youtube.com/embed/KzkZJARIeuQ?rel=0" frameborder="0" allowfullscreen></iframe><br>
I continued the implementation of weapon logic by creating an abstract base class called **Weapon** and three classes inheriting from it: **SemiAutomatic**, **Automatic** and **Shotgun**. I took most of the generic shooting logic found in the former **RaycastShoot** class - which is now called **WeaponManager** - and moved it to **Weapon**. **WeaponManager** checks for the fire button held down (in the case of a shotgun or automatic weapon) or the fire button pressed once (in the case of semi-automatic guns). It calls **Weapon**'s **Shoot()** method with a nullable value of time elapsed since firing (which is required for automatic rifle bullet spread).

The rest of the logic is now found in **Weapon**: check if the weapon can be fired again, handling the raycast results and bullet spread calculation. The only functionality which currently differs between child classes is the override of **CalculateShot()** abstract method. Of course, there might be more than one shotgun or pistol in game. The instances of ScriptableObject **GunStats** are used as a drag-and-drop data storage for the different weapons. Let's say we want to replace a smaller pistol with a magnum which has higher damage and longer range, we just create a new **GunStats** for magnum and drag it onto the Pistol gameobject to replace the "small pistol" **GunStats**.

I attached the three weapon subclass scripts to gameobjects which are now attached to the **FPSController**'s **GunHolder** object. I grabbed some weapon models from the asset store (Free Guns Pack) and also added sound effects for the shots which are played via each gun's own AudioSource. Here's the code for **Weapon**:

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

using UnityEngine;

public abstract class Weapon : MonoBehaviour
{
    [SerializeField]
    protected GunStats gunStats;
    [SerializeField]
    protected GameObject hitMarker;

    protected AudioSource weaponSound;
    protected Camera fpsCam;
    protected Vector3 rayOrigin;
    protected RaycastHit hit;
    protected bool readyToFire = true;
    protected float? firingSince;

    public bool IsAutomatic
    {
        get
        {
            return gunStats.isAutomatic;
        }
    }

    public float Range
    {
        get
        {
            return gunStats.range;
        }
    }

    private void Start()
    {
        Debug.Assert(gameObject.layer == LayerMask.NameToLayer("Gun"), "Gun should be on gun layer.");
        fpsCam = GetComponentInParent<Camera>();
        weaponSound = GetComponentInParent<AudioSource>();
    }

    public void Shoot(float? firingSince)
    {
        if (readyToFire)
        {
            this.firingSince = firingSince;
            readyToFire = false;
            Invoke("SetReadyToFire", gunStats.fireRate);

            rayOrigin = fpsCam.ViewportToWorldPoint(new Vector3(0.5f, 0.5f, .0f));
            CalculateShot();
            weaponSound.Play();
        }
    }

    protected abstract void CalculateShot();

    protected Quaternion CalculateBulletSpread()
    {
        Quaternion fireRotation = Quaternion.LookRotation(transform.forward);
        Quaternion randomRotation = Random.rotation;

        float currentSpread;

        if (firingSince == null)
        {
            currentSpread = gunStats.maxBulletSpreadAngle;
        }
        else
        {
            currentSpread = Mathf.Lerp(.0f, gunStats.maxBulletSpreadAngle, (float)firingSince / gunStats.timeUntilMaxSpreadAngle);
        }

        fireRotation = Quaternion.RotateTowards(fireRotation, randomRotation, currentSpread);
        return fireRotation;
    }

    protected void SetReadyToFire()
    {
        readyToFire = true;
    }

    protected void OnRayCastHit()
    {
        hit.collider.SendMessage("Damage", gunStats.damage, SendMessageOptions.DontRequireReceiver);

        if (hit.rigidbody != null)
        {
            hit.rigidbody.AddForce(-hit.normal * gunStats.hitForce);
        }

        Instantiate(hitMarker, hit.point, Quaternion.identity);
    }
}

{% endhighlight %}

And here are the subclasses:

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

using UnityEngine;

class SemiAutomatic : Weapon
{
    protected override void CalculateShot()
    {
        if (Physics.Raycast(rayOrigin, fpsCam.transform.forward, out hit, gunStats.range, gunStats.layerMask))
        {
            OnRayCastHit();
        }
    }
}

{% endhighlight %}

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

using UnityEngine;

class Automatic : Weapon
{    
    protected override void CalculateShot()
    {
        if (Physics.Raycast(rayOrigin, CalculateBulletSpread() * Vector3.forward, out hit, gunStats.range, gunStats.layerMask))
        {
            OnRayCastHit();
        }
    }
}

{% endhighlight %}

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

using UnityEngine;

class Shotgun : Weapon
{
    [SerializeField]
    private int shotsPerBullet = 10;

    protected override void CalculateShot()
    {
        firingSince = null;

        for (int i = 0; i < shotsPerBullet; i++)
        {
            if (Physics.Raycast(rayOrigin, CalculateBulletSpread() * Vector3.forward, out hit, gunStats.range, gunStats.layerMask))
            {
                OnRayCastHit();
            }
        }
    }
}

{% endhighlight %}

One more addition to **WeaponManager** is the weapon switching logic. Now the player can switch between weapons using the mouse wheel. This is acquired via activating and deactivating the relevant gameobjects. Current code of **WeaponManager**:

{% highlight c# %}

using UnityEngine;

public class WeaponManager : MonoBehaviour
{
    [SerializeField]
    private GameObject hitMarker;
    [SerializeField]
    private Weapon[] weapons;

    private float firingSince;
    private int currentWeaponIndex = 0;

    public float WeaponRange
    {
        get
        {
            return weapons[currentWeaponIndex].Range;
        }
    }

    private void Start()
    {
        weapons[currentWeaponIndex].gameObject.SetActive(true);
    }

    private void LateUpdate()
    {
        OnWeaponFire();
        OnWeaponChange();
        //Debug.Log(currentWeapon);
    }

    private void OnWeaponChange()
    {
        if (Input.GetAxis("Mouse ScrollWheel") > .0f)
        {
            weapons[currentWeaponIndex].gameObject.SetActive(false);
            currentWeaponIndex = (currentWeaponIndex + 1) % weapons.Length;
            weapons[currentWeaponIndex].gameObject.SetActive(true);
        }
        else if (Input.GetAxis("Mouse ScrollWheel") < .0f)
        {
            weapons[currentWeaponIndex].gameObject.SetActive(false);

            if (currentWeaponIndex == 0)
            {
                currentWeaponIndex = weapons.Length - 1;
            }
            else
            {
                currentWeaponIndex = (currentWeaponIndex - 1) % weapons.Length;
            }

            weapons[currentWeaponIndex].gameObject.SetActive(true);
        }
    }

    private void OnWeaponFire()
    {
        if (weapons[currentWeaponIndex].IsAutomatic)
        {
            if (Input.GetButton("Fire1"))
            {
                firingSince += Time.deltaTime;
                weapons[currentWeaponIndex].Shoot(firingSince);
            }
            else
            {
                firingSince = .0f;
            }
        }
        else
        {
            if (Input.GetButtonDown("Fire1"))
            {
                weapons[currentWeaponIndex].Shoot(null);
            }
        }
    }
}

{% endhighlight %}