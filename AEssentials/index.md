# The AEssentials Package

![AEssentials](/../assets/AEssentials.png)

## Singletons  

Singletons are the relatively common programming pattern where you ensure that there can only be one instance of a specific class.  
While there are many ways do make singletons, the following two images show how I decided to create the pattern inside the Unity Game Engine:

(NOTE: Both of these showcases has had their comments and summaries removed to save on image size and improve readability!)  

![Singleton](/assets/AE_Singleton.png)  

![LazySingletonPersistent](/assets/AE_SingletonLazyPersistent.png)  

These showcase the simplest and most complex version of this pattern that I decided to create.  

When a class inherits from **Singleton**, it ensures that there can only be at most one instance of the class in a scene. It also gains an *Instance* property which can be used by any code to locate the class instance.  

While the **LazySingletonPersistent** class is fundementally the same, it has two adittional features that the  Singleton class does not have. The first, and most simple is that the singleton persists between scenes by using the DontDestroyOnLoad() function.  
Secondly, and more importantly, is the fact that the instance can be *lazily initialized*. If a class instance does not exist in the active scene, calling the Instance property causes an instance of the class to be created.  

Another class that has Singleton-like behaviour is the following:  
![ServiceHandler](/assets/AE_Service.png)  

This is a class that functions similairly to the previous Singleton classes, in the sence that they have a Lazy-Instance pattern. Do however note that the ServiceHandler class is a pure C# class, that does not need to be placed inside the Unity Game Engine to function.  

## Object Management

My package contains a set of classes that provide different ways of managic objects and data. One of the more interesting in my opinion is the **URef** class:

![URef](/assets/AE_URef.png)  

This class is fundementally simple; it is a wrapper-class. The idea is that it allows me to create a pseudo-reference to any type ever, and while it might not seem very useful, it fills a very important role that opens up more ways to help me make good code.  
  
  

When learning how to make games, one of the common things I learned was that it was bad to destroy objects, as that can cause lag-spikes due to the C# garbage collector. One of the many workarounds is a pattern known as Object-Pooling.  

![Pool](/assets/AE_Pool.png)  

This class is a way to manage objects in a way that is friendly on memory. By simply calling **Pool.Get()**, you get the first inactive object in the pool and activate it. Also, as you can see, the class contains a handful of other functions, such as prewarming and trimming.  
It should be noted that this class works on C# objects, however if you intend to use this in Unity for MonoBehaviours, there is another version of the class called **MonoPool** which is fundementally the same, but has slight different code as to adhere to the restrictions placed on Monobehaviours.  

# Tweening  

One package that exists for Unity is called "DOTween". What it does is allow you to "tween" values; changing their value over time.  
I do not need to use DOTween, because I have made my own version, **ATween**.

ATween is still a part of the AEssentials package, and is actively being worked on, seeing as it is relatively new.

One of the main problems I encountered while developing this code was value types. How could I change the value of a value type variable from another part of the code?  
The answer is actually quite simple, and has been mentioned already: the **URef** class.  

I can create a URef&#60;float&#62;, and then tween the float inside the URef. Since the URef is a class, and as such a reference-type, the internal value-type will still get tweened as expected.  

Tweens are created by using extention functions, allowing me to "attach" new functions to already existing types:

![TweenFloat](/assets/AT_TweenFloat.png)  

Using this same structure, I can, theoretically, define tweens for any type that I want.  

This code creates a Tween and passes is to the TweenUpdater class:

![TweenUpdater](/assets/AT_TweenUpdater.png)  

And finally, for the actual Tween class:

![Tween](/assets/AT_Tween.png)  



An example of how to use this tween code could be as follows:  

![How to ATween](/assets/AT_Example.png)  

This causes the gameObject to smoothly move 7 units along the x-axis, and then move 7 units back, back and forth, forever.