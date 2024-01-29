---
title: "Singletons: Why are they bad?"
date: 2024-01-29 00:00:00 -03:00
classes: wide
categories:
  - blog
tags:
  - software-architecture
  - unity
  - c#
---

**"Singletons are bad!"** - You've probably heard this one before, or at least some variation of it. 

This opinion gets thrown around pretty often, specially in game dev communities, and usually there isn't really any explanation along with it. 
So, What makes it so bad? Should you avoid them?

Let's find out!

## What's a Singleton?
It's a design pattern used to ensure that a class has exactly one instance and also provides static access to it.

There are many different implementations of it (`SingletonMonoBehaviour` is pretty common in Unity), but let's take this simple one as our example:
```csharp
public class MySingleton {
    // access it anywhere using: MySingleton.Instance
    public static MySingleton Instance => _instance ??= new MySingleton(); 
    static MySingleton _instance;

    // private constructor so we can make sure that only one instance exists
    MySingleton() { }
}
```

## Benefits
### Data Sharing
It's easy to share data between Unity scenes with Singletons. You can access any Singleton in any scene, since they're available globally.
### Ease of access
It's fast and easy to access Singletons. There's a clear winner when comparing searching the scene for an object and simply accessing `MySingleton.Instance`.

## Issues

### Global State
If you're using [Play Mode Options - Domain Reloading](https://docs.unity3d.com/Manual/DomainReloading.html) in you project, you'll need to avoid using static fields. If you Singleton isn't storing any data, it shouldn't be a problem, but that's rarely the case. This means that if you're using Singletons along with **Play Mode Options**, they'll preserve information from the previous play mode sessions and it can cause issues that might be pretty hard to track. 

### Guarantees of Uniqueness
It's hard to have an absolute guarantee of having only a single instance of a class. You can instantiate things with reflection, your singleton might be a `MonoBehaviour`, and programmers are usually very creative for circumventing arbitrary limitations.
So this isn't a problem that a simple Singleton implementation will solve by itself, you'll need to apply a considerable effort here.

### Lazy Initialization
Singletons are usually lazy loaded, they're initialized when called for the first time. This is usually good, since you only grab the resources when you actually need them. But sometimes it can be pretty harmful for your game's performance.

Imagine the following scenario:
The enemy is just about to cast a powerfull spell targeting the player but it has to run the following code during spell cast:
```csharp
float spellDamage = EnemySettingsDatabase.Instance.GetSpellDamage(enemyLevel);
```
It will actually load the entire `EnemySettingsDatabase` in memory here, if it has never been accessed before, and if it's big enough, it will certainly cause a performance hitch 

With all that said, this can easily be worked around by simply accessing the Singleton beforehand, in a more apropriate location, like a loading screen.

### Singletons vs Dependency Injection
Take a look at the following code snippets:

Using Singleton:
```csharp
public class Enemy {
    public int MaxHealth => MeleeEnemySettings.Instance.GetMaxHealth();

    public Enemy() { }
}
```
Using Dependency Injection (DI):
```csharp
public class Enemy {
    public int MaxHealth => settings.GetMaxHealth();

    readonly IEnemySettingsDatabase settings;

    public Enemy(IEnemySettingsDatabase settings)
    {
        this.settings = settings;
    }
}
```

The first thing you'll probably notice is how shorter the Singleton version is. At first it looks like a positive point for Singletons, but let's dig deeper.

**Coupling**
Notice that that the Singleton approach is using a concrete class for settings access, the `MeleeEnemySettings`, while the DI approach is using an interface. This gives a lot more flexibility for the DI version. You can now use the same `Enemy` class and simply provide a different settings instance, like a `MeleeEnemySettings`, or a `BossEnemySettings` and it'll behave in a different manner.

**Testability**
Testability goes hand in hand with coupling. If your class is coupled to the actual `MeleeEnemySettings` values, it means that altering the values might also break your automated tests. In the DI version you're able to use mock values and create unit tests easily.

**Implicit dependencies**
It's dead obvious to list all the dependencies of the DI version since they're stated in the constructor. In the Singleton version not so much, because you need to look at the whole class for static accessess.
Knowing the class dependencies is important to create sane systems (should the logic depend on the UI? should your pathfinding system be coupled with your trade system? probably not) and avoid circular dependencies.


## Conclusion
Have you ever been affected by any of the issues listed above? If so, it might be time to ditch Singletons and consider using a more robust approach with Dependency Injection. If not, well, keep it up!

In any case, try to be pragmatic. What's your current project? Are you creating a game for a Game Jam? Are you prototyping something? It might be perfectly fine to use Singletons in those cases, while in more serious projects you'll probably want a more robust architecture that avoids them.

## Additional Reading

## References
