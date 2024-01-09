---
title: The Importance of Naming Conventions
date: 2021-04-03
authors:
  - name: rthomasv3
    link: https://github.com/rthomasv3
    image: https://avatars.githubusercontent.com/u/39268276?v=4
tags:
  - Naming
excludeSearch: false
---

## Intro

Have you ever been completely lost in a codebase? Whether it's legacy code you're trying to maintain or you just started a new job, it can be extremely overwhelming to navigate the depths of someone else's creation. What can be most frustrating is there's a pretty straightforward solution to this problem.

As the title totally spoiled, the answer is code conventions. Specifically, I'd like to talk about naming conventions and less about formatting or design.<!--more-->

## Naming Consistency

Naming is one of the areas I see neglected most often. I've seen pristine code with near perfect formatting and logic, but it was ridiculously hard to read due to poor variable, class, and namespace names. To make matters worse, there was very little consistency. When I ran into an `XService` or a `YManager` within the code, the purpose of the class wasn't clear and I wasn't sure if there was a difference. When a new team member asks, anyone on the team should be able to answer because it should be part of the code conventions.

Consistency is one of the biggest pieces of the puzzle. I find it's often way more important than what the actual convention is. For example, some people prefer one return statement at the end of a method with a return variable prefixed with ‘r'. Is this the best convention? I don't personally prefer it. But teaching it to someone is super easy and the code immediately becomes more readable to them.

## Class Names

This simple idea can be taken one step further by applying it to class names. Does the class implement a certain design pattern? Append the pattern name to the class name to make it immediately identifiable. This simple step tells someone quite a bit about the code and they didn't even have to search through it.

For example, what can you tell me about a `ProductRepository` class? I would hope it implements the repository pattern and is used to manage product data.

## Layers

Another practice I've found helpful is to base class names and namespace names off of the architectural layer in which they live. For namespaces this may seem a bit obvious, but you'd be surprised how often it's ignored.

For class names, you can add a simple suffix like `ViewModel` or `Entity` to tell people about the class's purpose. Anything that's a `ViewModel` would obviously live on the view model layer and is bound to a UI in some way. Any `Entity` might refer to a class in your data access layer that works with entity framework.

The biggest benefit I've found in consistent naming is how easy it makes training new people. I can sit down with them for five minutes and explain that a `Service` is reserved for logic within the business layer or a `Document` class represents serializable objects on the data access layer. I've found that this really helps new people get up to speed quickly and hit the ground running.

## What About Existing Codebases?

Existing or older projects can be the wild west, but there's no need to do a complete refactor all at once. Talk with your team and decide on some good conventions. Then follow them as you create new code. Stay consistent. Any time you work in pre-existing code, try to do some small updates in the area you're working to help it better fit the conventions the team decided on. The code will be significantly better before you know it.

An argument I've heard against all of this is that it contributes to making developers replaceable. Many have referred to their awful code as "job security". As you may have guessed, I've never found that argument to be very convincing. At the end of the day, you're the one who has to work in the code and maintain it. Don't make your own day-to-day life harder!

After you've worked in truly immaculate code you'll never want to go back.