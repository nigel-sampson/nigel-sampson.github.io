---
layout: post
title: Keeping my Powershell profile in sync
tags: powershell
---

These days I’m doing my development work over a number of machines both here at home and at [Marker Metro][mm]. I’m a big fan of having the development environment being as similar as possible across all the machines (including things like keyboard and mouse where possible). One thing I really want to keep in sync is my [Powershell profile][profile] across all these machines.

This sets up things like [PoshGit][poshgit], Visual Studio Variables and some helper methods and shortcuts for me (I lean on [GitHub for Windows][github] for installation of Git and PostGit). These are obviously things I want to be exactly the same way on every single machine. So how do I achieve this?

The first is to lean on something like [SkyDrive][skydrive] (or is it OneDrive now?) or [DropBox][dropbox] to sync files between machines. Next I’ll extract everything that was in the profile into a separate *.ps1* file. You may need to make sure it’s all machine agnostic. I’ll then update my actual profile file to the following:

```powershell
& "C:\Users\Nigel\SkyDrive\Code\Powershell\Profile.ps1"
```

This simply executes the file now in SkyDrive. I’ll then grab that profile file and back that up in SkyDrive as well. When working with a new machine all I need to do is grab the profile file from SkyDrive and place it in the appropriate folder and I’m done.

One caveat is that you will need to set the [Execution Policy][exec] of Powershell to RemoteSigned.

I’m still looking for an effective way to sync my Git and Mercurial configurations between machines.

[mm]: http://www.markermetro.com/
[profile]: http://technet.microsoft.com/en-us/library/ee692764.aspx
[poshgit]: https://github.com/dahlbyk/posh-git
[github]: http://windows.github.com/
[skydrive]: skydrive.live.com
[dropbox]: https://www.dropbox.com/
[exec]: http://technet.microsoft.com/en-us/library/ee176961.aspx
