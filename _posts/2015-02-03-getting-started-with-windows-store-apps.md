---
layout: post
title: "Getting started with Windows Store apps"
description: ""
category: 
tags: [windows-store, c#, .net]
---
{% include JB/setup %}

# Introduction

In my day job, we recently started a project that we thought would be a good fit for a Windows 
Store app - especially with the upcoming Windows 10 which will allow Store Apps to run in a
window and look more like traditional desktop apps.

This blog post is a diary of getting started with Window Store apps from the perspective of a
relatively seasoned WPF developer. **Spoiler**: It didn't go well.

# Background

We needed the application the be modular so I figured this would be the perfect excuse to try out
[Prism](https://msdn.microsoft.com/en-us/library/ff648465.aspx "Prism") which has been updated to
work with Windows Store apps. having watched a couple of videos and read a few articles I felt like
I was ready to get started. I'm using Visual Studio 2013 - at the time of writing Visual Studio
2015 is still in CTP.

# Getting Started

First thing to do was create a new project. 

Now; Universal App vs Windows App? It's unlikely that this app will ever be run on phones, but 
tablets? Maybe. Also people are saying [Universal Apps are the future](http://www.citeworld.com/article/2835278/development/universal-apps-is-the-future-of-windows-development.html)? Lets go with that. There's
not really any guidance I could find on whether or not Windows Apps should still be used.

So **Blank App (Universal App)** it is:

	using System;
	using System.Collections.Generic;
	using System.IO;
	using System.Linq;
	using System.Runtime.InteropServices.WindowsRuntime;
	using Windows.ApplicationModel;
	using Windows.ApplicationModel.Activation;
	using Windows.Foundation;
	using Windows.Foundation.Collections;
	using Windows.UI.Xaml;
	using Windows.UI.Xaml.Controls;
	using Windows.UI.Xaml.Controls.Primitives;
	using Windows.UI.Xaml.Data;
	using Windows.UI.Xaml.Input;
	using Windows.UI.Xaml.Media;
	using Windows.UI.Xaml.Media.Animation;
	using Windows.UI.Xaml.Navigation;
	
	// The Blank Application template is documented at http://go.microsoft.com/fwlink/?LinkId=234227
	
	namespace Vera
	{
	    /// <summary>
	    /// Provides application-specific behavior to supplement the default Application class.
	    /// </summary>
	    public sealed partial class App : Application
	    {
	#if WINDOWS_PHONE_APP
	        private TransitionCollection transitions;
	#endif
	
	        /// <summary>
	        /// Initializes the singleton application object.  This is the first line of authored code
	        /// executed, and as such is the logical equivalent of main() or WinMain().
	        /// </summary>
	        public App()
	        {
	            this.InitializeComponent();
	            this.Suspending += this.OnSuspending;
	        }
	
	        /// <summary>
	        /// Invoked when the application is launched normally by the end user.  Other entry points
	        /// will be used when the application is launched to open a specific file, to display
	        /// search results, and so forth.
	        /// </summary>
	        /// <param name="e">Details about the launch request and process.</param>
	        protected override void OnLaunched(LaunchActivatedEventArgs e)
	        {
	#if DEBUG
	            if (System.Diagnostics.Debugger.IsAttached)
	            {
	                this.DebugSettings.EnableFrameRateCounter = true;
	            }
	#endif
	
	            Frame rootFrame = Window.Current.Content as Frame;
	
	            // Do not repeat app initialization when the Window already has content,
	            // just ensure that the window is active
	            if (rootFrame == null)
	            {
	                // Create a Frame to act as the navigation context and navigate to the first page
	                rootFrame = new Frame();
	
	                // TODO: change this value to a cache size that is appropriate for your application
	                rootFrame.CacheSize = 1;
	
	                if (e.PreviousExecutionState == ApplicationExecutionState.Terminated)
	                {
	                    // TODO: Load state from previously suspended application
	                }
	
	                // Place the frame in the current Window
	                Window.Current.Content = rootFrame;
	            }
	
	            if (rootFrame.Content == null)
	            {
	#if WINDOWS_PHONE_APP
	                // Removes the turnstile navigation for startup.
	                if (rootFrame.ContentTransitions != null)
	                {
	                    this.transitions = new TransitionCollection();
	                    foreach (var c in rootFrame.ContentTransitions)
	                    {
	                        this.transitions.Add(c);
	                    }
	                }
	
	                rootFrame.ContentTransitions = null;
	                rootFrame.Navigated += this.RootFrame_FirstNavigated;
	#endif
	
	                // When the navigation stack isn't restored navigate to the first page,
	                // configuring the new page by passing required information as a navigation
	                // parameter
	                if (!rootFrame.Navigate(typeof(MainPage), e.Arguments))
	                {
	                    throw new Exception("Failed to create initial page");
	                }
	            }
	
	            // Ensure the current window is active
	            Window.Current.Activate();
	        }
	
	#if WINDOWS_PHONE_APP
	        /// <summary>
	        /// Restores the content transitions after the app has launched.
	        /// </summary>
	        /// <param name="sender">The object where the handler is attached.</param>
	        /// <param name="e">Details about the navigation event.</param>
	        private void RootFrame_FirstNavigated(object sender, NavigationEventArgs e)
	        {
	            var rootFrame = sender as Frame;
	            rootFrame.ContentTransitions = this.transitions ?? new TransitionCollection() { new NavigationThemeTransition() };
	            rootFrame.Navigated -= this.RootFrame_FirstNavigated;
	        }
	#endif
	
	        /// <summary>
	        /// Invoked when application execution is being suspended.  Application state is saved
	        /// without knowing whether the application will be terminated or resumed with the contents
	        /// of memory still intact.
	        /// </summary>
	        /// <param name="sender">The source of the suspend request.</param>
	        /// <param name="e">Details about the suspend request.</param>
	        private void OnSuspending(object sender, SuspendingEventArgs e)
	        {
	            var deferral = e.SuspendingOperation.GetDeferral();
	
	            // TODO: Save application state and stop any background activity
	            deferral.Complete();
	        }
	    }
	}

## Holy shit!

That's a lot of boilerplate! Takes me back to the MFC days (yes, I'm *just about* old enough). 
Luckily having watched the Channel 9 video 
[Prism for Windows Store apps](http://channel9.msdn.com/Shows/Visual-Studio-Toolbox/Prism-for-Windows-Store-Apps) I know that it will all be deleted.

----------
*Aside: It appears from watching the above video that the default template is geared toward 
developers who prefer the code-behind approach to coding. Except: every developer I've ever heard
of that uses the XAML stack uses MVVM. Why are MS still promoting code-behind to developers 
starting out? IMO it should be MVVM from the beginning. The only excuse is that code-behind 
development requires less code for a basic starting app. Except you can plainly see that that is
not the case here.*

----------

# Installing Prism

Right-click solution -> Manage NuGet Packages for Solution, search "prism", "Prism.StoreApps", 
install in both Windows and Windows Phone projects - easy.

Now we can delete all that nonsense in App.xaml.cs. Yay!

	using Microsoft.Practices.Prism.Mvvm;

	// The Blank Application template is documented at http://go.microsoft.com/fwlink/?LinkId=234227

	namespace Vera
	{
	    /// <summary>
	    /// Provides application-specific behavior to supplement the default Application class.
	    /// </summary>
	    public sealed partial class App : MvvmAppBase
	    {
	        /// <summary>
	        /// Initializes the singleton application object.  This is the first line of authored code
	        /// executed, and as such is the logical equivalent of main() or WinMain().
	        /// </summary>
	        public App()
	        {
	            this.InitializeComponent();
	        }
	    }
	}

*MUCH* better! Now I have to go and do the same in the XAML designer (wouldn't it be nice if it
could do this automatically?).

![XAML intellisense FTW!]({{ site.url }}/assets/2015-02-03-getting-started-with-windows-store-apps/xaml-intellisense-ftw.png)

I see XAML intellisense is helpful as ever! I was wondering if the WPF experience was second rate
compared to Windows Store but it looks like they both have the same problems. Never mind, I know 
what the class is called and after all, I understand: [enumerating the content of a namespace is 
difficult.](http://stackoverflow.com/questions/949246/how-to-get-all-classes-within-namespace)

# Navigating to the Start page

Now we can add the OnLaunchApplicationAsync method override in to navigate to the start page which
we're just about to create:

    protected override Task OnLaunchApplicationAsync(LaunchActivatedEventArgs args)
    {
        NavigationService.Navigate("Start", null);
        return Task.FromResult<object>(null);
    }

Ok, and now onto... hold on... Ummm... What? You can't find InitializeComponent now?

![Ummm... What?!]({{ site.url }}/assets/2015-02-03-getting-started-with-windows-store-apps/initialize-component-missing.png)

F6... Ok, you can find it now. It seems that whenever I make an edit in App.xaml.cs it confuses
intellisense and marks `this.InitializeComponent();` as an error. That's something I never saw in
WPF. Starting to look like we didn't have it so bad.

# Adding the start page

