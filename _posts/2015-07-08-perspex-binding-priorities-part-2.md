---
layout: post
title: "Perspex Binding Priorities Part 2"
excerpt: "Perspex has a CSS-style styling system which means that binding priorites (or precedence) are very important."
category: Perspex
tags: [perspex, c#, .net]
comments: true
---
In the [last post]({{ site.url }}/perspex/perspex-binding-priorities/) I described the
first attempt at binding priorites in [Perspex](https://github.com/grokys/Perspex/).

To see the problem with this simple system consider the following example:

{% highlight csharp %}
var textBox = new TextBox();
textBox.Bind(TextBox.TextProperty, viewModel.WhenAnyValue(x => Text));
textBox.GetObservable(TextBox.TextProperty).Subscribe(x => viewModel.Text = x);
{% endhighlight %}

This creates a two-way binding between the TextBox.Text property and the
viewModel.Text property. But there's a problem. With the system described in
part 1, *local values are stored as bindings*, as the `PriorityValue` is just
a linked list of bindings.

**Which means that a user typing into the TextBox overrides the binding setup
above.** So a binding would fire only once after user interaction and remain
dormant from then on.

Clearly this isn't desirable behavior.

The solution was that for each priority level, you need a "local value" : a
value set using `PerspexObject.SetValue` that takes affect only until the next
time a binding of the same priority fires.

And this is the story of how `PriorityValue` became a collection of
`PriortyLevel`s.

However, this isn't the end of the story...
