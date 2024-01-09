---
title: SOLID
date: 2023-10-30
authors:
  - name: rthomasv3
    link: https://github.com/rthomasv3
    image: https://avatars.githubusercontent.com/u/39268276?v=4
tags:
  - SOLID
  - OO
excludeSearch: false
type: docs
weight: 1
---

## Summary
SOLID is a set of object-oriented design principles that can help make code significantly more stable, readable, and maintainable.

* S - Single-responsiblity Principle
    * There should never be more than one reason for a class or method to change.
* O - Open-closed Principle
    * Software entities should be open for extension, but closed for modification.
* L - Liskov Substitution Principle
    * Methods that use references to base classes must be able to use objects of derived classes without knowing it.
* I - Interface Segregation Principle
    * Clients should not be forced to depend upon interfaces that they do not use.
* D - Dependency Inversion Principle
    * Depend upon abstractions, not concretions.

<br>

## Single-responsiblity Principle

> <br>
> There should never be more than one reason for a class or method to change.
> <br><br>

While this statement is a bit vague, by "reason ... to change", it basically means that there shouldn't be multiple very different enhancements that could cause you to alter parts of the exact same module.

For example, say you have a method that validates and saves a user to your system. This method could have multiple reasons to change. Validation may have started by making sure a user with the same email doesn't already exist in the system, but later you need to make sure the organization the user is being added to is still active. Saving the user might need to include new columns or be updated to a use different table structure. The different reason to change here make these two different responsibilities, and therefore the code should live in two different modules or classes.

Having this code in two different places reduces risk and increases testability. If they were kept in the same method, the more change there is to the user validation process, the greater the risk of breaking the code that writes the user to the database. Keeping these things separate modules also greatly simplifies testing by reducing the number and type of things in both your 'arrange' and 'assert' steps. This is one way unit testing can inform good design.

That isn't to say you can't have code that calls both the `ValidateUser` and `SaveUser` method. Simple code like this would only have one reason to change - the steps required to add a user to the system changed.

### Example Code

#### Bad
```cs
class UserUtility
{
    private readonly AccountEfContext _context;

    public UserUtility(AccountEfContext context)
    {
        _context = context ?? throw new ArgumentNullException(nameof(context));
    }

    public async Task<bool> AddUser(Guid orgId, string email, string passwordHash)
    {
        bool userAdded = false;

        if (orgId != Guid.Empty && !String.IsNullOrWhiteSpace(email) && !String.IsNullOrWhiteSpace(passwordHash))
        {
            // select count(1) from Accounts where Id = @orgId
            int matchingAccounts = await _context.Accounts
                .Where(account => account.Id == orgId)
                .CountAsync();

            if (matchingAccounts == 1)
            {
                string trimmedEmail = email.Trim();

                if (MailAddress.TryCreate(trimmedEmail, out MailAddress mailAddress))
                {
                    int matchingUsers = await _context.Users
                        .Where(user => user.Email == mailAddress.Address)
                        .CountAsync();

                    if (matchingUsers == 0)
                    {
                        _context.Users.Add(new User()
                        {
                            AccountId = orgId,
                            Email = mailAddress.Address,
                            Password = passwordHash
                        });

                        int rowsAdded = await _context.SaveChangesAsync();

                        if (rowsAdded == 1)
                        {
                            userAdded = true;
                        }
                    }
                }
            }
        }

        return userAdded;
    }
}
```

#### Good
```cs
interface IUserValidator
{
    bool UserDataIsValid(Guid orgId, string email, string passwordHash);
}

...

interface IUserRepository
{
    Task<bool> AddUser(Guid orgId, string email, string passwordHash);
}

...

class UserService
{
    private readonly IUserValidator _userValidator;
    private readonly IUserRepository _userRepository;

    public UserService(IUserValidator userValidator, IUserRepository userRepository)
    {
        _userValidator = userValidator ?? throw new ArgumentNullException(nameof(userValidator));
        _userRepository = userRepository ?? throw new ArgumentNullException(nameof(userRepository));
    }

    public async Task<bool> AddUser(Guid orgId, string email, string passwordHash)
    {
        bool userAdded = false;

        if (_userValidator.UserDataIsValid(orgId, email, passwordHash))
        {
            userAdded = await _userRepository.AddUser(orgId, email, passwordHash);
        }

        return userAdded;
    }
}
```

<br>

## Open-closed Principle

> <br>
> Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.
> <br><br>

The goal of open closed principle is to keep code as stable as possible. The simplest way to do this is to avoid making changes to existing code. However, we still need to be able to add new features, so how can we do both?

### Example Code

A simple example exists in a responders system I helped design at a previous job. When we received a text message we needed to perform certain actions. This might include adding the mobile number to a contact list, assigning the message to a given agent, or any number of other things. 

This is roughly how the first iteration of responders accomplished this goal:

```cs
foreach (ResponderAction action in responder.ResponderActions)
{
    switch (action.ResponderActionType)
    {
        case ResponderAction.OptIn:
            ignoreOptStatus = await _responderExecutorService.UpdateOptStatus(customerLineId, externalPhoneNumber, false);
            break;

        case ResponderAction.OptOut:
            ignoreOptStatus = await _responderExecutorService.UpdateOptStatus(customerLineId, externalPhoneNumber, true);
            break;

        case ResponderAction.AddToContactList:
            await _responderExecutorService.AddToContactList(accountId, externalPhoneNumber, action);
            break;

        case ResponderAction.AssignToAgent:
            await _responderExecutorService.AssignToAgent(conversationId, externalPhoneNumber, action);
            break;

        case ResponderAction.InitiateChatBot:
            botSessionInitiated = await _responderExecutorService.InitiateChatBot(accountId, externalPhoneNumber, customerLineId, conversationId, messageBody, action);
            break;
    }
}
```

While this code works just fine and is relatively simple, it will need to be updated every time a new action type is added. An alternative design that respects open-closed principle is to create an `IResponderAction` abstraction that each action type would then need to implement. This abstraction defines the extension point. Then, when new action types are added, the execution code won't need to be modified (i.e. use the command pattern).

```cs
interface IResponderActionExecutor
{
    Task<ResponderExecutionResult> Execute(ResponderEventArguments args);
}

---

class InitiateChatBotResponderAction : IResponderActionExecutor
{
    public InitiateChatBotResponderAction(/** whatever dependencies **/)
    {
    }

    public Task<ResponderExecutionResult> Execute(ResponderEventArguments args)
    {
        // start a chat bot session
    }
}

---

interface IResponderActionFactory
{
    IResponderActionExecutor GetResponderActionExecutor(ResponderActionType responderActionType);
}

---

foreach (ResponderAction action in responder.ResponderActions)
{
    IResponderActionExecutor actionExecutor = _actionFactory.GetResponderActionExecutor(action.ResponderActionType);

    if (actionExecutor != null)
    {
        ResponderExecutionResult result = await responderAction.Execute(args);
    }
}
```

Each responder action can be created using a dependency injection system, or with a factory if needed. Either way your business logic doesn't need to change when you add new responder action types.

<br>

## Liskov Substitution Principle

> <br>
> Methods that use references to base classes must be able to use objects of derived classes without knowing it.
> <br><br>

There can be many different breaking changes introduced by inheritance; method overrides that provide different behavior, methods that aren't required by the derived class, and methods that require additional knowledge to use properly. All of these are violations of Liskov Substitution Principle and indicate your inheritance is likely to cause bugs.

### Example Code

An example where additional knowledge would be required to use the derived class properly.

```cs
class Rectangle
{
    private int _width;
    private int _height;

    public Rectangle(int width, int height)
    {
        _width = width;
        _height = height;
    }

    public void SetWith(int width)
    {
        _width = width;
    }

    public void SetHeight(int height)
    {
        _height = height;
    }
}

// violates Liskov Substitution Principle
class Square : Rectangle
{
    public Square(int width, int height)
        : base(width, height)
    {
    }
}
```

<br>

## Interface Segregation Principle

> <br>
> Clients should not be forced to depend upon interfaces that they do not use.
> <br><br>

The biggest red flag to indicate a violation of interface segregation principle is a `NotImplementedException` or a `NotSupportedException`. If you don't need to implement one of the methods defined in the abstraction, you don't actually meet the requirements of that abstraction and should make a new one.

### Example Code

#### Bad
```cs
interface ICoffeeMachine
{
    void AddCoffee(string coffeeType);
    void BrewFilteredCoffee();
    void BrewEspresso();
}

class BasicCoffeeMachine : ICoffeeMachine
{
    public void AddCoffee(string coffeeType)
    {
        // do things to add coffe to the machine
    }
    public void BrewFilteredCoffee()
    {
        // do things start brewing coffee
    }

    public void BrewEspresso()
    {
        throw new NotSupportedException("I can't brew espresso.");
    }
}

class EspressoMachine : ICoffeeMachine
{
    public void AddCoffee(string coffeeType)
    {
        // do things to add coffe to the machine
    }
    public void BrewFilteredCoffee()
    {
        throw new NotSupportedException("I can't brew filtered coffee.");
    }

    public void BrewEspresso()
    {
        // do things to start brewing espresso
    }
}
```

#### Good
```cs
interface ICoffeeMachine
{
    void AddCoffee(string coffeeType);
}

interface IFilteredCoffeeMachine : ICoffeeMachine
{
    void BrewFilteredCoffee();
}

interface IEspressoMachine : ICoffeeMachine
{
    void BrewEspresso();
}

class BasicCoffeeMachine : IFilteredCoffeeMachine
{
    public void AddCoffee(string coffeeType)
    {
        // do things to add coffe to the machine
    }

    public void BrewFilteredCoffee()
    {
        // do things start brewing coffee
    }
}

class EspressoMachine : IEspressoMachine
{
    public void AddCoffee(string coffeeType)
    {
        // do things to add coffe to the machine
    }

    public void BrewEspresso()
    {
        // do things to start brewing espresso
    }
}
```

<br>

## Dependency Inversion Principle

> <br>
> Depend upon abstractions, not concretions.
> <br><br>

When designing the interaction between a high-level module and a low-level one, the interaction should be thought of as an abstract interaction between them. This not only has implications on the design of the high-level module, but also on the low-level one; the low-level one should be designed with the interaction in mind, and it may be necessary to change its usage interface.

Put another way, you need to keep in mind the nature of the interaction between the two modules in order to create a sufficient abstraction, which should naturally lead to looser coupling. Just having an interface doesn't necessarily mean coupling will be loose - lots of parameters, side effects, strict order of operations, and more can lead to code being co-dependent.

Dependency inversion allows for code to be loosely coupled without the use of additional design patterns, which can sometimes add complexity and make code harder to follow. You can also pair the idea with something like dependency injection, leading to even more flexibility throughout a system.

### Example Code

A simple example is a `UserPermissionsService` that wants to make use of a cache to improve performance.

```cs
class CacheValue<T>
{
    public T Value { get; init; }
    public bool HasValue { get; init; }
    public DateTime Expiration { get; init; }
}

---

interface ICacheService
{
    CacheValue<T> Get<T>(string key);
    bool Set<T>(string key, T value, TimeSpan? lifetime = null);
}

---

class UserPermissionsService
{
    private readonly IUserPermissionsRepository _userPermissionsRepo;
    private readonly ICacheService _cacheService;

    public UserPermissionsService(IUserPermissionsRepository userPermissionsRepo, ICacheService cacheService)
    {
        _userPermissionsRepo = userPermissionsRepo;
        _cacheService = cacheService;
    }

    public async Task<UserPermissions> GetUserPermissions(string email)
    {
        UserPermissions userPermissions = null;

        CacheValue<UserPermissions> cachedUserPermissions = _cacheService.Get<UserPermissions>(email);

        if (cachedUserPermissions.HasValue)
        {
            userPermissions = cachedUserPermissions.Value;
        }
        else
        {
            userPermissions = await _userPermissionsRepo.GetUserPermissions(email);

            if (userPermissions != null)
            {
                _cacheService.Set(email, userPermissions);
            }
        }

        return userPermissions;
    }
}
```

The `UserPermissionsService` doesn't need to know anything about the cache service or how it's implemented.

```cs
class MemoryCacheService : ICacheService
{
    public CacheValue<T> Get<T>(string key)
    {
        // check a local dictionary for the key
    }

    public bool Set<T>(string key, T value, TimeSpan? lifetime = null)
    {
        // set the value in the local dictionary
    }
}

---

class RedisCacheService : ICacheService
{
    public CacheValue<T> Get<T>(string key)
    {
        // reach out to redis service and look for the given key
    }

    public bool Set<T>(string key, T value, TimeSpan? lifetime = null)
    {
        // reach out to redis service and set the value
    }
}

```

Using dependency injection, it's very easy to choose which concrete implementation is used without breaking or changing the `UserPermissionsService` in any way.

```cs
services.AddSingleton<ICacheService, MemoryCacheService>();

// vs

services.AddSingleton<ICacheService, RedisCacheService>();
```

Another benefit is that you can now write and test a new cache service while keeping it isolated from the rest of the system. This helps with continuous integration - you can make progress toward a goal and merge daily without conflicts. Once the new service is fully tested and ready, just change one line of code to upgrade the whole system.

<br>

## How to Write SOLID Code
1. Write a rough draft of the code that "works".
2. Review the code and look for violations of SOLID principles one at a time.
3. For each, if you find an issue:
    * Do you know any design patterns that will help?
        + Don't be afraid to look some up - no one has them all memorized.
    * Do you have any libraries available to you that will help?
    * Are there similar situations in other parts of the code base? How was it solved there?
3. Unit tests can be very helpful in informing you of design issues. The [code in the single responsibility section](#example-code) is a good example.
