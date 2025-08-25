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

We were able to place this component on a lot of GameObjects, and because the component uses the new input system, it barely drains any resources.  

In adittion, it also displays the current interaction with the **InteractPrompt** class, which is a singleton class that handles the interaction prompt. When you enter an interaction trigger, it calls the **AddInteraction** function, which takes the object's InstanceID and InteractType.  

The reason I make the prompt recieve the InstanceID is to prevent duplicate interactions from the same object.  

### Dialogue  

The second important system I made for this game was the dialogue system.  

The dialogue in Monkey Paw is based on ScriptableObjects, and I made one called DialogueData.  

<details><summary>DialogueData.cs</summary>
  <pre><code class="language-csharp">

  [CreateAssetMenu(fileName = "New Dialogue", menuName = "ScriptableObjects/Dialogue")]
    public class DialogueData : ScriptableObject
    {
        [SerializeField] private List&#60;DialogueEntry&#62; dialogue;
        public List&#60;DialogueEntry&#62; Dialogue =&#62; dialogue;

        [SerializeField] private List&#60;DialogueOption&#62; dialogueOptions;
        public List&#60;DialogueOption&#62; Options =&#62; dialogueOptions;

        [SerializeField] private DialogueData dialogueAfterConversation;
        public DialogueData PostDialogue =&#62; dialogueAfterConversation;

        [SerializeField] private AudioClip dialogueMusic;
        public AudioClip DialogueMusic =&#62; dialogueMusic;

        [SerializeField] private DialogueData branchInto;
        public DialogueData BranchInto =&#62; branchInto;

        [SerializeField] private List&#60;GameEvent&#62; triggersGameEvent;
        public List&#60;GameEvent&#62; TriggersGameEvent =&#62; triggersGameEvent;

        [Serializable]
        public class DialogueOption
        {
            public string name;
            public DialogueData dialogue;
            public string command;
        }
    }

    [Serializable]
    public enum Entity
    {
        Player,
        Mother,
        Child,
        Friend,
        Boss,
        None,
    }

    [Serializable]
    public class DialogueEntry
    {
        [SerializeField, Tooltip("Person who is speaking")]
        private Entity entity;

        [SerializeField, Multiline, Tooltip("The text that will be printed into the dialogue window")]
        private string text;

        [SerializeField, Tooltip(
             "The command the CommandInterpreter will execute. \nCommands are seperated by a bar character ' | '\n\n" +
             "The available commands are:\n" +
             "   &#60;b&#62;log&#60;/b&#62; {string}\n" +
             "   &#60;b&#62;money&#60;/b&#62; {int}\n" +
             "   &#60;b&#62;debt&#60;/b&#62; {int}\n" +
             "   &#60;b&#62;call&#60;/b&#62; {string} {args}\n" +
             "   &#60;b&#62;flag&#60;/b&#62; {string} {boolean}\n" +
             "   &#60;b&#62;item&#60;/b&#62; {'add'/'remove'} {string}\n" +
             "   &#60;b&#62;scene&#60;/b&#62; {int/string}")]
        private string command;

        public Entity Entity =&#62; entity;
        public string Text =&#62; text;
        public string Command =&#62; command;
    }

    </code></pre>
</details>>

The DialogueData class, among other things, contains a list of DialogueEntries.  

DialogueEntry is a pure c# class that contain information about a specific line of dialogue. The DialogueData is the conversation, while the DialogueEntry is each character's line.

Each dialogeEntry contains information such as:  
- What character is speaking
- What is the character saying
- Are there any special functions that should be called while they are speaking

The "command" here is special. In most cases, it is completely blank, however in  the game there are several moments where, for example, people give you money. This act of "giving money" is purely done through the command system, which is in turn called by the dialogue system.

The DialogueData contains information such as:
- What lines of dialogue should be written out  
- What music should play during the dialogue  
- Does the dialogue branch into multiple options  
- ...and more!  

These DialogueDatas can be fed into the DialogueWindow class, which contains a (very) large Coroutine that writes out all the text as you'd expect.  

The Dialogue System and interaction system also go hand-in-hand, through the DialogueHolder class.  

<details><summary>DialogueData.cs</summary>
  <pre><code class="language-csharp">

  public class DialogueHolder : MonoBehaviour
{
    [SerializeField] DialogueData dialogue;
    GameObject dialogueWindow;

    public DialogueData Dialogue
    {
        get { return dialogue; }
        set { dialogue = value; }
    }

    [SerializeField] DialogueData reservedDialogue;

    public DialogueData ReservedDialogue
    {
        get { return reservedDialogue; }
        set { reservedDialogue = value; }
    }

    Animator animator;

    [SerializeField] Transform cameraMovePos;

    void Start()
    {
        dialogueWindow = GameObject.Find("Dialogue").transform.GetChild(0).gameObject;
        animator = GetComponentInChildren&#60;Animator&#62;();
    }

    public void OnInteract()
    {
        if (dialogueWindow.activeInHierarchy)
        {
            return;
        }
        Action _callback = () =&#62; { };
        if (animator != null)
        { 
            _callback += () =&#62; animator.SetBool("IsTalking", false); 
        }
        _callback += () =&#62; DialogueCamera.Instance.StartTransition(CameraManager.Instance.MainCamera.transform, 0.5f,
            DialogueCamera.TransitionMode.ToPlayer);

        DialogueCamera.Instance.StartTransition(cameraMovePos, 0.5f, DialogueCamera.TransitionMode.FromPlayer);

        DialogueWindow.Instance.StartDialogue(dialogue, _callback, transform.position, reservedDialogue, this);
        if (animator != null)
        { 
            animator.SetBool("IsTalking", true); 
        }
    }

    public void OnInteract(UnityEvent callback)
    {
        if (dialogueWindow.activeInHierarchy)
        {
            return;
        }
        if (dialogue.DialogueMusic != null)
        {
            MusicManager.Instance.FadeOutAndPlayNew(dialogue.DialogueMusic);
        }

        if (animator != null)
        { 
            animator.SetBool("IsTalking", true); 
        }
        Action _callback = () =&#62; { callback.Invoke(); };
        if (animator != null)
        { 
            _callback += () =&#62; animator.SetBool("IsTalking", false); 
        }
        _callback += () =&#62; DialogueCamera.Instance.StartTransition(CameraManager.Instance.MainCamera.transform, 0.5f,
            DialogueCamera.TransitionMode.ToPlayer);

        DialogueCamera.Instance.StartTransition(cameraMovePos, 0.5f, DialogueCamera.TransitionMode.FromPlayer);

        DialogueWindow.Instance.StartDialogue(dialogue, _callback, transform.position, reservedDialogue, this);

        if (reservedDialogue != null)
        {
            if (reservedDialogue.PostDialogue != null)
            {
                dialogue = reservedDialogue.PostDialogue;
            }
            else
            {
                dialogue = reservedDialogue;
            }

            reservedDialogue = null;
            return;
        }

        if (dialogue.PostDialogue != null)
        {
            dialogue = dialogue.PostDialogue;
            return;
        }
    }
}

    </code></pre>
</details>>

This class' **OnInteract()** and **OnInteract(UnityEvent callback)** can both be called from the interactables. The callback in this case is called whenever the dialogue is completely finished. 

The DialogueWindow class has the function *StartDialogue(DialogueData data, Action callback, ...)*.  

Since the StartDialogue function takes an Action, it does not nesessairly have to be a dialogue holder that starts dialogue. In the games intro and tutorial, the tutorial dialogue is called directly from code, and the callback simply is the continued execution of the code.  


<details>
<summary>Test</summary>

```csharp

void ThisIsMyCode()
{

}

```
</details>