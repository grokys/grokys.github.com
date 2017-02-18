---
layout: post
title: "Supporting Multiple Visual Studio Versions in a Single Package"
excerpt: "A few people have asked how we support both Visual Studio 2015 and Visual Studio 2017 in the GitHub Extension for Visual Studio..."
category: GHfVS
tags: [github, ghfvs, visualstudio, c#, .net]
comments: true
---

# Supporting Multiple Visual Studio Versions in a Single Package

A few people [have](https://github.com/github/VisualStudio/issues/828) [asked](https://gitter.im/github/VisualStudio) how we support both Visual Studio 2015 and Visual Studio 2017 in the [GitHub Extension for Visual Studio](https://github.com/github/VisualStudio). In particular we needed to support Team Foundation 14 and 15, and we wanted to do it from a single package.

>  It's important to note that although each new version of Visual Studio [includes](https://www.nuget.org/packages/Microsoft.VisualStudio.Shell.Interop.12.0) all the previous [versions](https://www.nuget.org/packages/Microsoft.VisualStudio.Shell.Interop.11.0) of [assemblies](https://www.nuget.org/packages/Microsoft.VisualStudio.Shell.Interop.10.0) of many core APIs, Team Explorer is an extension and so a single version is installed.

The way we achieved this was to create two assemblies: [GitHub.TeamFoundation.14](https://github.com/github/VisualStudio/tree/master/src/GitHub.TeamFoundation.14) and [GitHub.TeamFoundation.15](https://github.com/github/VisualStudio/tree/master/src/GitHub.TeamFoundation.15). The files in the two assemblies are links to the same files, and thus identical - so how does this solve the problem?

The difference is that the two assemblies reference different versions of `Microsoft.TeamFoundation.Controls` and *each exported MEF class implements an interface, or has an attribute, property, field or constructor parameter of a type from that assembly*. When Visual Studio starts up and does its MEF scan, it loads both the assemblies and scans the types contained therein. If MEF determines that it can't create an instance of the type because of a missing reference, *then those types won't be added to the MEF catalog* and thus only a single version of each type will be registered: the one for the correct version of Visual Studio.

So if you look at the files in the `GitHub.TeamFoundation.X` projects you will see that [each](https://github.com/github/VisualStudio/blob/master/src/GitHub.TeamFoundation.14/Connect/GitHubConnectSection.cs#L26) [class](https://github.com/github/VisualStudio/blob/master/src/GitHub.TeamFoundation.14/Services/VSGitServices.cs#L31) has a reference to something in `Microsoft.TeamFoundation.Controls`: if this wasn't the case then two identical classes would be registered twice with MEF.
