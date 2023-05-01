# Legendary Tools - Service Locator

Provide a global point of access to a service without coupling users to the concrete class that implements it.

[TOC]

### Motivation

Some objects or systems in a game tend to get around, visiting almost every corner of the codebase. It’s hard to find a part of the game that won’t need a memory allocator, logging, or random numbers at some point. Systems like those can be thought of as services that need to be available to the entire game.

### The Pattern

A service class defines an abstract interface to a set of operations. A concrete service provider implements this interface. A separate service locator provides access to the service by finding an appropriate provider while hiding both the provider’s concrete type and the process used to locate it.

### When to Use It

Anytime you make something accessible to every part of your program, you’re asking for trouble. That’s the main problem with the Singleton pattern, and this pattern is no different. My simplest advice for when to use a service locator is: sparingly.

Instead of using a global mechanism to give some code access to an object it needs, first consider passing the object to it instead. That’s dead simple, and it makes the coupling completely obvious. That will cover most of your needs.

But… there are some times when manually passing around an object is gratuitous or actively makes code harder to read. Some systems, like logging or memory management, shouldn’t be part of a module’s public API. The parameters to your rendering code should have to do with rendering, not stuff like logging.

Likewise, other systems represent facilities that are fundamentally singular in nature. Your game probably only has one audio device or display system that it can talk to. It is an ambient property of the environment, so plumbing it through ten layers of methods just so one deeply nested call can get to it is adding needless complexity to your code.

In those kinds of cases, this pattern can help. As we’ll see, it functions as a more flexible, more configurable cousin of the Singleton pattern. When used well, it can make your codebase more flexible with little runtime cost.

### Keep in Mind

The core difficulty with a service locator is that it takes a dependency — a bit of coupling between two pieces of code — and defers wiring it up until runtime. This gives you flexibility, but the price you pay is that it’s harder to understand what your dependencies are by reading the code.

#### The service actually has to be located

With a singleton or a static class, there’s no chance for the instance we need to not be available. Calling code can take for granted that it’s there. But since this pattern has to locate the service, we may need to handle cases where that fails. Fortunately, we’ll cover a strategy later to address this and guarantee that we’ll always get some service when you need it.

#### The service doesnt know who is locating it

Since the locator is globally accessible, any code in the game could be requesting a service and then poking at it. This means that the service must be able to work correctly in any circumstance. For example, a class that expects to be used only during the simulation portion of the game loop and not during rendering may not work as a service — it wouldn’t be able to ensure that it’s being used at the right time. So, if a class expects to be used only in a certain context, it’s safest to avoid exposing it to the entire world with this pattern.

### How to Use

#### Registering services

To register a service simply use the ServiceLocator.Register() API, as follows:

```csharp
    public interface IAudioSystem : IDisposable
    { }

    public class AudioSystem : IAudioSystem
    {
        public void Dispose()
        { }
    }

    public class SampleCode : MonoBehaviour
    {
        private void Start()
        {
            IAudioSystem audioSystem = new AudioSystem();
            ServiceLocator.Register<IAudioSystem>(audioSystem);
        }
    }
```

Note: To ensure that services follow SOLID principles, only *interfaces* can be registered in ServiceLocator.

#### Getting services

```csharp
    public class SampleCode : MonoBehaviour
    {
        public IAudioSystem GetAudioService()
        {
            return ServiceLocator.GetService<IAudioSystem>();
        }
    }
```
#### Notes
- It is not possible to register duplicate services of the same type, only one type must exist in the service locator.
- If you want to replace an instance of a service, remove the service and then add it again.
