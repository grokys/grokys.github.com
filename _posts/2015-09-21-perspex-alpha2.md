---
layout: post
title: "Perspex Alpha 2"
excerpt: "Alpha 2 of Perspex is now out."
category: Perspex
tags: [perspex, c#, .net]
comments: true
---

We're pleased to announce that alpha 2 of
[Perspex](https://github.com/grokys/Perspex/) is now available.

Perspex is a cross platform .NET UI framework inspired by WPF, with XAML, data
binding, lookless controls and much more. Take a look at the video to see our
current progress:

[![Perspex Alpha 2](http://img.youtube.com/vi/c_AB_XSILp0/0.jpg)](https://www.youtube.com/watch?v=c_AB_XSILp0 "Perspex Alpha 2")

Here are some of the highlights of this release - we've come a long way in the
3 weeks since alpha 1!

## Visual Studio Designer

[Nikita Tsukanov](https://github.com/kekekeks) has done the impossible and
got a basic designer for Visual Studio working. It's still early days and not
everything is supported yet, but it's a massive step towards a user-friendly
designer experience with Perspex. Take a look at the video above to see it in
action or download the [VS Plugin](https://visualstudiogallery.msdn.microsoft.com/87db356c-cec9-4a07-b7db-a4ed8a921ac9)

## Styles in XAML

We've added support for expressing styles in XAML.

{% highlight xml %}
<StackPanel>
  <StackPanel.Styles>
    <Style Selector="Button:pointerover">
      <Setter Property="Button.Foreground" Value="Red"/>
    </Style>
  </StackPanel.Styles>
  <Button>I will have red text when hovered.</Button>
</StackPanel>
{% endhighlight %}

To see more examples of what you can do with Perspex styles, [check out the documentation](https://github.com/Perspex/Perspex/blob/master/docs/styles.md)

Of course, our XAML support is only available thanks to [OmniXAML](https://github.com/superjmn/omnixaml)!

## \*Nix support

Perspex now runs on \*nix platforms using mono - this includes Linux and Mac
OSX. For information on building Perspex on \*nix take a look at the [build
instructions](https://github.com/Perspex/Perspex/blob/master/docs/build.md).

## HTML View

As well as doing the impossible once with the designer, [Nikita Tsukanov](https://github.com/kekekeks) has also ported the [HTML Renderer](https://htmlrenderer.codeplex.com/) component to Perspex, giving us a
100% managed HTML 4.01 and CSS 2 renderer directly in Perspex.

## New Controls Showcase

![Controls Showcase]({{ site.url }}/images/2015-09-21-perspex-alpha2/controls-showcase.png)

[Nelson Carrillo](https://github.com/ncarrillo) (as well as hammering our Cairo
backend into shape) has contributed a new controls showcase to Perspex. This
is a lot better looking than our previous cobbled-togther test application and
should give a good idea of what Perspex is capable of at the moment.

## New Features

The following new features have been implemented:

- ImageBrush
- VisualBrush
- Clipboard
- Canvas (thanks to [Amer Koleci](https://github.com/amerkoleci))
- Cursor support

## Join us

If you're wanting to join in, take a look at the [`up-for-grabs` issues on
GitHub](https://github.com/Perspex/Perspex/labels/up-for-grabs) - these issues
are an ideal place to start for a newcomer - then come join us in our [Gitter Room](https://gitter.im/Perspex/Perspex)
and let us know what you're thinking of doing.
