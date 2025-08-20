[<< Back](https://salmaster1.github.io/Portfolio/)

# Monkey Paw

Monkey Paw was developed duting by time at the school Yrgo in Gothenburg. The Project was a part of the *Game Production* course, and was developed over a time of approxomately 8 weeks (2 months).

During this project, my role was as a programmer. The game was developed in the Unity Game Engine using the language C#.

During development, I mainy worked on the interaction system, as well as the dialogue system.

## My Work

### Interacting

The interact system is, at its core, quite simple;

Any GameObject that has the **Interactable** component can be interacted with.

We can then, through UnityEvents, specify what should happen when the GameObject has been interacted with.

<details><summary>Interactable.cs (excerpt)</summary>
  <pre><code class="language-csharp">

  public class Interactable : MonoBehaviour
{
    bool isInteractable;

    [SerializeField] UnityEvent&#60;UnityEvent&#62; onInteract;

    [SerializeField] UnityEvent interactOverCallback;

    [SerializeField] InputAction interactAction;

    [SerializeField] InteractType interactType;

    string id;

    [SerializeField] bool clearInteractabilityAfterUse;

    private void Awake()
    {
        id = GetInstanceID().ToString();
    }

    private void Start()
    {
        if (interactAction.bindings.Count == 0)
        {
            Debug.LogWarning($"InteractAction on {gameObject.name} does not have any bindings!", this);
        }

        interactAction.performed += (a) =&#62; TryInteract();
        if (enabled)
        {
            interactAction.Enable();
        }
    }

    private void OnTriggerEnter(Collider other)
    {
        if (enabled && other.gameObject.CompareTag("Player"))
        {
            {
                var inst = InteractPrompt.Instance;
                if (inst != null)
                {
                    inst.AddInteraction(id, interactType);
                }
            }

            isInteractable = true;
        }
    }

    private void OnTriggerExit(Collider other)
    {
        if (enabled && other.gameObject.CompareTag("Player"))
        {
            {
                var inst = InteractPrompt.Instance;
                if (inst != null)
                {
                    inst.RemoveInteraction(id);
                }
            }

            isInteractable = false;
        }
    }

    private void OnDisable()
    {
        var inst = InteractPrompt.Instance;
        if (inst != null)
        {
            inst.RemoveInteraction(id);
        }

        isInteractable = false;

        interactAction.Disable();
    }

    private void OnEnable()
    {
        interactAction.Enable();
    }

    private void TryInteract()
    {
        if (isInteractable)
        {
            if (onInteract.GetPersistentEventCount() &#62; 0)
            {
                onInteract.Invoke(interactOverCallback);
            }
            else
            {
                interactOverCallback.Invoke();
            }

            if (clearInteractabilityAfterUse)
            {
                {
                    var inst = InteractPrompt.Instance;
                    if (inst != null)
                    {
                        inst.RemoveInteraction(id);
                    }
                }

                isInteractable = false;
            }
        }
    }
    </code></pre>
</details>>

Since the component uses the new Input System (becuse we are using an InputAction), this component becomes very cheap because we are not polling for Input.