[<< Back](https://salmaster1.github.io/Portfolio/)  

# The AEssentials Package

![AEssentials](/../assets/AEssentials.png)

## Singletons  

Singletons are the relatively common programming pattern where you ensure that there can only be one instance of a specific class.  
While there are many ways do make singletons, the following two images show how I decided to create the pattern inside the Unity Game Engine:

(NOTE: All code showcases in this page has had their comments and summaries removed to save on image size and improve readability!)  


<details><summary>Singleton</summary>
  <pre>
   <img src="https://salmaster1.github.io/Portfolio/AEssentials/assets/AE_Singleton.png" alt="Singleton">
  </pre>
</details>

<details><summary>LazySingletonPersistent</summary>
  <pre>
   <img src="https://salmaster1.github.io/Portfolio/AEssentials/assets/AE_SingletonLazyPersistent.png" alt="LazySingletonPersistent">
  </pre>
</details>

These showcase the simplest and most complex version of this pattern that I decided to create.  

When a class inherits from **Singleton**, it ensures that there can only be at most one instance of the class in a scene. It also gains an *Instance* property which can be used by any code to locate the class instance.  

While the **LazySingletonPersistent** class is fundementally the same, it has two adittional features that the Singleton class does not have. The first, and most simple is that the singleton persists between scenes by using the DontDestroyOnLoad() function.  
Secondly, and more importantly, is the fact that the instance can be *lazily initialized*. If a class instance does not exist in the active scene, calling the Instance property causes an instance of the class to be created.  

Another class that has Singleton-like behaviour is the following:  
<details><summary>ServiceHandler</summary>
  <pre>
   <img src="https://salmaster1.github.io/Portfolio/AEssentials/assets/AE_Service.png" alt="ServiceHandler">
  </pre>
</details>

This is a class that functions similairly to the previous Singleton classes, in the sence that they have a Lazy-Instance pattern. Do however note that the ServiceHandler class is a pure C# class, that does not need to be placed inside the Unity Game Engine to function.  

## Object Management

My package contains a set of classes that provide different ways of managic objects and data. One of the more interesting in my opinion is the **Ref** class:

<details><summary>URef</summary>
  <pre>
   <img src="https://salmaster1.github.io/Portfolio/AEssentials/assets/AE_Ref.png" alt="Ref">
  </pre>
</details>

This class is fundementally simple; it is a wrapper-class. The idea is that it allows me to create a pseudo-reference to any type ever, and while it might not seem very useful, it fills a very important role that opens up more ways to help me make good code.  
  
  

When learning how to make games, one of the common things I learned was that it was bad to destroy objects, as that can cause lag-spikes due to the C# garbage collector. One of the many workarounds is a pattern known as Object-Pooling.  

<details><summary>Pool</summary>
  <pre>
   <img src="https://salmaster1.github.io/Portfolio/AEssentials/assets/AE_Pool.png" alt="Pool">
  </pre>
</details>

This class is a way to manage objects in a way that is friendly on memory. By simply calling **Pool.Get()**, you get the first inactive object in the pool and activate it. Also, as you can see, the class contains a handful of other functions, such as prewarming and trimming.  
It should be noted that this class works on C# objects, however if you intend to use this in Unity for MonoBehaviours, there is another version of the class called **MonoPool** which is fundementally the same, but has slight different code as to adhere to the restrictions placed on Monobehaviours.  

# Tweening  

In programming, there exists the idea of "Tweening", that being the process of changing a value from A to B over a set period of time.  
While there are several packages out there that do this, the AEssentials package contains my own version, **ATween**.

One of the main problems I encountered while developing this code was value types. How could I change the value of a value type variable from another part of the code?  
The answer is actually quite simple, and has been mentioned already: the **Ref** class.  

I can create a Ref&#60;float&#62;, and then tween the float inside the Ref. Since the Ref is a class, and as such a reference-type, the internal value-type will still get tweened as expected.  

Tweens are created by using extention functions, allowing me to "attach" new functions to already existing types:

<details><summary>TweenFloat</summary>
  <pre>
   <img src="https://salmaster1.github.io/Portfolio/AEssentials/assets/AT_TweenFloat.png" alt="TweenFloat">
  </pre>
</details>

Using this same structure, I can, theoretically, define tweens for any type that I want.  

This code creates a Tween and passes is to the TweenUpdater class:

<details><summary>TweenUpdater</summary>
  <pre>
   <img src="https://salmaster1.github.io/Portfolio/AEssentials/assets/AT_TweenUpdater.png" alt="TweenUpdater">
  </pre>
</details>

And finally, for the actual Tween class:

<details><summary>Tween</summary>
  <pre>
   <img src="https://salmaster1.github.io/Portfolio/AEssentials/assets/AT_Tween.png" alt="Tween">
  </pre>
</details>



An example of how to use this tween code could be as follows:  

<details><summary>How to ATween</summary>
  <pre>
   <img src="https://salmaster1.github.io/Portfolio/AEssentials/assets/AT_Example.png" alt="Example">
  </pre>
</details>

This causes the gameObject to smoothly move to position (3, 3), and then move to (1, 1), back and forth, forever.  

# Driven Variables

**Driven Variables** is a system that I have implemented that is based on a common but powerful pattern.  

The main idea is that certain variable types, such as *int, float, string* etc. are value types. What this means is that it is not possible to create a direct reference to the variable.  

The main problem with this is that if Class1 needs a variable that is in Class2, you run into the issue of Class2 needing to exist in order for Class1 to work.  

Driven Variables aims to solve part of this issue, by delegating variables into ScriptableObject containers.  

Now whenever you might need access to a variable in many locations, instead of continuously needing to make sure that the variable even exists in the first place, you can create a Driven Variable of the same type, and all accessors can operate on this variable without needing to coexist.

AEssentials comes prebuilt with Driven Variables for most, if not all, standard structs in Unity and C#, which includes, but is not limited to: int, float, string, bool, Vector2, Quaternion, byte, and long.  

These ScriptableObjects also have some aditional functionality; In the inspector, you can mark your variables as "Read Only", strictly prohibiting code from changing the value. Objects can also "lock" the variables, restricting set access until all locks have been removed.  

The built-in driven variables also come with unique icons, to help distinguish them from other scriptableObjects:  

![Bool](/assets/Bool.png)![Float](/assets/Float.png)![Int](/assets/Int.png)![Vector2](/assets/Vector2.png)  
(These icons are for bools, floats, ints, and Vector2 respectively)  

Ultimately, Driven Variables exists to help make more modular and flexible code, whilst remaning clean and efficient.  
