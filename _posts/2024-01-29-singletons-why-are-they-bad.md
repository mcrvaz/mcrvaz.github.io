---
title: "Singletons: Why are they bad?"
excerpt: "So, what makes Singletons so bad? Should you avoid them? How can you replace them? Let's find out!"
date: 2024-01-29 00:00:00 -03:00
classes: wide
categories:
  - blog
tags:
  - software-architecture
  - unity
  - c#
---

> **"Singletons are bad!"**

You've probably heard this one before, or at least some variation of it. 

This opinion gets thrown around pretty often, especially in game dev communities, and most of the time there isn't really any explanation along with it.
So, what makes Singletons so bad? Should you avoid them? How can you replace them?

Let's find out!

## What's a Singleton?
It's a [design pattern](https://en.wikipedia.org/wiki/Software_design_pattern) used to ensure that a class has exactly one instance and also provides static access to it.

There are many different implementations of it (such as `SingletonMonoBehaviour`, which is pretty common in Unity), but let's take this simple one as our reference:
```csharp
public class MySingleton {
    // access it globally using MySingleton.Instance
    public static MySingleton Instance => _instance ??= new MySingleton(); 
    static MySingleton _instance;

    // private constructor so we can ensure that only one instance exists
    MySingleton() { }
}
```

## Benefits
### Data Sharing
It's easy to share data between Unity scenes with Singletons. You can access any Singleton in any scene, since they're available globally.

### Ease of access
It's fast and easy to access Singletons. There's a clear winner when comparing searching the scene for an object (`FindObject`, `GetComponent`, *etc*.) and simply accessing `MySingleton.Instance`.

## Issues

### Global State
If you're using [Play Mode Options - Domain Reloading](https://docs.unity3d.com/Manual/DomainReloading.html) in your project, and you probably should be, you'll need to avoid using static fields. If you're using Singletons along with **Play Mode Options - Domain Reloading**, they'll preserve information from the previous play mode sessions, and it can cause issues that might be pretty hard to track. 

### Guarantees of Uniqueness
It's hard to have an absolute guarantee of having only a single instance of a class. You can instantiate things with [Reflection](https://learn.microsoft.com/en-us/dotnet/api/system.reflection?view=net-8.0), your Singleton might be a `MonoBehaviour`, and programmers are usually very creative for circumventing arbitrary limitations.
Therefore, this isn't a problem that a simple Singleton implementation will solve by itself; you'll need to apply considerable effort here.

### Lazy Initialization
Singletons are usually lazy-loaded; they're instantiated when called for the first time. This is usually a good thing, since you only grab the resources when you actually need them. However, sometimes it can be harmful to your game's performance.

Imagine the following scenario:

The enemy is about fire a bullet targeting the player and it has to run the following code just before firing:
```csharp
float bulletDamage = EnemySettingsDatabase.Instance.GetBulletDamage(enemyLevel);
```
If the `EnemySettingsDatabase.Instance` wasn't accessed before, this will actually load the entire `EnemySettingsDatabase` in memory here, and if it's big enough, the game might even freeze for a while.

{% include figure image_path="/assets/images/posts/2024-01-29-singletons-why-are-they-bad/atomicrops.png" alt="Atomicrops" caption="This would be a terrible time for loading assets. From [Atomicrops](https://www.atomicrops.com/)." %}

With all that said, this can easily be worked around by accessing the Singleton beforehand in a more appropriate location, like a loading screen.

### Singletons vs Dependency Injection
Take a look at the following code snippets:

```csharp
// Singleton version
public class Enemy {
    public int MaxHealth => MeleeEnemySettings.Instance.GetMaxHealth();

    public Enemy() { }
}
```
```csharp
// Dependency Injection (DI) version:
public class Enemy {
    public int MaxHealth => settings.GetMaxHealth();

    readonly IEnemySettingsDatabase settings;

    public Enemy(IEnemySettingsDatabase settings)
    {
        this.settings = settings;
    }
}
```

The first thing you'll probably notice is how much shorter the Singleton version is. At first, it looks like a positive point for Singletons, but let's dig deeper.

**Coupling**  
Notice that the Singleton approach is using a concrete class for settings access, the `MeleeEnemySettings`, while the DI approach is using an interface, `IEnemySettingsDatabase`. This gives a lot more flexibility for the DI version. You can now use the same `Enemy` class and simply provide a different settings instance, like a `MeleeEnemySettings`, or a `BossEnemySettings`, and it'll behave in a different manner.

**Testability**  
Testability goes hand in hand with coupling. If your class is coupled to the actual `MeleeEnemySettings` implementation, it means that altering the values might also break your automated tests. In the DI version you're able to use mock values and create unit tests easily.

**Implicit dependencies**  
It's trivial to list all the dependencies of the DI version since they're stated in the constructor. On the other hand, it's not so trivial in the Singleton version, because you need to look at the whole class for static accessess.
Knowing the class' dependencies is important to create reasonable systems (Should your pathfinding system be coupled with your trading system? Probably not!) and avoid circular dependencies.

## Conclusion
Have you ever been affected by any of the issues listed above and you still use Singletons? If so, it might be time to ditch them and consider using a more robust approach with Dependency Injection — a topic that I'll cover in-depth later. If not, well, who needs this automated testing stuff anyways, right?

In any case, try to be pragmatic. What's your current project? Are you creating a game for a Game Jam? Are you prototyping something? It might be perfectly fine to use Singletons in those cases, while in a more serious project you'll probably want a robust architecture that avoids them.
