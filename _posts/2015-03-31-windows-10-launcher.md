---
layout: post
title: What's new in Windows 10 - Launcher
tags: windows-apps windows-phone  
---

Microsoft have finally started to show off what's available in Windows 10 for us app developers. Over the next few days we can start to take a look at some of these changes and what it can mean. To begin with let's look at what you can do with `Windows.System.Launcher`.

## Disclaimer

This was written using the initial release of the Windows 10 Developer Tools running on Windows 10 build 10041 and things will mostly change before final release. I'll try to come back and keep these posts up to date as time goes on.

## Launcher

A number of new methods were added to `Windows.System.Launcher`, previously it had the capabilities to launch files and uri's. This would open the app responsible for either that file extension or uri protocol. If none existed on the user's machine then it would prompt to open the store to download the appropriate apps. If multiple apps were installed then an application picker was displayed.

### LaunchFolderAsync

You can use `Windows.System.Launcher.LaunchFolderAsync` to open the File Explorer to the folder that you specify.

``` csharp
var folder = KnownFolders.PicturesLibrary;

await Launcher.LaunchFolderAsync(folder);
```

To use the Pictures Library ensure you have the following capability declared in `Package.appxmanifest`.

``` xml
<uap:Capability Name="picturesLibrary" />
```

### LaunchUriForResultsAsync

In Windows 10 there's new systems for app to app communication, one of which is a way to launch an app using a protocol and await for the launched app to return some result. This could be used for something like a third party payment processor as an app on the device or potentially social authentication. Imagine rather than launching `WebAuthenticationBroker` for logging into something like Twitter but instead the Twitter app.

Here we use `TargetApplicationPackageFamilyName` to ensure we launch a trusted app and not some other app trying to hijack our protocol.

``` csharp
var options = new LauncherOptions
{
	TargetApplicationPackageFamilyName = "Desired.App_fahbwp72s5828"
};

var result = await Launcher.LaunchUriForResultsAsync(new Uri("custom://auth?username=test"), options);

if (result.Status == LaunchUriStatus.Success)
{
    ...
}
```

On the target app end they use `OnActivated` to receive the correct activation arguments and a reference to the operation.

``` csharp
protected override void OnActivated(IActivatedEventArgs args)
{
	if (args.Kind == ActivationKind.ProtocolForResults)
	{
		var protocolArgs = (ProtocolForResultsActivatedEventArgs) args;

		var operation = protocolArgs.ProtocolForResultsOperation;

		...
    }
}
```

and then when the target app is completed use the operation to report completion and hand off to the original app.

``` csharp
var values = new ValueSet
{
	{ "Success", true },
	{ "Token", "ec1b29ac-2296-4b27-8ea1-848188e83ea2" }
};

operation.ReportCompleted(values);
```

You can view more on this at [Developer's Guide to Windows 10 Preview: (10) App-to-App Communication](https://channel9.msdn.com/Series/Developers-Guide-to-Windows-10-Preview/10).

### QueryUriSupportAsync

With all this new functionality you may what to first check to see if an operation is supported. If you have a collection of apps that use protocols to communicate data then it would be better user experience if you can detect whether the other app is installed and ready to handle your protocol. 

This would also be very useful if you're trying to support third party apps such as [MetroTube](http://www.metrotubeapp.com/), they have a custom protocol you can use to launch their app to view YouTube videos. You can now conditionally show your *View in MetroTube* button only when their app is installed.

``` csharp
var status = await Launcher.QueryUriSupportAsync(new Uri("mailto:"), LaunchUriType.LaunchUri);

if (status == QueryUriSupportStatus.Success)
{
    ...
}
```

### Settings

Windows Phone 8 has had this for a while but now it's universal, being able to launch into specific settings areas can increase your usability by enabling users to quickly fix issues your app is encountering.

For instance to launch the Wi-Fi settings we use.

``` csharp
await Launcher.LaunchUriAsync(new Uri("ms-settings://network/wifi"));
```

## Conclusion

That sums up some of the major changes to `Windows.System.Launcher`. There's a few quality of life changes but the major one is the `LaunchUriForResultsAsync` method. It will be interesting to see what developers make of this and if an ecosystem of third party apps emerges.