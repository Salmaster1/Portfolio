[<< Back](https://salmaster1.github.io/Portfolio/)

# Gravitas  

Gravitas is a game that I have been developing by myself for the past year or so (as of time of writing).  

The game is a puzzle-platformer, where you as teh player has to utilize various gravity-based abilities to complete the game's levels.  

## The Gravity  

Gravity is a funny thing. While it is something that we, as people, take for granted; it is not something that becomes immediately obvious when trying to make it yourself.  

What we as people refer to as "down", is the direction in which gravity is pulling us. In Unity, there is (optionally) gravity, that works as you'd expect. You place out a physics object, and it accelerates down.  

But what if you want your own, custom gravity?  Unity does have the ability to change the Physics2D gravity vector, however I wanted something different, something that I could control and customize to my liking.  

<details><summary>GravityController.cs (excerpt)</summary>
  <pre><code class="language-csharp">

    void UpdateGravity()
    {
        if (textWindow.IsDisplayingText || (abilityCount &#60;= 0))
        {
            return;
        }

        if (Input.GetAxisRaw("Gravity Vertical") != 0)
        {
            previousParticleSystem.SetActive(false);

            Gravity = 9.81f * Input.GetAxisRaw("Gravity Vertical") * Vector2.up;
            CountAbility();

            if(Gravity.y &#62; 0)
            {
                upSystem.SetActive(true);
            }
            else
            {
                downSystem.SetActive(true);
            }
        }
        else if (Input.GetAxisRaw("Gravity Horizontal") != 0)
        {
            previousParticleSystem.SetActive(false);

            Gravity = 9.81f * Input.GetAxisRaw("Gravity Horizontal") * Vector2.right;
            CountAbility();

            if(Gravity.x &#62; 0)
            {
                rightSystem.SetActive(true);
            }
            else
            {
                leftSystem.SetActive(true);
            }
        }
    }

    </code></pre>
</details>>
(NOTE: This code was written some time ago, and does not reflect my current skillset as a programmer.)  

With this code, by pressing the arrow keys you as the player will change the direction of my custom Gravity.

This gravity is then used in all physics-object, through the **GravityReciever** component.

<details><summary>GravityReciever.cs</summary>
  <pre><code class="language-csharp">

[RequireComponent(typeof(Rigidbody2D))]
public class GravityReciever : MonoBehaviour
{
    Rigidbody2D rb2D;
    GravityController gravityController;
    IGravityReciever gravityReciever;
    GravityOrb pull, push;

    private void Awake()
    {
        if(!TryGetComponent(out gravityReciever))
        {
            rb2D = GetComponent&#60;Rigidbody2D&#62;();
        }
    }

    private void Start()
    {
        gravityController = GravityController.Instance;
        pull = GravityOrb.Pull;
        push = GravityOrb.Push;
    }

    protected virtual void FixedUpdate()
    {
        Vector2 gravitySum = (gravityController.Gravity + pull.GetGravity(transform.position) + push.GetGravity(transform.position)) * Time.fixedDeltaTime;

        if (gravityReciever != null)
        {
            gravityReciever.ApplyGravity(gravitySum);
        }
        else
        {
            rb2D.linearVelocity += gravitySum;
        }
    }
}

public interface IGravityReciever
{
    public abstract void ApplyGravity(Vector2 gravity);
}
    </code></pre>
</details>>

This code will every FixedUpdate get the gravity vector for its current position, and accelerate in that direction.  

All gravityRecievers will have a Rigidbody2d, due to the [RequireComponent] at the top, however if there *also* is an IGravityReciever, it will gain the acceleration instead. As of writing this, the only IGravityReciever is the PlayerMovement class.  

## Movement

When making movement for a game, most people make a pretty generous assumption: that down is... well down.  

*But what is that wasn't the case?*  

Since the player's down can be in any orthogonal direction, its movement code has to reflect that.  

<details><summary>PlayerMovement.cs (excerpt)</summary>
  <pre><code class="language-csharp">

    void Movement()
    {
        if (input.sqrMagnitude &#60; 0.25f)
        {
            Decelerate();
        }
        else
        {
            if (down.x == 0)
            {
                if ((internalVelocity.x) * input.x &#60; maxSpeed)
                {
                    internalVelocity += acceleration * input.x * Time.fixedDeltaTime * Vector2.right;

                    if((internalVelocity.x &#60; 0) == (externalVelocity.x &#62; 0))
                    {
                        externalVelocity.x = 0;
                    }

                    if((internalVelocity.x &#62; 0) == (input.x &#60; 0))
                    {
                        internalVelocity += deceleration * input.x * Time.fixedDeltaTime * Vector2.right;
                    }
                }
            }
            else if (down.y == 0)
            {
                if ((internalVelocity.y) * input.y &#60; maxSpeed)
                {
                    internalVelocity += acceleration * input.y * Time.fixedDeltaTime * Vector2.up;

                    if ((internalVelocity.y &#60; 0) == (externalVelocity.y &#62; 0))
                    {
                        externalVelocity.y = 0;
                    }

                    if ((internalVelocity.y &#62; 0) == (input.y &#60; 0))
                    {
                        internalVelocity += deceleration * input.y * Time.fixedDeltaTime * Vector2.up;
                    }
                }
            }
        }
    }
        </code></pre>
</details>>

This code is just a small snippet to demonstrate the somewhat bizzare code I had to make in order to support this kind of movement.  

Do also note that in order to move up and down a wall (assuming the current gravity is sideways), you need to use the [W] and [S] keys.  

Also, doe to applying gravity to this class and not the rigidbody, I need to seperate velocity into **internalVelocity** and **externalVelocity**  

InternalVelocity is the velocity generated by the player, such as movement input or jumping.  

ExternalVelocty is basically only the velocity caused by the current gravity.  

Later in the code, the GameObject's Rigidbody2D velocity is set to "internalVelocity + externalVelocity".  

## Text

While there any many parts of this game that I am proud of, one of the most interesting in my opition is how I manage text.  

In the game, there are several places where text is displayed in a window in screen. While fundementally simple, the way I did it allows me to easily support game localization.  

Within the game project, there is a file called "English.txt". This file contains every single piece of dialogue that can be printed into the dialogue window.  

<details><summary>English.txt (excerpt)</summary>
  <pre><code class="language-csharp">

&#35;&#35; 1-1_s
You seem to be in quite the predicament.
As you might have noticed, you are unable to jump up to the portal.
May I suggest... pressing the "up" arrow on your keyboard?
I think you'll figure it out from there.
/#

        </code></pre>
</details>>

Shown here is a piece of text in a custom format. This section is called "1-1_s", as specified by the text after the "##". Then, every line after the section declaration becomes a seperate line for the TextWindow to display. Finally, the section is closed with the characters "/#".

In this case, 1-1_s stands for *level 1-1, starting room*, however the sections name is not of high importance, as long as it is one word and is unique.

The way this supports localization is simple: just change what file you are reading from. As long as the section you are looking for exists, it will be printed out.