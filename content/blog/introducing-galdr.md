---
title: Introducing Galdr
date: 2024-04-23
authors:
  - name: rthomasv3
    link: https://github.com/rthomasv3
    image: https://avatars.githubusercontent.com/u/39268276?v=4
tags:
  - Galdr
  - C#
  - Cross-platform
  - Multi-platform
excludeSearch: false
---

## Introducing Galdr

[Galdr](https://github.com/rthomasv3/Galdr) is a multi-platform desktop framework powered by [webview](https://github.com/webview/webview) and C#. It allows you to create desktop applications for Windows, Mac, and Linux using any front-end framework of your choice with C# powering all your back-end logic.

![GaldrPOC](/images/galdr-poc.png)

<!--more-->

### Features
* Call C# methods asynchronously from javascript/typescript
* Compatible with any frontend framework (Vue, React, etc.)
* Cross-platform (Windows, Linux, macOS)
* Dependency injection
* Hot-reload
* Native file system integration
* Optional Single file executable
* Reasonable binary size (POC is 26MB)


## Background

I'd been looking for a good multi-platform desktop framework for a long time, but I wasn't very pleased with any of the options. I tried many different languages (Vala, C#, C++, Rust, and more) and many different frameworks (GTK, MAUI, Avalonia, Qt, Tauri, and others), but every option had a significant downside or a flat out dealbreaker for the projects I had in mind.

The framework I ended up enjoying the most was [Tauri](https://tauri.app/). It let me create just about anything I wanted, and I was able to use any web frontend framework. It also resulted in small binaries and had a very consistent look and feel across each platform. The only downside for me is the backend is in Rust. I've been working on learning Rust and really enjoy the language, but I wanted to be able to make fast progress on some of my side-projects, and learning a new language is never fast.

However, Tauri gave me a great idea for how I could solve my all of my issues. I would just make Tauri, but with C# as the backend instead of Rust.


## How It Works

Galdr uses a combination of native bindings to `webview` along with an ASP.NET Core [WebApplication](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.webapplication?view=aspnetcore-8.0) that runs in the background and serves the UI from embedded content. It can also listen on `localhost`, allowing for hot-reload.

Galdr also collects a series of commands to make available on the front-end. It does this by looking for classes tagged with the `[Commands]` atrribute or methods tagged with `[Command]`. It then registers a function named `galdrInvoke` with webview. This makes the method availabled anywhere in the JavaScirpt on your front end.

```cs
[Command]
public static string Greet(string name)
{
    return $"Hello, {name}! You've been greeted from C#!";
}
```

```js
greetMsg.value = await galdrInvoke("greet", { name: name.value });
```

## Projects Using Galdr

I have a few projects in the works, but the one that is currently published and open-source is called [Scum Bag](https://github.com/rthomasv3/Scum-Bag). It's an automatic save scummer and manager that's been working great on both Windows and Linux. It was created with [Galdr](https://www.nuget.org/packages/Galdr/), [Vue](https://vuejs.org/), and [PrimeVue](https://primevue.org/).

I have a few more projects in the works, but those aren't quite ready to show off yet.

![Scum Bag](/images/scum_bag_er_3.png)


## Conclusion

For me Galdr solves all the problems I've had with other multi-platform desktop frameworks and has resulted in a pretty big increase in productivity on my side-projects. It allows me to easily develop desktop applications with the option of quickly converting them over to a website in the future. Best of all, I get to use the language I'm most familiar with and it just so happens to be a great one for back-ends.

You can find Galdr on [NuGet](https://www.nuget.org/packages/Galdr/) and small [POC](https://github.com/rthomasv3/GaldrPOC) on my GitHub that you can use as a template.
