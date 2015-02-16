---
layout: post
title: "Perspex Properties"
excerpt: "PerspexProperties are Perspex's version of the XAML frameworks DependencyProperty."
category: Perspex
tags: [perspex, c#, .net]
comments: true
---

In this post I'm going to explore Perspex's version of the XAML frameworks' `DependencyProperty` -
`PerspexProperty`.

# Perspex Properties #

PerspexProperty is the equivalent of WPF's DependencyProperty. Dependency/Perspex properties give 
you a number of important features over simple .NET properties with `INotifyPropertyChanged`: 

* **Property value inheritance**. If the `FontFamily` property is not set on a control it will use the 
`FontFamily` of its parent, which if it's not set will use the value of *it's* parent and so on. In 
this way, setting `FontFamily` on the top-level `Window` can affect all of the controls contained 
in that window.
* **Attached Properties**. Attached properties can be defined which can add arbitrary properties to 
controls. The control itself doesn't need to know what to do with an attached property - the 
behaviour can be handled elsewhere. An example of an attached property is `Grid.Column`.
* **Default values**. The value for each property applicable to a control doesn't need to be stored 
unless it differs from the default value.
* **Coercion**. Perspex properties can also register a callback to handle coercion.
* **Binding**. The value of a `PerspexProperty` can be bound to the result of an `IObservable`.
* **Binding Priorities**. Bindings may have a priority, so that values that come from the styling
system can be overridden by values that are set locally. 

# Declaring a Perspex Property #

Declaring a `DependencyProperty` in WPF looks something like this:

{% highlight csharp %}
public static readonly DependencyProperty PropertyDeclaration =
	DependencyProperty.Register(
	    "PropertyName",
	    typeof(PropertyType),
	    typeof(OwnerClass),
	    new FrameworkPropertyMetadata(
	        default(PropertyType),
	        FrameworkPropertyMetadataOptions.Inherits));

public PropertyType PropertyName
{
    get { return (PropertyType)this.GetValue(PropertyDeclaration); }
    set { this.SetValue(PropertyDeclaration, value); }
}
{% endhighlight %}

A lot of boilerplate there. With  generics and default parameters we can make it look a bit nicer:

{% highlight csharp %}
public static readonly PerspexProperty<PropertyType> PropertyDeclaration =
    PerspexProperty.Register<OwnerClass, PropertyType>("PropertyName", inherits: true);

public PropertyType PropertyName
{
    get { return this.GetValue(PropertyDeclaration); }
    set { this.SetValue(PropertyDeclaration, value); }
}
{% endhighlight %}

What can we see here?

- PerspexProperties are typed, so no more having to cast in the getter.
- We pass the property type and owner class as a generic type to `Register()` so we don't have to 
write `typeof()` twice.
- We used default parameter values in `Register()` so that defaults don't have to be restated.

Like `DependencyProperty`s in XAML, perspex properties need to be defined on a class that inherits
from the `PerspexObject` class. 

# Adding a Perspex Property to Another Type #

Just like in WPF, you can share perspex properties between unrelated controls by calling 
`PerspexProperty.AddOwner':

{% highlight csharp %}
public static readonly PerspexProperty<int> FooProperty =
    RegisteringControl.FooProperty.AddOwner<AnotherControl>();
{% endhighlight %}

# Property Value Inheritance #

Here we declare an inherited integer property called "Foo":

{% highlight csharp %}
public static readonly PerspexProperty<int> FooProperty =
    PerspexProperty.Register<OwnerClass, int>("Foo", inherits: true);
{% endhighlight %}

By specifying the `inherits: true` parameter in the call to `PerspexProperty.Register`, we are 
saying that in the absence of an explicitly set value for "Foo" on a control, the value from the
parent control should be used. 

Note, `PerspexObject` is defined at a lower level than the concept of "parent controls" so 
`PerspexObject` uses the protected `InheritanceParent` property to determine the parent control. 
This is automatically set by `Visual` (which inherits from `PerspexObject`) so you shouldn't usually 
have to worry about it. 

## Attached Properties

Attached properties are essentially the same as attached dependency properties in WPF. They are 
defined by calling `PerspexProperty.RegisterAttached`:

{% highlight csharp %}
public static readonly PerspexProperty<int> ColumnProperty =
    PerspexProperty.RegisterAttached<Grid, Control, int>("Column");
{% endhighlight %}

## Default Values

Default values are provided in the call to `PerspexProperty.Register` or `RegisterAttached`. If no
default value is provided the default is taken to be `default(TValue)`.

{% highlight csharp %}
public static readonly PerspexProperty<int> FooProperty =
    PerspexProperty.Register<OwnerClass, int>("Foo", defaultValue: 42);
{% endhighlight %}

Just like in WPF, you can override the default value for a specific type:

{% highlight csharp %}
FooProperty.OverrideDefaultValue(typeof(AnotherControl), 64);
{% endhighlight %}
 

## Coercion

Coercion allows a control to react to changes in a property's value and make sure that the value is
within a valid range. For example a "Percentage" property may only allow values between 0 and 100:

{% highlight csharp %}
public static readonly PerspexProperty<double> PercentageProperty =
    PerspexProperty.Register<OwnerClass, double>("Percentage", coerce: CoercePercentage);

private static double CoercePercentage(PerspexObject o, double value)
{
	return Math.Min(100, Math.Max(0, value));
}
{% endhighlight %}
 
The coercion method must be static, so the object on which the property change has taken place is
passed as a parameter. Note that you should not assume that the type of the object is the same as
the type that registered the property as properties can be added to other classes using 
`PerspexProperty.AddOwner`. If you need to access the object on which the property change has taken
place you should check the type first.

Currently coercion cannot be overridden for other classes, this is a limitation that may need to be
lifted in future. 

# Binding #

Binding in Perspex uses Reactive Extensions' [IObservable](http://msdn.microsoft.com/library/dd990377.aspx). To bind an IObservable to a property, use the `Bind()` method:

{% highlight csharp %}
control.Bind(BorderProperty, someObject.SomeObservable());
{% endhighlight %}

Note that because PerspexProperty is typed, we can check that the observable is of the correct type.

To get the value of a property as an observable, call `GetObservable()`:

{% highlight csharp %}
var observable = control.GetObservable(Control.FooProperty);
{% endhighlight %}

# Binding Priorities #

A binding may also have a priority. The `BindingPriority` enumeration gives a set of common 
priorities:

{% highlight csharp %}
public enum BindingPriority
{
    Animation = -1,
    LocalValue,
    StyleTrigger,
    TemplatedParent,
    Style,
    Unset = int.MaxValue,
}
{% endhighlight %}

As you can see, lower integral values are considered to be of a higher priority. We'll explore how
priorites work in another post.

# Binding in Initialization Lists #

One of the goals of perspex was to make defining a UI in code almost as painless as using markup.
To these ends, you can use initalization lists everywhere to give a XAML-like feel to your control
declarations in C#:

{% highlight csharp %}
return new Panel
{
	Children = new Controls
	{
		new TextBlock
		{
			Text = "Hello World!"
		}
	}
};
{% endhighlight %}

This works fine for perspex properties that are exposed as standard .NET properties, but how can we
make this work for attached properties and bindings. Well, C# 6 introduces a new feature called    
[index initializers](https://roslyn.codeplex.com/wikipage?title=Language%20Feature%20Status&referringTitle=Home)
which can allow us to do this: 

{% highlight csharp %}
var control = new Control
{
	Property1 = "Foo",
    [Attached.Property] = "Bar",
}
{% endhighlight %}

Nice... Now lets suppose we want to bind a property:

{% highlight csharp %}
var control = new Control
{
	Property1 = "Foo",
    [Attached.Property] = "Bar",
	[!Property2] = something.SomeObservable,
}
{% endhighlight %}

Yep, by putting a bang in front of the property name you can **bind** to a property (attached or 
otherwise) from the object initializer.

Binding to a property on another control? Easy:

{% highlight csharp %}
var control = new Control
{
	Property1 = "Foo",
[Attached.Property] = "Bar",
	[!Property2] = anotherControl[!Property1],
}
{% endhighlight %}

Two way binding? Just add two bangs:

{% highlight csharp %}
var control = new Control
{
	Property1 = "Foo",
[Attached.Property] = "Bar",
	[!!Property2] = anotherControl[!!Property1],
}
{% endhighlight %}

If you're writing a control template however, you don't want to bind at the LocalValue binding 
priority, you want to bind using the TemplatedParent binding priority. To do this use the tilde
operator instead. Here's an example from `ScrollViewer`'s control template:

{% highlight csharp %}
new ScrollContentPresenter
{
    Id = "contentPresenter",
    [~ScrollContentPresenter.ContentProperty] = control[~ScrollViewer.ContentProperty],
    [~~ScrollContentPresenter.ExtentProperty] = control[~~ScrollViewer.ExtentProperty],
    [~~ScrollContentPresenter.OffsetProperty] = control[~~ScrollViewer.OffsetProperty],
    [~~ScrollContentPresenter.ViewportProperty] = control[~~ScrollViewer.ViewportProperty],
    [~ScrollContentPresenter.CanScrollHorizontallyProperty] = control[~ScrollViewer.CanScrollHorizontallyProperty],
},
{% endhighlight %}

As before, doubling up the tilde operator creates a two-way binding.