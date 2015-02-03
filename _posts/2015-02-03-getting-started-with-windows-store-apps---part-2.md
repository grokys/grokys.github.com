---
layout: post
title: "Getting started with Windows Store apps - Part 2"
description: ""
category: 
tags: []
---
{% include JB/setup %}

In the [last post]({{ site.url }}/2015/02/03/getting-started-with-windows-store-apps/) I described 
getting started with writing a Windows Store app up until I hit the snag of "how to create a round 
browse button".

Lets see how I got on after that!

# Actually Creating The Browse Button

After posting a question on StackOverflow, [I got an answer](http://stackoverflow.com/a/28289658/6448):

	<Button Style="{StaticResource TextBlockButtonStyle}">
	    <Button.RenderTransform>
	        <ScaleTransform ScaleX="2" ScaleY="2" />
	    </Button.RenderTransform>
	    <Grid>
	        <Rectangle Fill="DimGray" />
	        <Grid Margin="7,5,5,5">
	            <Grid.ColumnDefinitions>
	                <ColumnDefinition Width="22" />
	                <ColumnDefinition />
	            </Grid.ColumnDefinitions>
	            <TextBlock Grid.Column="0" Text="" FontFamily="Segoe UI Symbol" FontSize="18" VerticalAlignment="Center" HorizontalAlignment="Left" Margin="0,-2,0,0" />
	            <TextBlock Grid.Column="0" Text="" FontFamily="Segoe UI Symbol" FontSize="12" VerticalAlignment="Center" HorizontalAlignment="Center" />
	            <TextBlock Grid.Column="1" Text="Browse" FontWeight="Light" Margin="4,0,0,0" VerticalAlignment="Center" HorizontalAlignment="Left"  />
	        </Grid>
	    </Grid>
	</Button>
 
Seems rather excessive. I was kind of hoping for something like we had with AppBarButton, or at 
least something like:

	<Button Style={StaticResource CircleButtonStyle}>
        <Image Source="/Resources/Browse.png"/>
    </Button>

But anyway, we have our answer for now (please, if there's an easier way to do this, add a 
comment!).

# Writing a ViewModel

So lets add a view model - in the shared project, create a ViewModels folder and add 
StartPageViewModel.cs.

For now, I'll create a "Name" property in the ViewModel and then create a TextBox in the view, just 
to make sure that everything is wired up properly.

    public class StartPageViewModel
    {
        private string name = "Hello World!";

        public string Name
        {
            get { return this.name; }
            set { this.name = value; }
        }
	}

And

    <TextBox Text="{Binding Name}"/>

Now, I need to add an attached property to the StartPage to tell Prism to auto-locate a view model
for the page:

![Views must implement IView]({{ site.url }}/assets/2015-02-03-getting-started-with-windows-store-apps---part-2/views-must-implement-iview.png)

Ok, let's implement IView on StartPage. Hmm the error is still there. \*quick googling\*. So it
seems that's a bug in Prism - fair enough - lets ignore it, it shouldn't cause a problem.

Time to run the program to see if that worked!

![Failed to assign property]({{ site.url }}/assets/2015-02-03-getting-started-with-windows-store-apps---part-2/failed-to-assign-property.png)

Oh... "The text associated with this error code could not be found." - that's helpful! Um... How 
about we manually implement IView:

    public sealed partial class StartPage : Page, IView
    {
        public StartPage()
        {
            this.InitializeComponent();
        }

        object IView.DataContext
        {
            get { return this.DataContext; }
            set { this.DataContext = value; }
        }
    }

Aaaannnnd... That works. WTF? I can't even imagine what's going on here. Page obviously implements
DataContext so why couldn't it be bound before? What difference does explicitly implementing the 
interface make?

# Binding the Button to a Command

Anyway, it's working. Lets skip ahead 5 minutes now, when I find the `Command` binding on 
my button isn't calling the ICommand I've created for it on the ViewModel. I try changing the 
binding from `{Binding OpenFileCommand}` to `{Binding BadOpenFileCommand}`. BadOpenFileCommand 
doesn't exist, but I want to make sure that binding errors are being displayed in the Output window 
as they would in WPF.      

Oh, it seems that those messages appear [only when the debugging mode is set to "Mixed"](https://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.debugsettings.isbindingtracingenabled).
Lets try that. 

*Now* we're getting somewhere:

    Error: BindingExpression path error: 'BadOpenFileCommand' property not found on 'Vera.ViewModels.StartPageViewModel'. BindingExpression: Path='BadOpenFileCommand' DataItem='Vera.ViewModels.StartPageViewModel'; target element is 'Windows.UI.Xaml.Controls.Button' (Name='null'); target property is 'Command' (type 'ICommand')

OH HOLD ON! I've forgotten to bind the DataContext property to the strongly typed ViewModel property 
needed by ReactiveUI (I *love* ReactiveUI)! My bad.

----------

**LESSON LEARNED: Use "Mixed" debugging mode!** Maybe this should be the default? Binding error
messages are invaluable in debugging.

----------
 
# And Crash 

![Unhandled Exception]({{ site.url }}/assets/2015-02-03-getting-started-with-windows-store-apps---part-2/unhandled-exception.png)

I've not changed anything! That worked last time I ran. In fact, it seems to occur every other 
time I run. And the [stacktrace]({{ site.url }}/assets/2015-02-03-getting-started-with-windows-store-apps---part-2/stacktrace.txt) doesn't tell me much. 

Lets try turning off "Mixed" debugging.

It doesn't crash any more! Aha, so *that's* why Mixed debugging isn't the default!

----------

**LESSON LEARNED: Don't use "Mixed" debugging mode!**

----------

# The File Picker

Next thing to do was show the file picker. That went pretty smoothly!

# Navigate to a new Document

Once the user has selected a file, I need to navigate to a new Page. The NavigationService we used
in App.xaml.cs isn't available in Page as it's a part of Prism. It seems like the way to
do this is by making the page inherit from VisualStateAwarePage.

When this didn't work, I gave up in frustration and vented on Twitter. 

However, it looks like the XAML team have the last laugh. The problem was that I'd mistyped the 
xml namespace declaration:

    xmlns:prism="using Microsoft.Practices.Prism.StoreApps"

Note the space there instead of a ":". It might be nice if the XAML editor showed errors in using
declarations, but this was 100% my fault.

# Conclusion

I came to Windows Store apps with high hopes. I'm a relatively common WPF user, and I'm even in the 
process of writing my own windowing framework [Perspex](https://github.com/grokys/Perspex/) so I 
understand how difficult this stuff is. But compared to WPF and MahApps.Metro, I was quite shocked 
at how many roadblocks were placed in front of me. 

I wonder how a new developer coming to the XAML stack might find it? Apart from the problem of the 
bugs, the direction seems to have gone away from streamlining the framework and into adding more 
boilerplate. Unless of course you're willing to use something like Prism, but that itself seems to 
introduce a number of problems. Of course maybe this is my problem, as a somewhat veteran and this 
would all make sense to newcomers. 

Maybe if I decide to continue, I will add to this series, but that's all for now. 