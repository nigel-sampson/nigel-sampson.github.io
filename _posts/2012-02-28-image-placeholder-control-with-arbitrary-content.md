---
layout: post
title: Image Placeholder Control with Arbitrary Content
tags: silverlight windows-phone
---

For any application that's displaying remote images having a backup placeholder content is necessary for a polished user experience for all the times where the user has a slow internet connection, or none at all.

One possible solution from [Marker Metro][marker] use Caliburn Micro and defines a behavior in order to handle failed image requests to place a placeholder image, this solution doesn't handle displaying any content before the image is loaded. David Ansom from Microsoft has created a [Placeholder Image control][delay] that lets you define an image to display before the remote image is loaded and if the image is loaded.

One requirement I had was not to just display an image has a placeholder but arbitrary xaml content. What we really want to create is a marriage of Content Control and Image where we display the Content until the Image loads or if the Image fails. Since Image is a sealed control we'll create a new control based off the Content Control. This gives up the properties Content and Content Template, we'll need to add the Image properties we need Source and Stretch.

``` csharp
public class PlaceholderImage : ContentControl
{
    public static readonly DependencyProperty SourceProperty =
        DependencyProperty.Register("Source", typeof(ImageSource), typeof(PlaceholderImage), new PropertyMetadata(OnSourceChanged));

    public static readonly DependencyProperty StretchProperty =
        DependencyProperty.Register("Stretch", typeof(Stretch), typeof(PlaceholderImage), null);

    public PlaceholderImage()

    {
        DefaultStyleKey = typeof(PlaceholderImage);
    }

    public ImageSource Source
    {
        get { return (ImageSource)GetValue(SourceProperty); }
        set { SetValue(SourceProperty, value); }
    }

    public Stretch Stretch
    {
        get { return (Stretch)GetValue(StretchProperty); }
        set { SetValue(StretchProperty, value); }
    }
    
    ...
}

```

I want to use the Visual State Manager in this control to be able to have an animated transition from the content to the image; our initial state will be "Content" so we'll transition to that state in the override of OnApplyTemplate.

``` csharp
public override void OnApplyTemplate()
{
    base.OnApplyTemplate();

    VisualStateManager.GoToState(this, "Content", false);
}
```

In our handler for the Source property changing I follow what seems to be a standard pattern that the static method calls into a similar non static method on the control. Given this callback will be called when the image changes we’ll first make sure we’re in the Content state, we then remove our image opened handler from the previous image and add it to the new image. Once the image is loaded we’ll shift to the Image state making sure we use transitions. 

``` csharp
private void OnSourceChanged(ImageSource oldValue, ImageSource newValue)
{
    VisualStateManager.GoToState(this, "Content", false);

    var oldBitmapSource = oldValue as BitmapImage;
    var newBitmapSource = newValue as BitmapImage;

    if(oldBitmapSource != null)
    {
        oldBitmapSource.ImageOpened -= OnImageOpened;
    }

    if(newBitmapSource != null)
    {
        newBitmapSource.ImageOpened += OnImageOpened;
    }
}

private void OnImageOpened(object sender, EventArgs e)
{
    VisualStateManager.GoToState(this, "Image", true);
}

private static void OnSourceChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
{
    var placeholderImage = (PlaceholderImage)d;
    
    placeholderImage.OnSourceChanged((ImageSource)e.OldValue, (ImageSource)e.NewValue);
}
```

That’s it for code, very simple; however some of the magic lives the Style for the control in Generic.xaml.  This defines the Visual States and the animation between them, in this case we’ll fade the image in.

``` xml
<Style TargetType="controls:PlaceholderImage">
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="controls:PlaceholderImage">
                <Grid Width="{TemplateBinding Width}" Height="{TemplateBinding Height}" Margin="{TemplateBinding Margin}">
                    <VisualStateManager.VisualStateGroups>
                        <VisualStateGroup x:Name="ContentStates">
                            <VisualStateGroup.Transitions>
                                <VisualTransition GeneratedDuration="0:0:1.0">
                                    <VisualTransition.GeneratedEasingFunction>
                                        <ExponentialEase EasingMode="EaseInOut" Exponent="6"/>
                                    </VisualTransition.GeneratedEasingFunction>
                                </VisualTransition>
                            </VisualStateGroup.Transitions>
                            <VisualState x:Name="Content" />
                            <VisualState x:Name="Image">
                                <Storyboard>
                                    <DoubleAnimation Duration="0" To="1" Storyboard.TargetProperty="(UIElement.Opacity)" Storyboard.TargetName="ImageContent" d:IsOptimized="True"/>
                                </Storyboard>
                            </VisualState>
                        </VisualStateGroup>
                    </VisualStateManager.VisualStateGroups>
                    <ContentControl Content="{TemplateBinding Content}" ContentTemplate="{TemplateBinding ContentTemplate}" />
                    <Image x:Name="ImageContent" Source="{TemplateBinding Source}" Stretch="{TemplateBinding Stretch}" Opacity="0" />
                </Grid>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

The usage of the control is pretty simple, any content declared inside the control is used as placeholder content. 

``` xml
<controls:PlaceholderImage Source="{Binding Product.ThumbUrl, Converter={StaticResource BitmapImage}}" HorizontalAlignment="Center">
    <TextBlock Text="Loading"/>
</controls:PlaceholderImage>
```

I've included the code for this in the [Compiled Experience Phone Toolkit][toolkit] I'm putting together so you can download it and use within your apps. Take a look and let me know how it can be improved.

[marker]: http://www.markermetro.com/2011/06/technical/mvvm-placeholder-images-for-failed-image-load-on-windows-phone-7-with-caliburn-micro/ 

[delay]: http://blogs.msdn.com/b/delay/archive/2011/09/08/know-your-place-in-life-free-placeimage-control-makes-it-easy-to-add-placeholder-images-to-any-wpf-silverlight-or-windows-phone-application.aspx

[toolkit]: https://bitbucket.org/nigel.sampson/compiled-experience-toolkit/
