---
layout: post
title: Building the iOS 7 parallax background on Windows Phone
tags: windows-phone
---

After looking at the new iOS 7 design videos and seeing it in action I was intrigued by some of the concepts Apple are using. One of which is that while being a “flat” design it has the idea of layers of UI which can be shown off by the parallax background behind the app icons (I typed tiles out of habit when writing this).  You can see it in the [launch video][ios7] at around the 2:45 mark.

So how can we create something similar for Windows Phone, below is a quick guide on using the accelerometer and render transformations to achieve a similar result. Definitely not quite as polished but certainly on the way. 

The basic premise of the feature is using the accelerometer to move a background image around underneath the content of the page. This gives the illusion of depth between the background and the content. 

There important part of the xaml is this, notice the background spans both the standard rows in the grid template and it's behind the rest of the content. We'll use a render transform for the actual movement so we'll set one up now. The other important feature is the negative margin, this scales the image out in all directions further than the screen, we'll need this to have something to move to. The value -24 itself is arbitrary and you can tweak it to change the amount of "travel" the background can have. 

``` xml
<Grid Margin="-24" Grid.RowSpan="2">
    <Grid.RenderTransform>
        <TranslateTransform x:Name="BackgroundTransform" />
    </Grid.RenderTransform>
    <Image x:Name="Background" Source="/Assets/Background.jpg" Stretch="Fill" />
</Grid>
 
<StackPanel Grid.Row="0" Margin="12,17,0,28">
    <TextBlock Text="COMPILED EXPERIENCE" Style="{StaticResource PhoneTextNormalStyle}" Margin="12,0"/>
    <TextBlock Text="parallax demo" Margin="9,-7,0,0" Style="{StaticResource PhoneTextTitle1Style}"/>
</StackPanel>
```

Initially when I spiked this prototype I was using the raw accelerometer on the phone and while it worked the shaky 50Hz nature of the samples caused the background to very shaky and not particularly good looking. While searching for ways to smooth out the raw data I discovered [this post by the Windows Phone team][accel] on the accelerometer and how to deal with things like smoothing and calibration. So we'll go ahead and use that. 

On page load we wire up an event listener for our handy AccelerometerHelper class, enable it and calibrate it to the level the user is currently holding it. 

``` csharp
private void OnLoaded(object sender, RoutedEventArgs e)
{
    AccelerometerHelper.Instance.Calibrate(true, true);
 
    AccelerometerHelper.Instance.ReadingChanged += OnAccelerometerHelperReadingChanged;
    AccelerometerHelper.Instance.Active = true;
}
```

On reading changed we take the x and y components of the acceleration, I'm using Average Acceleration for the smoothest result and we're scaling it by -64. This number feels a little arbitrary and it sort of is. If we use -24 (the same as the margin) then background would have shifted it's travel distance only when the phone is at right angles to the user which isn't that helpful. Increasing the number lets the background travel quicker (but also means we need to make sure it stops travelling when necessary). 

The accelerometer events are fired on a background thread so we need to dispatch back the UI thread before updating the transform. Our last step is to make sure the values don't exceed -24 and 24 so the background doesn't start to show it's edges (apparently I was imagining it when I thought Math had a Clamp method). 

``` csharp
private void OnAccelerometerHelperReadingChanged(object sender, AccelerometerHelperReadingEventArgs e)
{
    var x = e.AverageAcceleration.X * -64.0;
    var y = e.AverageAcceleration.Y * 64.0;
 
    Dispatcher.BeginInvoke(() =>
    {
        BackgroundTransform.X = Math.Max(-24, Math.Min(24, x));
        BackgroundTransform.Y = Math.Max(-24, Math.Min(24, y));
    });
}
```

That's it, really simple. The full code for the example is up on [GitHub][github].

[ios7]: http://www.apple.com/ios/ios7/
[accel]: http://blogs.windows.com/windows_phone/b/wpdev/archive/2010/09/08/using-the-accelerometer-on-windows-phone-7.aspx
[github]: https://github.com/nigel-sampson/parallax-demos
