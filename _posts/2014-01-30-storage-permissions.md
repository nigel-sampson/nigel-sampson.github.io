---
layout: post
title: Storage Permissions in WinRT
tags: csharp windows-apps
---

Windows 8 applications operate in a sandbox that severely restricts access to the file system. By default your app only has access to its own local, roaming and temporary storage folders. This can be extended by declaring some capabilities such that your app can access some specific known folders such as the Pictures Library. But what about elsewhere? 

This is where the Pickers come in, specifically [FileOpenPicker][fileopen], [FileSavePicker][filesave] and [FolderPicker][folder], these are equivalent to the Open / Save dialogs of old. By the using these pickers the user grants implicit permission to the files and folders they’ve picked while the app is running. As long as you keep a reference to the StorageFile or StorageFolder you can work with them.

And now what about tomorrow or next week? The implicit permission to those files and folders only lasts as long as the app is running. How do we maintain those permissions over multiple sessions of the app? Thankfully Microsoft have included the classes in the namespace Windows.Storage.AccessCache that help us deal with these problems. Specifically the [StorageApplicationPermissions][sap] class and its two properties FutureAccessList and MostRecentlyUsed. 

These classes allow you to add Files and Folders with the Add method, either returning a custom identifying token or use one of your own. 

```csharp
var token = StorageApplicationPermissions.FutureAccessList.Add(file);
```

You can then store this token somewhere such as local storage or wherever makes sense for your app. Later on using the GetFIleAsync method allows us to restore the StorageFile and its implicit permission. If the file is renamed or moved then it all still works as the permission is moved with it. Very cool indeed.

```csharp
var file = await StorageApplicationPermissions.FutureAccessList.GetFileAsync(token);
```

[fileopen]: http://msdn.microsoft.com/library/windows/apps/br207847
[filesave]: http://msdn.microsoft.com/en-us/library/windows/apps/windows.storage.pickers.filesavepicker.aspx
[folder]: http://msdn.microsoft.com/en-us/library/windows/apps/windows.storage.pickers.folderpicker.aspx
[sap]: http://msdn.microsoft.com/en-us/library/windows/apps/windows.storage.accesscache.storageapplicationpermissions.aspx
