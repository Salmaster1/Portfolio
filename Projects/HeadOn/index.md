[<< Back](https://salmaster1.github.io/Portfolio/)

# Head On

Head On was developed during my time at the school Yrgo in gothenburg. The project was a part of our *Game Design* course, where we used our knowledge and lessions of game design to create a game. This project was not my idea, but one of my group members.

During this project, my role was as a programmer, and the game was developed in the Unity Game Engine, with the programming language C#.

During the development process, the programming that I mainly focused on was the player character, the signal-button logic, and the system for grabable items.

## My Work

### Grabbing & Throwing

The player character in **Head On** is able to pick up and grab various items throughout the game. The following is an excerpt from the *PlayerGrabbing* class:

<details><summary>PlayerGrabbing.cs (excerpt)</summary>
  <pre><code class="language-csharp">

private void TryGrabObject(bool requireClick)
{
    Grabable g = GetClosestGrabable();
    if (g == null) return;
    if (requireClick != g.RequireClick) return;
    if (g != null && g.GetType() == typeof(Throwable) && grabDelayTimer <= 0)
    {
        if (transform.parent.parent = g.transform) //Unchild player if grabbing box that player is standing on
            transform.parent.parent = null;
        SetHeldItem((Throwable)g);
    }
    else if (g != null)
    {
        //Ledge grab
        if (activeInputType == InputType.Controller && grabDelayTimer <= 0)
        {
            grabDelayTimer = 0.2f;
            if (currentGrabable != null)
            {
                currentGrabable.ToggleGrabableVisual(false);
                currentGrabable = null;
            }
            g.HoldItem();
        }
        else if (activeInputType == InputType.KeyboardMouse && grabDelayTimer <= 0)
        {
            grabDelayTimer = 0.2f;
            if (currentGrabable != null)
            {
                currentGrabable.ToggleGrabableVisual(false);
                currentGrabable = null;
            }
            g.HoldItem();
        }
    }
    //Grab lever
    if (currentThrowable == null)
    {
        EnableArms();
    }
    else
    {
        foreach (var item in arms)
        {
            item.Renderer.enabled = false;
        }
        playerAnimations.SetAnimationMode(AnimationMode.Carrying);
        playerAnimations.UpdateAnimation();
    }
}

public void SetHeldItem(Throwable newThrowable)
{
    if (carrying)
    {
        Throw(0, Vector2.down);
    }
    if (newThrowable != null)
    {
        currentThrowable = newThrowable;
        currentThrowable.transform.parent = heldItemPosition;
        currentThrowable.transform.localPosition = Vector2.zero;
        currentThrowable.HoldItem();
        if (currentThrowable.CompareTag("Head"))
        {
            currentThrowable.transform.rotation = Quaternion.identity;
            currentThrowable.transform.localScale = new Vector3(2, 2);
        }
        Invoke(nameof(SetCarryingTrue), pickupGracePeriod);
        heldItemRenderer.enabled = true;
        playerMovement.ArmSFX(true);
    }
    else
    {
        EnableArms();
    }
}

private void Throw(float force, Vector2 direction)
{
    if (currentThrowable == null) return;
    currentThrowable.ThrowItem(force, direction);
    currentThrowable = null;
    currentGrabable = null;
    carrying = false;
    aiming = false;
    currentForce = 0;
    playerMovement.SetMovementStatus(carrying, aiming);
    heldItemRenderer.enabled = false;
    grabDelayTimer = 0.7f;
    EnableArms();
    if (force != 0)
    {
        playerMovement.ThrowSFX();
        playerAnimations.ThrowAnimation();
        playerAnimations.SetAnimationMode(AnimationMode.None);
    }
    else
    {
        playerAnimations.SetAnimationMode(AnimationMode.None);
        playerAnimations.UpdateAnimation();
    }
}

float GetForceModifier(Vector2 delta)
{
    //Throws object based on mouse position
    if (activeInputType == InputType.KeyboardMouse)
    {
        return Mathf.Clamp(delta.magnitude * mouse_forceModifier / maxThrowForceModifier, 3, maxThrowForceModifier);
    }
    else if (activeInputType == InputType.Controller)
    {
        return maxThrowForceModifier;
    }
    return maxThrowForceModifier;
}

Vector2 GetCalculatedPosition(float velocity, Vector2 direction, float time)
{
    //Calcuates the future position of a throw, using physics formulas for Projectile Motion in 2D space
    float x = currentThrowable.transform.position.x + direction.x * velocity * time;
    float y = currentThrowable.transform.position.y + direction.y * velocity * time - 9.82f * time * time / 2;
    return new Vector2(x, y);
}
  </code></pre>
</details>

This code, in turn, can be used to grab any **Grabable** objects, and is also able to pick up, carry and throw any **Throwable** objects.

<details><summary>Grabable.cs</summary>
  <pre>

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public abstract class Grabable : MonoBehaviour
{
    [SerializeField] bool requireClickToGrab = true;
    public bool RequireClick { get { return requireClickToGrab; } }

    protected Collider2D col;
    public Collider2D Collider { get { return col; } }
    public abstract void HoldItem();

    [SerializeField] protected GameObject highlight;
    
    private void Start()
    {
        if (GrabablesManager.Instance != null && GrabablesManager.Instance.IsInitialized && !GrabablesManager.Instance.Grabables.Contains(this))
        {
            GrabablesManager.Instance.Grabables.Add(this);
        }
    }

    private void OnEnable()
    {
        if (GrabablesManager.Instance != null && GrabablesManager.Instance.IsInitialized && !GrabablesManager.Instance.Grabables.Contains(this))
        {
            GrabablesManager.Instance.Grabables.Add(this);
        }
    }

    private void OnDisable()
    {
        if (GrabablesManager.Instance == null) { return; }
        GrabablesManager.Instance.Grabables.Remove(this);
    }

    private void OnDestroy()
    {
        if(GrabablesManager.Instance == null) { return; }
        GrabablesManager.Instance.Grabables.Remove(this);
    }

    public virtual void ToggleGrabableVisual(bool toggle)
    {
        if (highlight != null)
        {
            highlight.SetActive(toggle);
        }
    }
}
  </pre>
</details>

<details><summary>Throwable.cs</summary>
  <pre>

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Throwable : Grabable
{
    public Rigidbody2D Rigidbody { get; private set; }

    Transform defaultParent;
    public Transform DefaultParent { get { return defaultParent; } }
    [SerializeField] AudioClip collisionClip;
    [SerializeField] float throwRotation = 25;
    [SerializeField] bool causeCamShake = false;
    [SerializeField] bool resetOnDeath = false;
    [SerializeField] float maxShakeAmount = 0.15f;
    ParticleSystem ps;
    bool isHeld;
    public bool IsHeld { get { return isHeld; } }
    CameraMovement cameraMovement;

    public static Throwable Head {get; private set;}

    private void Awake()
    {
        Rigidbody = GetComponent&#60;Rigidbody2D&#62;();
        col = GetComponent&#60;Collider2D&#62;();
        if(!TryGetComponent&#60;ParticleSystem&#62;(out ps))
        {
            ps = GetComponentInChildren&#60;ParticleSystem&#62;();
        }

        defaultParent = transform.parent;
        cameraMovement = Camera.main.GetComponent&#60;CameraMovement&#62;();
        if(gameObject.CompareTag("Head"))
        {
            Head = this;
        }
    }

    public override void HoldItem()
    {
        if(!enabled)
        {
            return;
        }

        isHeld = true;
        Rigidbody.bodyType = RigidbodyType2D.Static;
        col.enabled = false;
        Rigidbody.simulated = false;
    }

    public void ThrowItem(float force, Vector2 direction)
    {
        isHeld = false;
        transform.parent = defaultParent;
        //col.enabled = true;
        Rigidbody.bodyType = RigidbodyType2D.Dynamic;
        Rigidbody.simulated = true;
        Rigidbody.AddTorque(throwRotation * direction.x);
        Rigidbody.AddForce(force * direction, ForceMode2D.Impulse);
        Invoke(nameof(EnableCollider), 0.1f);
    }

    void EnableCollider()
    {
        col.enabled=true;
    }

    public override void ToggleGrabableVisual(bool toggle)
    {
        if (ps != null)
        {
            if (toggle)
            {
                ps.Play();
            }
            else
            {
                ps.Stop();
            }
        }

        if (highlight != null)
        {
            highlight.SetActive(toggle);
        }
    }

    private void OnCollisionEnter2D(Collision2D collision)
    {
        if(collisionClip == null)
        {
            return;
        }

        if(collision.relativeVelocity.sqrMagnitude > 0.01f)
        {
            AudioManager.Instance.PlayAudio(collisionClip, transform.position);
            
            if (causeCamShake)
            {
                cameraMovement.CameraShake(Mathf.Clamp(maxShakeAmount*(collision.relativeVelocity.sqrMagnitude*0.01f),0.1f,maxShakeAmount), collision.relativeVelocity);
            }
        }

        if(collision.gameObject.CompareTag("Player") && collision.enabled)
        {
            //Child player to throwable
            collision.transform.parent = transform;
        }
    }
    private void OnCollisionExit2D(Collision2D collision)
    {
        if (collision.gameObject.CompareTag("Player") && collision.enabled)
        {
            collision.transform.parent = null;
        }
    }

    public void ResetMe()
    {
        if (!resetOnDeath || gameObject.CompareTag("Head")) return;

        transform.localPosition = Vector3.zero;
        transform.eulerAngles = Vector3.zero;

        Rigidbody.angularVelocity = 0f;
        Rigidbody.velocity = Vector3.zero;
    }
}

  </pre>
</details>



In this case, since Throwable inherits from Grabable, the grabbing system is also able to recognise Throwable objects as grabable.

### The Signal System

The signal system the game uses is based on two classes, SignalReciever and SignalTransmitter:

<details><summary>SignalBases.cs</summary>
  <pre>

using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

public abstract class SignalTransmitter : MonoBehaviour
{
    protected List&#60;SignalReciever&#62; recievers = new();
    public List&#60;SignalReciever&#62; Recievers { get { return recievers; } set { recievers = value; } }

    protected bool state;

    public abstract void TransmitSignal();

    public abstract bool GetSignalState();
}

public abstract class SignalReciever : MonoBehaviour
{
    [SerializeField] protected Transmitter[] sources;

    [SerializeField] protected UnityEvent onSignalActivated, onSignalDeactivated;

    protected bool? state = null;

    protected virtual void Start()
    {
        if(sources == null || sources.Length == 0) 
        {
            return; 
        }

        foreach (var source in sources)
        {
            if(source != null && source.Source != null)
            {
                source.Source.Recievers.Add(this);
            }
        }

        RecieveSignal();
    }

    public abstract void RecieveSignal();
}


[System.Serializable]
public class Transmitter
{
    [SerializeField] SignalTransmitter source;
    public SignalTransmitter Source { get { return source; } }

    [SerializeField] bool activeState = true;
    public bool ActiveState { get { return activeState; } }
}
  </pre>
</details>

In this script, there are two MonoBehaviours, however, since they are marked as *abstract*, it does not matter, since no instances of these classes can be made anyway.

These classes are built to be inherited from, and the idea is that every object that can send a signal inherits from (or in other ways uses) the SignalTransmitter class. Likewise, any class that is supposed to recieve a signal must inherit from- or use the SignalReciever class.

The most generic implementation of a signal reciever used in the project is like the following:

<details><summary>BasicReciever.cs</summary>
  <pre>

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class BasicReciever : SignalReciever
{
    public override void RecieveSignal()
    {
        bool active = true;
        foreach (var source in sources)
        {
            if(source.Source.GetSignalState() != source.ActiveState)
            {
                active = false;
                break;
            }
        }

        state ??= !active;

        if (state != active)
        {
            if (active)
            {
                state = true;
                if(onSignalActivated.GetPersistentEventCount() &#62; 0)
                    onSignalActivated.Invoke();
            }
            else
            {
                state = false;
                if (onSignalDeactivated.GetPersistentEventCount() &#62; 0)
                    onSignalDeactivated.Invoke();
            }
        }
    }

    private void OnDrawGizmosSelected()
    {
        if((sources == null) || (sources != null && sources.Length == 0))
        {
            return;
        }

        foreach (var source in sources)
        {
            if (source.Source != null)
            {
                if (source.ActiveState)
                {
                    Gizmos.color = Color.green;
                }
                else
                {
                    Gizmos.color = Color.red;
                }

                Gizmos.DrawLine(transform.position, source.Source.transform.position);
            }
        }
    }
}
  </pre>
</details>

Pretty much every single signal reciever ever used in the game is a BasicReciever, because while it is basic it is extremely flexible.
Since I was using UnityEvents for this system, giving them the **[SerializeField]** attribute exposes the event in the Unity inspector, making it possible to add listeners outside of the playmode runtime.

In adittion, for ease of use, I also use the **OnDrawGizmosSelected()** Unity Message to make each basic reciever draw a line towards each of the reciever's transmission sources.
