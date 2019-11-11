---
title: Domain Event
tags: [DDD]
style: border
color: secondary
description: Implement Domain event.
---

There are many posts that explains very well about the benefit of using domain event. Here are a list of good reading.test

* [Domain events](https://docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/domain-events-design-implementation)
* [Strengthening your domain event](https://lostechies.com/jimmybogard/2010/04/08/strengthening-your-domain-domain-events)
* [Fully encapsulated Domain model](http://udidahan.com/2008/02/29/how-to-create-fully-encapsulated-domain-models/)

I have been looking for how the domain event works and how to implement it. There are a few posts out there and I have tried to create different implementations. This is just one way to do it

## Why do we have to create more code

Here this is my personal opinion. It might take more time and effort for wiring at the beginning but we will see the huge benefit when the project grow in the long term. It encapsulates business logic from the calling client and help to extend the functionalities without modifying the a lot of existing class or module. Refer to the [Open/Close principle](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle)

## Creating an AggregateRoot

The first step is to create a root object. It is usually called AggregateRoot. The job of this class is to manage all the events in the domain. It will need to hold a list of the domain events, adding, removing and executing the events.

```csharp
public abstract class AggregateRoot
{
    private readonly List<IDomainEvent> _domainEvents = new List<IDomainEvent>();
    public virtual IReadOnlyList<IDomainEvent> DomainEvents => _domainEvents;

    protected virtual void AddDomainEvent(IDomainEvent eventToAdd)
    {
        _domainEvents.Add(eventToAdd);
    }

    protected virtual void ClearEvents()
    {
        _domainEvents.Clear();
    }

    public virtual void PerformInvocation()
    {
        foreach (var domainEvent in _domainEvents)
        {
            DomainEvent.DomainEvents.Dispatch(domainEvent);
        }
        ClearEvents();
    }

}
```

`IDomainEvent` is an empty interface and it will become clear once we start to create more classs with name `SomethingChangedEvent` that will implement this empty interface

The PerformInvocation method is the important method that will help to release each event happened in the domain. It runs through each events and call a method `Dispatch(eventToDispatch)` We will create a new class called `DomainEvents` below

## Central class to control all of the events

```csharp
public static class DomainEvents
{
    private static List<Type> _handlers;
    public static void Init()
    {
        _handlers = Assembly.GetExecutingAssembly()
            .GetTypes()
            .Where(x => x.GetInterfaces().Any(y => y.IsGenericType && y.GetGenericTypeDefinition() == typeof(IHandle<>)))
            .ToList();
    }

    public static void Dispatch(IDomainEvent domainEvent)
    {
        foreach (Type handlerType in _handlers)
        {
            bool canHandleEvent = handlerType.GetInterfaces()
                .Any(x => x.IsGenericType
                          && x.GetGenericTypeDefinition() == typeof(IHandle<>)
                          && x.GenericTypeArguments[0] == domainEvent.GetType());

            if (canHandleEvent)
            {
                dynamic handler = Registration.Container.Resolve(handlerType);
                handler.Handle((dynamic)domainEvent);
            }
        }
    }
}
```

## Adding generic method to handle different type of events

This interface contains a single method to execute logic for specific domain entity

```csharp
public interface IHandle<T> where T : IDomainEvent
{
    void Handle(T domainEvent);
}
```

The handle method will be called from the Eventhandler classes. Now we can create 2 related classes: The domain `Event` and the `EventHandler`. It would be a good practice to keep the naming convention so it is easier to track down the issue while doing debugging. For example:

* BalanceChangedEvent
* BalanceChangedEventHandler
* `Event` is the class that will hold all the information needed for our event handler
* `EventHandler` will use event's information to processing business logic

Here is how it looks like:

```csharp
public class BalanceChangedEventHandler : IHandle<BalanceChangedEvent>
{
    private readonly IEmailService _emailService;

    public BalanceChangedEventHandler(IEmailService emailService)
    {
        _emailService = emailService;
    }
    public void Handle(BalanceChangedEvent domainEvent)
    {
        // handle it // add your logic here
        Console.WriteLine($"Balance change amount {domainEvent.Delta}");

        // send email
        _emailService.SendEmailAsync();
        // do others...via services...
    }
}
```

## Consuming the event

At this point we don't need to worry about business logic and what will happen after the user withdraw money from the machine. Let's look at the entity `Machine'

```csharp
public class Machine : AggregateRoot
{
    public virtual void WidthdrawMoney(decimal amount)
    {
        AddDomainEvent(new BalanceChangedEvent(amount));
    }
}

```

This method just adds event and it will be handle by the `BalanceChangedEventHandler` at the later point...

## Raise the event

When it is time, we can raise the events or dispatch them like this:

```csharp
foreach (var domainEvent in entity.DomainEvents)
{
    DomainEvents.Dispatch(domainEvent);
}
```

## Committing before Dispatching

To make sure that we save all nessesary data before executing other actions. I added this repository. Event thought this will serve the purpose pretty well. There is definitely a better way to seperate this 2 different operation since it might feel a bit aweward for dispatching events in the repository class...that would be our next task

```csharp
public class TextBaseRepository<T>  : ILocalRepository where T : AggregateRoot
{
    public async Task Save<T>(T entity) where T:AggregateRoot
    {
        // saving ....
        await Task.Run(() =>{});
        var savingResult = true;
        Console.WriteLine($"Saving successfully...Dispatching events for {typeof(T).Name}");
        if (savingResult)
        {
            foreach (var domainEvent in entity.DomainEvents)
            {
                DomainEvents.Dispatch(domainEvent);
            }
        }
    }
}
```
