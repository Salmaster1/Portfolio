[<< Back](https://salmaster1.github.io/Portfolio/)

# Head On

Head On was developed during my time at the school Yrgo in gothenburg. The project was a part of our *Game Design* course, here wew were to use our knowledge of game design to create a game. The project was not my idea.

During this project, my role was as a programmer, and the game was developed in the Unity Game Engine, with the programming lagnuage C#.

During the development process, the programming that I mainly focused on was the player character, the signal-button logic, and the system for grabable items.

<details><summary>PlayerGrabbing.cs (extract)</summary>
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
        if(carrying)
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

    public void ForceDrop()
    {
        if (currentThrowable != null) 
        {         
            Throw(0, Vector2.down);
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
        playerMovement.SetMovementStatus(carrying,aiming);
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

    Grabable GetClosestGrabable()
    {
        //Finds the closest throwable that is within range
        float shortestSqrDistance = maxPickupDistance * maxPickupDistance;
        Grabable throwable = null;
        Grabable[] thrA = grabablesManager.Grabables.ToArray();

        foreach (var item in thrA)
        {
            if(item.gameObject.activeInHierarchy && item.enabled)
            {
                float sqrDist = (item.transform.position - pickupPoint.position).sqrMagnitude;
                if (sqrDist < shortestSqrDistance)
                {
                    shortestSqrDistance = sqrDist;
                    throwable = item;

                    // Prioritize ledges if can be grabbed
                    if (item.GetComponent<Ledge>() && PlayerMovement.Instance.transform.position.y < item.transform.position.y) break;

                    // Prioritize head
                    if (item.CompareTag("Head"))  break;

                }
            }
        }
        return throwable;
    }

    float GetForceModifier(Vector2 delta)
    {
        //Throws object based on mouse position

        if(activeInputType == InputType.KeyboardMouse)
        {
            return Mathf.Clamp(delta.magnitude * mouse_forceModifier/maxThrowForceModifier, 3, maxThrowForceModifier);
        }
        else if(activeInputType == InputType.Controller)
        {
            return maxThrowForceModifier;
        }
        return maxThrowForceModifier;
    }

    Vector2 GetCalculatedPosition(float velocity, Vector2 direction, float time)
    {
        //Calcuates the future position of a throw, using physics formulas for Projectile Motion in 2D space
        float x = currentThrowable.transform.position.x + direction.x * velocity * time;
        float y = currentThrowable.transform.position.y + direction.y * velocity * time - 9.82f*time*time/2;

        return new Vector2(x, y);  
    }

</code></pre></details>