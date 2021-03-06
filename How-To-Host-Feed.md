# How To Host Your Own [Private/Internal/Public] Package Repository Server (aka Package Feed)

## Why?
Please read [[Depending on the chocolatey.org as an organization|ChocolateyFAQs#should-my-organization-depend-on-use-the-community-feed-httpschocolateyorgpackages]]

## Host your own server
There are three types of package repositories, [folder/unc share](#local-folder--unc-share), [simple server](#simple-server), and the sophisticated [package gallery](#package-gallery).

**Alternative Hosting Options (that require less work/maintenance):**
* [MyGet](https://www.myget.org/) - MyGet has free public and paid for public/private cloud-hosting options if you don't want to handle all of the pain of setup and administration. MyGet offers some stellar options like multi-feed aggregation, mirroring, and source package build services!
* [ProGet](http://inedo.com/proget/overview) - ProGet gives you a ready to go On-Premise option ([installable with Chocolatey!](https://chocolatey.org/packages/proget)).
* [Artifactory](http://www.jfrog.com/open-source/) - see [Artifactory NuGet Repositories](http://www.jfrog.com/confluence/display/RTF/NuGet+Repositories) - not in the free version
* [TeamCity](https://www.jetbrains.com/teamcity/) has built-in Simple Server
* [Nexus](https://books.sonatype.com/nexus-book/reference/nuget-nuget_proxy_repositories.html) - Sonatype Nexus has a built-in simple server

## Package Version Immutability
A package version is immutable on some sources. This means that everybody's version 1.0.1 of a particular package is the same. You do not need to worry about this when updating with newer versions of packages, because each package version compiled nupkg has the unique version in the name (e.g `bob.1.0.0.nupkg` vs `bob.1.0.1.nupkg` ).

Package immutability is usually desired, because then you know that everyone on v1.0.0 of a package has exactly the same code as does even everyone else. Even a broken version v1.0.0 gives you a global understanding that everyone who has v1.0.0 has exactly the same bits. This really simplifies administration. Without immutability, there is no guarantee that a version of a package installed is the same as the version of the package at the source.

## Local Folder / UNC Share (CIFS)
Perhaps the easiest to set up and recommended for testing quick and dirty scenarios, local folder is easily a strong point when you need a quick source for packages.

**Advantages:**
* Speed of setup (can be setup almost immediately).
* Package store is filesystem.
* Can be easily upgrade to Simple Server.
* Permissions are based on file system/share permissions.
* There is no limitation on package sizes (or rather, it can likely handle 100MB+ file sizes, maybe even GB sized packages). Don't create multiple GB sized packages, what is wrong with you?! ;)

**Disadvantages:**
* Anyone with permission can push and overwrite packages.
* No HTTP/HTTPS pushing. Must be able to access the folder/share to push to it.
* Starts to affect choco performance once the source has over 500 packages (maybe?).
* **Big disadvantage**: For a file share there is not a guarantee of package version immutability. Does not do anything to keep from package versions being overwritten. This provides no immutability of a package version and no guarantee that a version of a package installed is the same as the version in the source.

### Local Folder Share Setup

No really, it's that easy. Just set your permissions appropriately and put packages in the folder. You are already done.

## Simple Server

There is where the bulk of NuGet OData compatible servers fall (Nexus, Nuget.Server, Chocolatey.Server, Artifactory, MyGet, etc). Since Chocolatey just uses an enhanced version of the NuGet framework, it is compatible everywhere you can put a NuGet package.

**Advantages:**
* Setup can be really simple - just a website and IIS for some simple servers.
* Windows is not required - there are at least two pure Java versions (see [Non-Windows Hosting](#non-windows-hosting)).
* Push over HTTP/HTTPS.
* API key for pushing packages.
* No direct access to packages so security can be locked down to just modify through push.
* Package store is file system.

**Disadvantages:**
* Only one API key, so no multi user scenarios.
* Starts to affect choco performance once the source has over 500 packages (maybe?).
* No moderation.
* No website to view packages.
* No package statistics.
* A package should may be limited to 28.61MB by default on some simple servers. Depending on your simple server - For IIS simple servers package size can be controlled through [maxAllowedContentLength](https://msdn.microsoft.com/en-us/library/ms689462(v=vs.90).aspx) and [maxRequestLength](https://msdn.microsoft.com/en-us/library/e1f13641(v=vs.100).aspx). For others like Nexus, it may already be set very high. You can host the installer internally somewhere and access it through packaging though.

The actual limit for package sizes varies depending on what each simple server can handle (usually determined by the limitation of pushing a package to the server). If you determine what those are, we'd be happy to list each one here.

#### Simple Server Setup

Many google searches will throw out good ways to set up your own feed (hint: search for host your own nuget server feed)

Some notable references:
 * Nuget Docs [Host Your Own Remote Feed](https://docs.nuget.org/Create/Hosting-Your-Own-NuGet-Feeds#creating-remote-feeds)
 * itToby - [Setup Your Own Chocolatey/NuGet Repository](http://blog.ittoby.com/2014/07/setup-your-own-chocoloateynuget.html)
 * Rich Hopkins - [Bake your own Chocolatey NuGet repository](https://souladin.wordpress.com/2014/12/05/bake-your-own-chocolatey-nuget-repository/)
 * Brandon - [Host NuGet Server in Azure](http://netitude.bc3tech.net/2015/01/07/create-your-own-hosted-nuget-server-in-azure/)

##### Chocolatey Server Setup
[Chocolatey Server](https://chocolatey.org/packages/chocolatey.server) is a simple Nuget.Server that is ready to rock and roll. It has already completed Steps 1-3 of NuGet's [host your own remote feed](https://docs.nuget.org/Create/Hosting-Your-Own-NuGet-Feeds#creating-remote-feeds). Version 0.1.2 has the following additional adds:

* Uses same enhanced NuGet that Chocolatey uses so you can see more information in search if you choose to use those things.
* Allows packages up to 2GB. Package size can be controlled through [maxAllowedContentLength](https://msdn.microsoft.com/en-us/library/ms689462(v=vs.90).aspx) and [maxRequestLength](https://msdn.microsoft.com/en-us/library/e1f13641(v=vs.100).aspx).

Set it up:

 1. Install the [Chocolatey Server](https://chocolatey.org/packages/chocolatey.server) package - `choco install chocolatey.server -y`
 1. Configure IIS appropriately for the package.
 1. Be sure to give the app pool modify permissions to the `App_Data` folder.

Alternative means of installation? If you have [Puppet](https://docs.puppetlabs.com/puppet/), you can take a look at the [chocolatey server module](https://forge.puppetlabs.com/chocolatey/chocolatey_server).

## Package Gallery
This is like what https://chocolatey.org (the community feed runs on). It is the most advanced, having both a file store for packages and a database for tracking all sorts of information.

**Advantages:**
* Can deal with thousands of packages with no performance issues.
* Package versions are immutable - in other words you can guarantee the version installed is the same as the version in the source.
* Package store can be File system, Azure blobs, or AWS S3 (**Chocolatey Package Gallery only**).
* Multiple users each having their own API keys.
* User registration with email confirmation.
* Interaction and collaboration based.
* Can have administrators.
* Can take advantage of moderation (**Chocolatey Package Gallery only**).
* Package statistics (download counts, etc).
* A website to view package information.
* Can be configured to send email.

**Disadvantages:**
* Speed of setup (can take longer than the rest). There are many moving parts to configure.
* Requires Windows/IIS/SQL Server/SMTP (hopefully with the proper licenses on each of those).
* Not well-documented, could require some diligence to get working.
* A package should not be bigger than 150MB. You can host the installer internally somewhere and access it through packaging though. Package size can be controlled through [maxAllowedContentLength](https://msdn.microsoft.com/en-us/library/ms689462(v=vs.90).aspx) and [maxRequestLength](https://msdn.microsoft.com/en-us/library/e1f13641(v=vs.100).aspx).

#### Package Gallery Setup
Only approach this if you are a Windows Admin with significant experience in setting up SQL Server databases and IIS for ASP.NET MVC sites. We don't have resources to help support the setup, but we can point you to [NuGet Gallery Setup](https://github.com/NuGet/NuGetGallery/wiki/Hosting-the-NuGet-Gallery-Locally-in-IIS).

##### Chocolatey Package Gallery Setup

At this time we don't have setup instructions and are not keen to answer questions specifically surrounding the setup of a Chocolatey Gallery (the code behind chocolatey.org). This is due to specific necessary settings regarding the community packages repository and tight integration to what it offers. Chocolatey for Business is likely to offer a gallery at some point, depending on prioritization.

## Non-Windows Hosting
If you don't want to host on Windows you have only the following options (from least advanced to most advanced - these options typically also work on Windows):
* CIFS share
* [JNuGet](https://bitbucket.org/aristar/jnuget/wiki/Home) - a simple server
* [NuGet.Java.Server](http://blog.jonnyzzz.name/2012/03/nuget-server-in-pure-java.html) ([NuGet Package](https://www.nuget.org/packages/NuGet.Java.Server)) - simple server (same tool used in TeamCity)
* [TeamCity](https://www.jetbrains.com/teamcity/) - contains built-in simple server
* [PHP NuGet](http://www.kendar.org/?p=/dotnet/phpnuget) - Simple server built in PHP
* [Hazel](https://github.com/MPIB/hazel) - Simple server built in Rust
* [Artifactory](http://www.jfrog.com/open-source/) - see [Artifactory NuGet Repositories](http://www.jfrog.com/confluence/display/RTF/NuGet+Repositories) - not in the free version
* [Nexus](https://books.sonatype.com/nexus-book/reference/nuget-nuget_proxy_repositories.html)

**Note:** NuGet.Java.Server, TeamCity and JNuGet are about the same in terms of sophistication (they are ordered alphabetically).
