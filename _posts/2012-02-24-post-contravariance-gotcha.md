---
title: "Type.IsAssignableFrom() and a contravariance gotcha"
excerpt: "I wanted to create an extension method on the class System.Type that would take another Type and return true if it is an interface implemented by the first Type. I thought this would be quick and easy. What I implemented did work most of the time. However I ran into a situation in which it returned an unexpected result, and it took me some time to figure out what was going on."
modified: 2012-02-24
header:
  teaser: 
tags: 
  - csharp
  - types
  - interfaces
---

I wanted to create an extension method on the class System.Type that would take another Type and return true if it is an interface implemented by the first Type. I thought this would be quick and easy. What I implemented did work most of the time. However I ran into a situation in which it returned an unexpected result, and it took me some time to figure out what was going on.

```csharp
public static bool ImplementsInterface(this Type objectType, Type interfaceType) 
{ 
    if (!interfaceType.IsInterface)
        throw new ArgumentException("interfaceType is not an interface");
    return interfaceType.IsAssignableFrom(objectType);
}
```

It seems simple enough. So why doesn't it always behave? Say we have a Manager class that inherits Person, and we want to compare two Person references using an EqualityComparer. And, for the sake of argument, we have a myComparer whose type we are unsure of. So we need to check if it can compare Person objects...

```csharp
bool isValidComparer = myComparer.GetType()
    .ImplementsInterface(typeof(IEqualityComparer<Person>));
```

If myComparer were a EqualityComparer&lt;Manager&gt; I would expect the above code to return true, because IEqualityComparer&lt;T&gt; is contravariant for T. (ie: it should accept anything of type T or less derived.) But it returns false. It seems that IsAssignableFrom returns false when executing it on a concrete type against a contravariant interface type. I don't understand why this is the case, and it's not important enough for me to go digging into the CLR specifications. But if anyone wants to enlighten me, I'm all ears. 

My new implementation addresses this issue by only calling IsAssignableFrom on interface types. If the given objectType is not an interface, it goes through all its implemented interfaces to see if at least one implements interfaceType... 

```csharp
public static bool ImplementsInterface(this Type objectType, Type interfaceType) 
{ 
    if (!interfaceType.IsInterface) 
        throw new ArgumentException("interfaceType is not an interface"); 
    if (objectType.IsInterface) 
        return objectType.IsAssignableFrom(interfaceType); 
    foreach (var checkInterfaceType in objectType.GetInterfaces()) 
    { 
        if (checkInterfaceType.IsAssignableFrom(interfaceType)) 
            return true; 
    } 
    return false; 
}
```
