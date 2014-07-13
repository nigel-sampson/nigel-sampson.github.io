---
layout: post
title: Minutes to Midnight
tags: windows-phone tutorial
category: windows-phone-tutorial
permalink: '/windows-phone/tutorials/minutes-to-midnight'
---

<span class="alignleft"><img src="/content/images/tutorials/minutes-to-midnight.png" alt="Minutes to Midnight"/></span>
How long till the end of the day? In this tutorial we cover creating your first Windows Phone 7 application, using a dispatch timer and embedding a custom font.


Our first Windows Phone 7 application will be a nice simple one, one that shows a countdown to midnight in an alarm clock style.

###Setting up the project
If you haven't gone to [developer.windowsphone.com](http://developer.windowsphone.com) and downloaded the WP7 tools then head over there and do that now. While that's downloading head over to [Font Zone](http://www.font-zone.com/download.php?fid=3505) and download the font Quartz Italic we'll be using. It's a nice simple alarm clock looking font. Once you have the tools fire up Visual Studio and create a  new "Windows Phone Application" project, I'm calling this one "Midnight". Add the font to the project using "Add Existing Item" and make sure it's Build Action is set to *Content* this ensure that the font file is packaged with the WP7 application.

###Creating the UI
Our UI is pretty simple, in the designer drag a <em>TextBlock</em> from the Toolbox on to device, give it the name "Countdown" and set the following properties.

 - **Width and Height**: Auto, you do this by selecting the "Advanced Properties" and choosing "Reset Value.
 - **Margin**: Clear the ones the designer set.
 - **Horizontal and Vertical Alignment**: Center
 - **Foreground**: Normally we'd make this red to mimic a real alarm clock but we'll let the user customise the colour by the phone theme. Under "Advanced Properties" select "Apply Resource" and then "Phone Accent Brush".
 - **Font**: Quartz Italic
 - **Font Size**: 64

The xaml for the ContentGrid should look like:

``` csharp
<Grid x:Name="ContentGrid" Grid.Row="1">
    <TextBlock x:Name="Countdown" VerticalAlignment="Center" FontFamily="quartzitalic.ttf#Quartz" Foreground="{StaticResource PhoneAccentBrush}" FontSize="64" HorizontalAlignment="Center" />
</Grid>
```

###Creating the Timer
What we'll be doing here is creating a timer that ticks every second, on these timer ticks we'll be updating our TextBlock. In the constructor of the Page we'll attach to the Loaded event, this occurs when the page is constructed and added to the object tree.

We'll attach the method OnTimerTick to the timer's tick event and in there resolve the time span till midnight and update the TextBlock. The full code behind looks like:

``` csharp
public partial class MainPage
{
    private DispatcherTimer timer;

    public MainPage()
    {
        InitializeComponent();

        Loaded += OnLoaded;
    }

    private void OnLoaded(object sender, RoutedEventArgs e)
    {
        timer = new DispatcherTimer
        {
            Interval = TimeSpan.FromSeconds(1)
        };

        timer.Tick += OnTick;

        timer.Start();
    }

 

    private void OnTick(object sender, EventArgs e)
    {
        var midnight = DateTime.Today.AddHours(24);

        var timeLeft = midnight - DateTime.Now;

        Countdown.Text = String.Format("{0:D2}:{1:D2}:{2:D2}", timeLeft.Hours, timeLeft.Minutes, timeLeft.Seconds);
    }
}
```


On a side note you may not get the expected result for the time till midnight, as it uses the time zone currently set on the phone itself (which I believe defaults to Alaska).

### Download the Code

The code for all the tutorials is available to download: [Windows Phone 7 Tutorials Solution][download].

[download]: http://compiledexperience.com/content/downloads/windows-phone-tutorials.zip