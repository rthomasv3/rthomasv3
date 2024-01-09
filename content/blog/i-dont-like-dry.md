---
title: I Don't Like DRY
date: 2021-12-18
authors:
  - name: rthomasv3
    link: https://github.com/rthomasv3
    image: https://avatars.githubusercontent.com/u/39268276?v=4
tags:
  - DRY
excludeSearch: false
---

DRY (Don't Repeat Yourself) is a programming principle intended to prevent code duplication, thus preventing a developer from doing the same work multiple times (both in initial development and when updating code).

While this is generally a really good idea and can greatly improve quality and maintainability of software, it's much too simple a phrase to accurately explain a fairly nuanced idea.
<!--more-->

A better, but still too ambiguous phrasing is:
> "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system."

So when I say I don't like DRY, it's more the _wording_ that I don't like. It seems so simple - just don't repeat your code. So instead of copying and pasting the same code around your program, just make one method or class and use it everywhere instead.

The code is better now, right? Well, maybe...

It depends on what purpose the code was serving and how it was made reusable. I've seen many `Utility` and `Helper` classes that eliminated some small duplication, but introduced a myriad of other problems. Was the code just tossed in a static class somewhere? How will that effect our ability to unit test the code using it? Did it really belong in that `Helper` and will I be able to find it there later?

The other issue is in how the code is being used. I've seen a lot of people try to make things overly generic in the pursuit of zero duplication, introducing strange inheritance hierarchies where a country and a product share the same base `NamedEntity` class, because God forbid they both have their own `Name` property - too much duplication!

What many people don't see is that sometimes a little duplication can actually save you a lot of headaches in the future. This is especially true in the case of classes. Just because a class looks the same doesn't mean it's bad duplication. Let's look at a simple example.

Here we have a class used to store information about a book:

```cs
public class BookEntity
{
   public string Title { get; set; }
   public string Series { get; set; }
   public string Author { get; set; }
   public string Description { get; set;}
   public string ISBN { get; set; }
   public string Publisher { get; set; }
   public DateTime PublicationDate { get; set; }
   public string Edition { get; set; }
   public int PageCount { get; set; }
}
```

And here we have a class used return information about a book from a REST API:

```cs
public class BookApiResponse
{
   public string Title { get; set; }
   public string Series { get; set; }
   public string Author { get; set; }
   public string Description { get; set;}
   public string ISBN { get; set; }
   public string Publisher { get; set; }
   public DateTime PublicationDate { get; set; }
   public string Edition { get; set; }
   public int PageCount { get; set; }
}
```

Sure, this is a really simple example, but I've seen a lot of cases where what the API returned looked almost exactly like the data table (especially in the early iterations of a piece of software).

DRY would lead me to think these two classes are the same: they're duplication, right? So I should delete one and just reuse the other?

__No__.

While DRY seems like it should apply, if followed obediently and the repeat code remove, you'd actually have a significantly worse piece of software.

Say you want to make some changes to the way a book is stored. You notice that properties like `Author` and `Publisher` are repeated and would make more sense in their own table/document. Now you have new classes to represent those properties in your book class:

```cs
public class BookEntity
{
   ...
   public AuthorEntity Author { get; set; }
   ...
   public PublisherEntity Publisher { get; set; }
   ...
}
```

The problem is, if you had reused this class to return book information from the API, you just introduced breaking changes to any clients using your API. By removing duplication, you, by definition, increased coupling.

Even though the classes looked the same, they weren't duplication because they served a very different purpose. One class is used to model book information in a way that's ideal for storage. The other is used to define your API contract and may be geared more toward displaying the book to a user.

There's a better principle that should have been applied here - [single-responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle). Ask yourself, what are some likely scenarios where I would want to make changes to the class? If you came up with several unrelated reasons, you're likely to run into problems when the time inevitably comes to do maintenance or enhancements.

Instead of always thinking about DRY, other design principles like [SOLID](https://en.wikipedia.org/wiki/SOLID) should be considered first. Then duplication should be considered and code consolidated only when it will be used for the same purpose in multiple places - in other words, if you'll have to update all of it later for the same reason.

So this is why I don't like DRY - if you take it literally, you'll get it dead wrong: it's the misunderstandings it causes and the disastrous manifestations of those misunderstandings.
