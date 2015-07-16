# Mixins

Here's another thing i've stolen from Polymer: Mixins. Well, in Polymer they're called Behaviors
but in XAML-land Behaviors are already a thing and I didn't want to create more confusion than
necessary.

In Perspex there are various controls that need to maintain selections. These classes have
SelectedIndex, SelectedItem etc properties that need to be kept in sync, and also need to react to
changes in the individual item's selection state, such that saying `item.IsSelected = true` has the
same effect as `list.SelectedItem = item`.

Unfortunately these often have different base classes, so we need some form of multiple inheritance.
Obviously that doesn't exist in .Net so I looked to Polymer and came up with something which works
quite nicely.

Mixins listen for property changed events and input events etc and react to these changes by setting
the related properties on the control.
