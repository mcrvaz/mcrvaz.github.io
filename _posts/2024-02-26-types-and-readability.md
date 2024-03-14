---
title: "Types and Readability"
excerpt: "Make your intent explicit with the correct types!"
date: 2024-02-19 00:00:00 -03:00
header: 
  teaser: "/assets/images/posts/2024-02-26-types-and-readability/typesystem.png"
classes: wide
categories:
  - blog
tags:
  - unity
  - c#
  - readability
  - quick-tips
---

Today we'll be exploring how to make your code more readable and maintainable by using more adequate types. 

Let's talk a bit about them.

## Hello World
You've probably written a "Hello world!" program in a few languages, right? Some are a bit more verbose, some less. 

Here's how it's done in Python:
```python
print("Hello world!")
```
And this is C#:
```csharp
public class Program
{
	public static void Main()
	{
		System.Console.WriteLine("Hello world!");
	}
}
```

The Python version is obviously shorter, but it's doing a bit less. In the C# version we are declaring a class, a static function inside this class, we are defining their access modifiers, and also accessing the function `WriteLine` inside the `Console` class that lives in the `System` namespace, and then printing the message. Quite a lot going on!

You might ask yourself, "why would I want to do all that just to print "Hello world!"? Surely this is unnecessary." — And it is! But most of the time you don't simply want to print "Hello world!", you want a fully functional program that does a bunch of things, and this additional information helps you achieve that. How so, you ask?

The additional information are mostly types (`Program`, `Main`, `Console`) and access modifiers (`public`) and they help you define rules for interaction between objects. Those rules are enforced by the compiler, which is a good thing! Besides we can double down on this and use the type system to help us read and understand the code more easily.

I've prepared a few examples of how using more suitable types can make your code more readable and maintainable, let's check them out. 

## Enums
Have a quick look at the code below:
```csharp
int GetResponseCode();
string GetResponseCodeDescription(int responseCode);
```
Seems pretty straightforward, but it raises a few questions:

What does the response value mean? Are those some kind of specific response codes? Are they custom?
And about the `GetResponseCodeDescription`, what are the valid values for this function? Can the `responseCode` be negative?

You could amend this a bit by adding comments specifying all that, but it's not a great solution.

After looking around the codebase for a bit we found out that the `responseCode` is actually just an [HTTP response status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status).
And now we can refactor it to use a custom enum, `HttpResponseCode`.
```csharp
HttpResponseCode GetResponseCode();
string GetResponseCodeDescription(HttpResponseCode responseCode);
```
With this new code all of our previous questions are answered by simply looking at the function's definitions. We know that they aren't custom values and we know all of the valid values by looking at the enum definition. And this is the result of, basically, changing the `int` type to a more appropriate one, `HttpResponseCode`.

## Common types
Some types are inherently linked with some operations, like `Quaternions` and rotations in Unity. Consider the following code:
```csharp
Vector3 GetTurnRate();
```
What's the meaning of this `Vector3` returned by the function?

The first and most obvious guess here is that it contains the Euler angles, but do we really need to guess? Why not just return a `Quaterion`?
```csharp
Quaternion GetTurnRate();
```
Now it's crystal clear that the function returns a rotation, and you can easily grab the Euler angles, if necessary.

What about time intervals?
```csharp
double GetTimeSinceStartup();

void InvokeRepeating(Action fn, double interval);
```
Is this `double` value representing seconds? ticks? hours? Who knows.

But we can make it obvious by changing the type to `TimeSpan`.
```csharp
TimeSpan GetTimeSinceStartup();

void InvokeRepeating(Action fn, TimeSpan interval);
```

When you see an unusual type being used, like in the `Vector3` and `Quaternion` example, you might even wonder if there's a special reason for that or if it was just overlooked. Be aware of the usual types used in the codebase and stick to them, unless there's a good reason not to.

## Nullables
Nullable types are a great way of adding more information to your function signatures.

Check this example:
```csharp
int GetSelectedIndex();
```
You can safely guess that it will return the index for something that's selected, whatever it is. But what if nothing is selected, what's going to happen? Is it throwing an exception? Returning -1?

What if we changed the return type to `int?`?
```csharp
int? GetSelectedIndex();
```
Now it's a pretty good guess that if nothing is selected, it will return `null`.

If you're using [Nullable Reference Types](https://learn.microsoft.com/en-us/dotnet/csharp/nullable-references), you can go one step further and also apply this same logic to reference types.

## Collections
The same applies to collections. By using the most appropriate collection type, you'll be making your code easier to understand and probably more performant.
Consider the following scenario, you have a deck of cards and you always want to grab the top card. What's the best implementation for it?
```csharp
// using a list?
List<Card> deck;
// using a stack?
Stack<Card> deck;
```
Spoiler: It's the `Stack`. But it also depends.

The data structure helps you enforce your business logic here (always grab the top card). By using a `List` you open up the possibility to grab cards from the bottom or middle of the deck, and that might not be desirable. What matters here is intent.

Let's see an additional example:
```csharp
class QuestManager 
{
    public List<Quest> GetActiveQuests() 
    { 
        // ...
    }
}
```
Consider that this function returns a reference to the list that holds the active quests. You probably don't want to mess around with it; not add, remove or reorder anything, since the `QuestManager` won't know about those modifications. In this case it would be probably better to use an `IReadOnlyList`:
```csharp
class QuestManager 
{
    public IReadOnlyList<Quest> GetActiveQuests() 
    { 
        // ...
    }
}
```
Now your intent is explicit. This collection is just for reading purposes, you shouldn't modify it. Another cool thing here is that `List` already implements `IReadOnlyList`, so it's even easier to refactor.
 
Notice that this isn't going to make it impossible to modify the collection, since it can be cast back to `List` — This is purely semantic.

## Conclusion
Good APIs are the ones that don't [surprise](https://en.wikipedia.org/wiki/Principle_of_least_astonishment) the end user, and using appropriate types are one of the ways to ensure that. 

Unfortunately, readability and maintainability doesn't always come for free. In some of the examples above, you might have noticed that some of them might be detrimental to your program's performance, for instance, iterating over a `IReadOnlyList` with a `foreach` loop generates temporary allocations. Wrapping every time interval in a `TimeSpan` might also be excessive. Being aware of those trade-offs is key to developing quality software.
