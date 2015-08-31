---
layout: post
title: "Perspex is Now in Alpha"
excerpt: "The first alpha release has just been released. Come celebrate here."
category: Perspex
tags: [perspex, c#, .net]
comments: true
---

I'm pleased to announce that [Perspex](https://github.com/grokys/Perspex/) is now
in alpha!

Perspex is a multi-platform windowing toolkit - somewhat like WPF - that is
intended to be multi-platform (more about that below). It supports XAML,
lookless controls and a flexible styling system, and runs on Windows using
Direct2D and other operating systems using Gtk & Cairo.

So what does "alpha mean? Well, it means that it's now at a stage where you
can have a play and hopefully create simple applications. There's now a [Visual
Studio Extension](https://visualstudiogallery.msdn.microsoft.com/87db356c-cec9-4a07-b7db-a4ed8a921ac9)
containing project and item templates that will help you get started, and
there's an initial complement of controls. There's still a lot missing, and you
*will* find bugs, and the API *will* change, but this represents the first time
where we've made it somewhat easy to have a play and experiment with the
framework.

# How do I try it out

The easiest way to try out Perspex is to install the [Visual Studio Extension](https://visualstudiogallery.msdn.microsoft.com/87db356c-cec9-4a07-b7db-a4ed8a921ac9).

This will add a Perspex project template and a Window template to the standard
Visual Studo "Add" dialog (yes, icons still to come :) ):

![Add Dialogs]({{ site.url }}/images/2015-08-31-perspex-alpha/add-dialogs.png)

Creating a Perspex Project will give you a simple project with a single XAML
window. There's currently no designer, and not even any type-checking or
intellisense for Perspex's xaml, but it works when you press F5, which is the
important part!

![Add Dialogs]({{ site.url }}/images/2015-08-31-perspex-alpha/hello-world-xaml.png)

There's also a Perspex Window template in there, but no User Control template
yet (in reality you just need to change `Window` to `UserControl`).

# Multi-platform you say?

Well, yes, that is the intention. However unfortunately as of the time of this
first alpha, Perspex is only shipping with a Windows backend. There *is* a
Gtk/Cairo backend that's working pretty well (at least on Windows) but it's not
included in this release due to packaging issues. In addition, the framework did
work on Linux at one point but with the recent Mono 4.0 something has gone
wrong, and we need time to work out what that is. Getting Perspex working again
on non-windows support is the next thing we'll be concentrating on. You can
track the progress on Linux in the [issue on GitHub](https://github.com/grokys/Perspex/issues/78).

Even though it's not quite there yet, the reason it's a close as it is is due to
the efforts of [ncarillo](https://github.com/ncarrillo) - thanks Nelson!

# OmniXAML

Perspex's XAML support has only just landed and its only thanks to the great
[OmniXaml](https://github.com/SuperJMN/OmniXAML) project that it's at all
possible. OmniXAML makes it possible to consume XAML from PCLs and best of all,
the XAML support is *completely* decoupled from the main Perspex library,
due to OmniXaml's high degree of configurability meaning that Perspex could well
support other types of markup in future. It's still early days, but check out
OmniXAML now! Thanks [@José ](https://github.com/SuperJMN)!
