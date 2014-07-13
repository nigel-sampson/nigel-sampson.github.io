---
layout: post
title: Blendable MVVM &#58; Introduction and Databinding
tags: csharp silverlight  expression-blend
---

<p>This is the first post in a series about the Model, View, ViewModel pattern (MVVM), there are a lot of posts out there on this pattern and all seem to have a slightly different focus. The focus this series is going to take is keeping the pattern "Blendable" i.e. making sure we can take advantage of the tooling support in Expression Blend and that we don't interupt the designer / developer workflow.</p>
<p>I'm also not going to try and present the full system all in one step, we'll start simple and build up to a full working application.</p>
<p>MVVM is the presentation pattern <em>du jour</em> for WPF and Silverlight, it's also referred to as the "Presentation Model" pattern (from Fowler). Most people understand the Model and the View, but what the ViewModel defines can vary from developer to developer. For me <strong>the View Model is the Model of the View</strong>, it holds properties the View is bound to and commands the view invokes.</p>
<p>Since our ViewModel will be the source for data binding I'll need it to implement INotifyPropertyChanged, we'll do this on a base class, I also like to use Linq Expressions to generate property names, less magic strings.</p>
<p><!-- {\rtf1\ansi\ansicpg\lang1024\noproof65001\uc1 \deff0{\fonttbl{\f0\fnil\fcharset0\fprq1 Courier New;}}{\colortbl;??\red0\green0\blue255;\red255\green255\blue255;\red0\green0\blue0;\red43\green145\blue175;}??\fs20 \cf1 public\cf0  \cf1 class\cf0  \cf4 ViewModelBase\cf0 &lt;T&gt; : \cf4 INotifyPropertyChanged\cf0  \cf1 where\cf0  T : \cf4 ViewModelBase\cf0 &lt;T&gt;\par ??\{\par ??\tab \cf1 public\cf0  \cf1 event\cf0  \cf4 PropertyChangedEventHandler\cf0  PropertyChanged;\par ??\par ??\tab \cf1 protected\cf0  \cf1 virtual\cf0  \cf1 void\cf0  OnPropertyChanged(\cf4 PropertyChangedEventArgs\cf0  e)\par ??\tab \{\par ??\tab \tab \cf1 var\cf0  propertyChanged = PropertyChanged;\par ??\par ??\tab \tab \cf1 if\cf0 (propertyChanged != \cf1 null\cf0 )\par ??\tab \tab \tab propertyChanged(\cf1 this\cf0 , e);\par ??\tab \}\par ??\par ??\tab \cf1 protected\cf0  \cf1 virtual\cf0  \cf1 void\cf0  OnPropertyChanged(\cf1 params\cf0  \cf4 Expression\cf0 &lt;\cf4 Func\cf0 &lt;T, \cf1 object\cf0 &gt;&gt;[] expressions)\par ??\tab \{\par ??\tab \tab \cf1 foreach\cf0 (\cf1 var\cf0  expression \cf1 in\cf0  expressions)\par ??\tab \tab \{\par ??\tab \tab \tab OnPropertyChanged(\cf1 new\cf0  \cf4 PropertyChangedEventArgs\cf0 (\cf4 ReflectOn\cf0 &lt;T&gt;.GetPropertyName(expression)));\par ??\tab \tab \}\par ??\tab \}\par ??\}} --></p>
<div class="code" style="background: white none repeat scroll 0% 0%; font-family: Courier New; font-size: 10pt; color: black;">
<p style="margin: 0px;"><span style="color: blue;">public</span> <span style="color: blue;">class</span> <span style="color: #2b91af;">ViewModelBase</span>&lt;T&gt; : <span style="color: #2b91af;">INotifyPropertyChanged</span> <span style="color: blue;">where</span> T : <span style="color: #2b91af;">ViewModelBase</span>&lt;T&gt;</p>
<p style="margin: 0px;">{</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: blue;">event</span> <span style="color: #2b91af;">PropertyChangedEventHandler</span> PropertyChanged;</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">protected</span> <span style="color: blue;">virtual</span> <span style="color: blue;">void</span> OnPropertyChanged(<span style="color: #2b91af;">PropertyChangedEventArgs</span> e)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">var</span> propertyChanged = PropertyChanged;</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">if</span>(propertyChanged != <span style="color: blue;">null</span>)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; propertyChanged(<span style="color: blue;">this</span>, e);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">protected</span> <span style="color: blue;">virtual</span> <span style="color: blue;">void</span> OnPropertyChanged(<span style="color: blue;">params</span> <span style="color: #2b91af;">Expression</span>&lt;<span style="color: #2b91af;">Func</span>&lt;T, <span style="color: blue;">object</span>&gt;&gt;[] expressions)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">foreach</span>(<span style="color: blue;">var</span> expression <span style="color: blue;">in</span> expressions)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; OnPropertyChanged(<span style="color: blue;">new</span> <span style="color: #2b91af;">PropertyChangedEventArgs</span>(<span style="color: #2b91af;">ReflectOn</span>&lt;T&gt;.GetPropertyName(expression)));</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">}</p>
</div>
<p>The sample application we'll build is a cocktail browser, so our first simple view model will just display a list of cocktails, again for first article simplicity I'm using a synchronous data source (could be an embedded xml file) which is a hard coded list of cocktails. This'll be replaced with a WCF service in a later article.</p>
<p><!-- {\rtf1\ansi\ansicpg\lang1024\noproof65001\uc1 \deff0{\fonttbl{\f0\fnil\fcharset0\fprq1 Courier New;}}{\colortbl;??\red0\green0\blue255;\red255\green255\blue255;\red0\green0\blue0;\red43\green145\blue175;\red163\green21\blue21;}??\fs20 \cf1 public\cf0  \cf1 class\cf0  \cf4 CocktailService\par ??\cf0 \{\par ??\tab \cf1 public\cf0  \cf4 IEnumerable\cf0 &lt;\cf4 Cocktail\cf0 &gt; GetAll()\par ??\tab \{\par ??\tab \tab \cf1 return\cf0  \cf1 new\cf0 []\par ??\tab \tab \{\par ??\tab \tab \tab \cf1 new\cf0  \cf4 Cocktail\par ??\cf0 \tab \tab \tab \{\par ??\tab \tab \tab \tab Id = 1,\par ??\tab \tab \tab \tab Name = \cf5 "Black Russian"\par ??\cf0 \tab \tab \tab \},\par ??\tab \tab \tab \cf1 new\cf0  \cf4 Cocktail\par ??\cf0 \tab \tab \tab \{\par ??\tab \tab \tab \tab Id = 2,\par ??\tab \tab \tab \tab Name = \cf5 "Fuzzy Duck"\par ??\cf0 \tab \tab \tab \}\par ??\tab \tab \};\par ??\tab \}\par ??\}} --></p>
<div class="code" style="background: white none repeat scroll 0% 0%; font-family: Courier New; font-size: 10pt; color: black;">
<p style="margin: 0px;"><span style="color: blue;">public</span> <span style="color: blue;">class</span> <span style="color: #2b91af;">CocktailService</span></p>
<p style="margin: 0px;">{</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: #2b91af;">IEnumerable</span>&lt;<span style="color: #2b91af;">Cocktail</span>&gt; GetAll()</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">return</span> <span style="color: blue;">new</span>[]</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">new</span> <span style="color: #2b91af;">Cocktail</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Id = 1,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Name = <span style="color: #a31515;">"Black Russian"</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; },</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">new</span> <span style="color: #2b91af;">Cocktail</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Id = 2,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Name = <span style="color: #a31515;">"Fuzzy Duck"</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; };</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">}</p>
</div>
<p>Our CocktailsViewModel will expose an ObservableCollection&lt;Cocktail&gt;, this can be a simple auto property as we the collection won't be changing, it's contents will. For this nice simple example we'll just be retrieving the cocktails from the data source and adding to them to the exposed collection. I've created a couple of extension methods to make this a little easier.</p>
<p><!-- {\rtf1\ansi\ansicpg\lang1024\noproof65001\uc1 \deff0{\fonttbl{\f0\fnil\fcharset0\fprq1 Courier New;}}{\colortbl;??\red0\green0\blue255;\red255\green255\blue255;\red0\green0\blue0;\red43\green145\blue175;}??\fs20 \cf1 public\cf0  \cf1 class\cf0  \cf4 CocktailsViewModel\cf0  : \cf4 ViewModelBase\cf0 &lt;\cf4 CocktailsViewModel\cf0 &gt;\par ??\{\par ??\tab \cf1 private\cf0  \cf1 readonly\cf0  \cf4 CocktailService\cf0  cocktailService = \cf1 new\cf0  \cf4 CocktailService\cf0 ();\par ??\par ??\tab \cf1 public\cf0  CocktailsViewModel()\par ??\tab \{\par ??\tab \tab AvailableCocktails = \cf1 new\cf0  \cf4 ObservableCollection\cf0 &lt;\cf4 Cocktail\cf0 &gt;();\par ??\par ??\tab \tab \cf1 var\cf0  recipes = cocktailService.GetAll();\par ??\par ??\tab \tab AvailableCocktails.AddRange(recipes);\par ??\tab \}\par ??\par ??\tab \cf1 public\cf0  \cf4 ObservableCollection\cf0 &lt;\cf4 Cocktail\cf0 &gt; AvailableCocktails\par ??\tab \{\par ??\tab \tab \cf1 get\cf0 ; \cf1 set\cf0 ;\par ??\tab \}\par ??\}} --></p>
<div class="code" style="background: white none repeat scroll 0% 0%; font-family: Courier New; font-size: 10pt; color: black;">
<p style="margin: 0px;"><span style="color: blue;">public</span> <span style="color: blue;">class</span> <span style="color: #2b91af;">CocktailsViewModel</span> : <span style="color: #2b91af;">ViewModelBase</span>&lt;<span style="color: #2b91af;">CocktailsViewModel</span>&gt;</p>
<p style="margin: 0px;">{</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">private</span> <span style="color: blue;">readonly</span> <span style="color: #2b91af;">CocktailService</span> cocktailService = <span style="color: blue;">new</span> <span style="color: #2b91af;">CocktailService</span>();</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> CocktailsViewModel()</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; AvailableCocktails = <span style="color: blue;">new</span> <span style="color: #2b91af;">ObservableCollection</span>&lt;<span style="color: #2b91af;">Cocktail</span>&gt;();</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">var</span> recipes = cocktailService.GetAll();</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; AvailableCocktails.AddRange(recipes);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: #2b91af;">ObservableCollection</span>&lt;<span style="color: #2b91af;">Cocktail</span>&gt; AvailableCocktails</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">get</span>; <span style="color: blue;">set</span>;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">}</p>
</div>
<p><!-- {\rtf1\ansi\ansicpg\lang1024\noproof65001\uc1 \deff0{\fonttbl{\f0\fnil\fcharset0\fprq1 Courier New;}}{\colortbl;??\red0\green0\blue255;\red255\green255\blue255;\red0\green0\blue0;\red43\green145\blue175;\red163\green21\blue21;}??\fs20 \cf1 public\cf0  \cf1 static\cf0  \cf1 class\cf0  \cf4 ObservableCollectionsExtensions\par ??\cf0 \{\par ??\tab \cf1 public\cf0  \cf1 static\cf0  \cf1 void\cf0  AddRange&lt;T&gt;(\cf1 this\cf0  \cf4 ObservableCollection\cf0 &lt;T&gt; collection, \cf4 IEnumerable\cf0 &lt;T&gt; items)\par ??\tab \{\par ??\tab \tab \cf1 if\cf0 (collection == \cf1 null\cf0 )\par ??\tab \tab \tab \cf1 throw\cf0  \cf1 new\cf0  \cf4 ArgumentNullException\cf0 (\cf5 "collection"\cf0 );\par ??\par ??\tab \tab \cf1 if\cf0 (items == \cf1 null\cf0 )\par ??\tab \tab \tab \cf1 throw\cf0  \cf1 new\cf0  \cf4 ArgumentNullException\cf0 (\cf5 "items"\cf0 );\par ??\par ??\tab \tab \cf1 foreach\cf0 (\cf1 var\cf0  item \cf1 in\cf0  items)\par ??\tab \tab \{\par ??\tab \tab \tab collection.Add(item);\par ??\tab \tab \}\par ??\tab \}\par ??\par ??\tab \cf1 public\cf0  \cf1 static\cf0  \cf1 void\cf0  Replace&lt;T&gt;(\cf1 this\cf0  \cf4 ObservableCollection\cf0 &lt;T&gt; collection, \cf4 IEnumerable\cf0 &lt;T&gt; items)\par ??\tab \{\par ??\tab \tab \cf1 if\cf0 (collection == \cf1 null\cf0 )\par ??\tab \tab \tab \cf1 throw\cf0  \cf1 new\cf0  \cf4 ArgumentNullException\cf0 (\cf5 "collection"\cf0 );\par ??\par ??\tab \tab \cf1 if\cf0 (items == \cf1 null\cf0 )\par ??\tab \tab \tab \cf1 throw\cf0  \cf1 new\cf0  \cf4 ArgumentNullException\cf0 (\cf5 "items"\cf0 );\par ??\par ??\tab \tab collection.Clear();\par ??\tab \tab collection.AddRange(items);\par ??\tab \}\par ??\}} --></p>
<div class="code" style="background: white none repeat scroll 0% 0%; font-family: Courier New; font-size: 10pt; color: black;">
<p style="margin: 0px;"><span style="color: blue;">public</span> <span style="color: blue;">static</span> <span style="color: blue;">class</span> <span style="color: #2b91af;">ObservableCollectionsExtensions</span></p>
<p style="margin: 0px;">{</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: blue;">static</span> <span style="color: blue;">void</span> AddRange&lt;T&gt;(<span style="color: blue;">this</span> <span style="color: #2b91af;">ObservableCollection</span>&lt;T&gt; collection, <span style="color: #2b91af;">IEnumerable</span>&lt;T&gt; items)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">if</span>(collection == <span style="color: blue;">null</span>)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">throw</span> <span style="color: blue;">new</span> <span style="color: #2b91af;">ArgumentNullException</span>(<span style="color: #a31515;">"collection"</span>);</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">if</span>(items == <span style="color: blue;">null</span>)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">throw</span> <span style="color: blue;">new</span> <span style="color: #2b91af;">ArgumentNullException</span>(<span style="color: #a31515;">"items"</span>);</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">foreach</span>(<span style="color: blue;">var</span> item <span style="color: blue;">in</span> items)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; collection.Add(item);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: blue;">static</span> <span style="color: blue;">void</span> Replace&lt;T&gt;(<span style="color: blue;">this</span> <span style="color: #2b91af;">ObservableCollection</span>&lt;T&gt; collection, <span style="color: #2b91af;">IEnumerable</span>&lt;T&gt; items)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">if</span>(collection == <span style="color: blue;">null</span>)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">throw</span> <span style="color: blue;">new</span> <span style="color: #2b91af;">ArgumentNullException</span>(<span style="color: #a31515;">"collection"</span>);</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">if</span>(items == <span style="color: blue;">null</span>)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">throw</span> <span style="color: blue;">new</span> <span style="color: #2b91af;">ArgumentNullException</span>(<span style="color: #a31515;">"items"</span>);</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; collection.Clear();</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; collection.AddRange(items);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">}</p>
</div>
<p>So now we have the worlds simplest ViewModel lets fire up Blend and take the role of the designer and hooking our layout to the ViewModel. This is my first attempt at a screen cast so lets see how well this works:</p>
<p>&nbsp;</p>
<div id="silverlightControlHost">
<object width="600" height="450" data="data:application/x-silverlight," type="application/x-silverlight">
<param name="source" value="/clientbin/mediaplayer.xap" />
<param name="autoUpgrade" value="true" />
<param name="minRuntimeVersion" value="3.0.40624.0" />
<param name="enableHtmlAccess" value="true" />
<param name="enableGPUAcceleration" value="true" />
<param name="initparams" value="playerSettings =                          &lt;Playlist&gt;                             &lt;AutoLoad&gt;true&lt;/AutoLoad&gt;                             &lt;AutoPlay&gt;false&lt;/AutoPlay&gt;                             &lt;DisplayTimeCode&gt;false&lt;/DisplayTimeCode&gt;                             &lt;EnableCachedComposition&gt;true&lt;/EnableCachedComposition&gt;                             &lt;EnableCaptions&gt;true&lt;/EnableCaptions&gt;                             &lt;EnableOffline&gt;true&lt;/EnableOffline&gt;                             &lt;EnablePopOut&gt;true&lt;/EnablePopOut&gt;                             &lt;StartMuted&gt;false&lt;/StartMuted&gt;                             &lt;StretchMode&gt;None&lt;/StretchMode&gt;                             &lt;Items&gt; 								&lt;PlaylistItem&gt; 									&lt;AudioCodec&gt;&lt;/AudioCodec&gt; 									&lt;Description&gt;&lt;/Description&gt; 									&lt;FileSize&gt;2797299&lt;/FileSize&gt; 									&lt;FrameRate&gt;15.000015000015&lt;/FrameRate&gt; 									&lt;Height&gt;600&lt;/Height&gt; 									&lt;IsAdaptiveStreaming&gt;false&lt;/IsAdaptiveStreaming&gt; 									&lt;MediaSource&gt;http://compiledexperience.com/content/video/BlendableViewModel.1.wmv&lt;/MediaSource&gt; 									&lt;ThumbSource&gt;http://compiledexperience.com/content/video/BlendableViewModel.1_Thumb.jpg&lt;/ThumbSource&gt; 									&lt;Title&gt;Blendable%20View%20Model%20Databinding&lt;/Title&gt; 									&lt;VideoCodec&gt;VC1&lt;/VideoCodec&gt; 									&lt;Width&gt;960&lt;/Width&gt; 								&lt;/PlaylistItem&gt;                             &lt;/Items&gt;                         &lt;/Playlist&gt;" />
<div onmouseover="highlightDownloadArea(true)" onmouseout="highlightDownloadArea(false)"><img style="border-style: none; position: absolute; width: 100%; height: 100%;" src="/content/video/BlendableViewModel.1_Thumb.jpg" alt="" /> <img style="border-style: none; position: absolute; width: 100%; height: 100%;" src="/content/video/Preview.png" alt="" /> 
<table style="position: absolute; height: 100%;" border="0" width="100%">
<tbody>
<tr>
<td align="center" valign="middle"><img src="http://go2.microsoft.com/fwlink/?LinkId=108181" alt="Get Microsoft Silverlight" /></td>
</tr>
</tbody>
</table>
<a href="http://go2.microsoft.com/fwlink/?LinkID=124807"> <img class="fadeCompletely" style="border-style: none; position: absolute; width: 100%; height: 100%;" alt="Get Microsoft Silverlight" /> </a></div>
</object>
</div>
<p>&nbsp;</p>
<p>We're going to be doing what's been called "View First" MVVM, this simply means we'll be creating the View which in turn will create the ViewModel (we'll introduce Dependency Injection later on). Other approaches include "ViewModel first" or some third party (a "ViewManager" creating the pair).</p>
<p>So to begin with we'll create our view model as the DataContext for the UserControl using the property grid, we then simply bind our list to the AvailableCocktails via the "Explicit DataContext" tab under Data Binding. I'll also set the DisplayMemberPath to the cocktail's name.</p>
<p>We're now all done! In the next post in this series we'll have our view model created from Ninject and unit test the ViewModel.</p>
<p><a href="http://www.dotnetkicks.com/kick/?url=http%3a%2f%2fcompiledexperience.com%2fblog%2fposts%2fBlendable-MVVM-Introduction-and-Databinding"><img src="http://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fcompiledexperience.com%2fblog%2fposts%2fBlendable-MVVM-Introduction-and-Databinding" border="0" alt="kick it on DotNetKicks.com" /></a> <a rev="vote-for" href="http://dotnetshoutout.com/Blendable-MVVM-Introduction-and-Databinding-Blog-Compiled-Experience-Silverlight-Development"><img style="border:0px" src="http://dotnetshoutout.com/image.axd?url=http%3A%2F%2Fcompiledexperience.com%2Fblog%2Fposts%2FBlendable-MVVM-Introduction-and-Databinding" alt="Shout it" /></a></p>
