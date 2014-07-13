---
layout: post
title: Blendable MVVM &#58; Commands and Behaviors
tags: csharp silverlight expression-blend
---

<p>In the first post of the series we looked at using Binding expressions to display data from the ViewModel on our screen. The other major part of our interaction is having the view invoke functionality in the ViewModel. In the MVVM pattern this is done using the Command pattern, specifically the ICommand interface.</p>
<p><img src="/content/images/posts/mvvm.png" alt="Model, View, View Model Overfiew" width="528" height="151" /></p>
<p>A Command object encapsulates an action the user can take and is exposed by the ViewModel. Unfortunately Silverlight unlike WPF doesn't have any built in support for Commands beyond the actual interface itself. All the major Silverlight frameworks such as Prism,Caliburn etc has their own variation on how they should be implemented (usually through Attached Behaviors).</p>
<p>Given that we want to keep our MVVM structure "Blendable" we'll be using the Behaviors that come with Blend. What'll we be doing is creating an <strong>ExecuteCommandAction</strong> that we can trigger using any Blend trigger, this action will be bound to a ICommand exposed by the ViewModel.</p>
<p>In the current version of Silverlight standard behaviors don't support Binding, there is a work around I discussed earlier on this blog by PeteBlois using BindingListeners and exposing Bindings rather than ICommand. You can view more on this at  <a id="m77g" title="his blog" href="http://blois.us/blog/2009/04/datatrigger-bindings-on-non.html">his blog</a>.  The code for our new action looks like this, it shouldn't have to be this complicated but thats another rant.</p>
<p><!-- {\rtf1\ansi\ansicpg\lang1024\noproof65001\uc1 \deff0{\fonttbl{\f0\fnil\fcharset0\fprq1 Courier New;}}{\colortbl;??\red0\green0\blue255;\red255\green255\blue255;\red0\green0\blue0;\red43\green145\blue175;\red163\green21\blue21;}??\fs20 \cf1 public\cf0  \cf1 class\cf0  \cf4 ExecuteCommandAction\cf0  : \cf4 TriggerAction\cf0 &lt;\cf4 FrameworkElement\cf0 &gt;\par ??\{\par ??\tab \cf1 private\cf0  \cf1 readonly\cf0  \cf4 BindingListener\cf0  commandListener;\par ??\tab \cf1 private\cf0  \cf1 readonly\cf0  \cf4 BindingListener\cf0  commandParameterListener;\par ??\par ??\tab \cf1 public\cf0  ExecuteCommandAction()\par ??\tab \{\par ??\tab \tab commandListener = \cf1 new\cf0  \cf4 BindingListener\cf0 ();\par ??\tab \tab commandParameterListener = \cf1 new\cf0  \cf4 BindingListener\cf0 ();\par ??\tab \}\par ??\par ??\tab \cf1 public\cf0  \cf1 static\cf0  \cf1 readonly\cf0  \cf4 DependencyProperty\cf0  CommandProperty =\par ??\tab \tab \cf4 DependencyProperty\cf0 .Register(\cf5 "Command"\cf0 , \cf1 typeof\cf0 (\cf4 Binding\cf0 ), \cf1 typeof\cf0 (\cf4 ExecuteCommandAction\cf0 ), \cf1 new\cf0  \cf4 PropertyMetadata\cf0 (\cf1 null\cf0 , OnCommandChanged));\par ??\par ??\tab \cf1 public\cf0  \cf1 static\cf0  \cf1 readonly\cf0  \cf4 DependencyProperty\cf0  CommandParameterProperty =\par ??\tab \tab \cf4 DependencyProperty\cf0 .Register(\cf5 "CommandParameter"\cf0 , \cf1 typeof\cf0 (\cf4 Binding\cf0 ), \cf1 typeof\cf0 (\cf4 ExecuteCommandAction\cf0 ), \cf1 new\cf0  \cf4 PropertyMetadata\cf0 (\cf1 null\cf0 , OnCommandParameterChanged));\par ??\par ??\tab \cf1 public\cf0  \cf4 Binding\cf0  Command\par ??\tab \{\par ??\tab \tab \cf1 get\par ??\cf0 \tab \tab \{\par ??\tab \tab \tab \cf1 return\cf0  (\cf4 Binding\cf0 )GetValue(CommandProperty);\par ??\tab \tab \}\par ??\tab \tab \cf1 set\par ??\cf0 \tab \tab \{\par ??\tab \tab \tab SetValue(CommandProperty, \cf1 value\cf0 );\par ??\tab \tab \}\par ??\tab \}\par ??\par ??\tab \cf1 public\cf0  \cf4 Binding\cf0  CommandParameter\par ??\tab \{\par ??\tab \tab \cf1 get\par ??\cf0 \tab \tab \{\par ??\tab \tab \tab \cf1 return\cf0  (\cf4 Binding\cf0 )GetValue(CommandParameterProperty);\par ??\tab \tab \}\par ??\tab \tab \cf1 set\par ??\cf0 \tab \tab \{\par ??\tab \tab \tab SetValue(CommandParameterProperty, \cf1 value\cf0 );\par ??\tab \tab \}\par ??\tab \}\par ??\par ??\tab \cf1 private\cf0  \cf1 static\cf0  \cf1 void\cf0  OnCommandChanged(\cf4 DependencyObject\cf0  d, \cf4 DependencyPropertyChangedEventArgs\cf0  e)\par ??\tab \{\par ??\tab \tab ((\cf4 ExecuteCommandAction\cf0 )d).OnCommandBindingChanged(e);\par ??\tab \}\par ??\par ??\tab \cf1 private\cf0  \cf1 static\cf0  \cf1 void\cf0  OnCommandParameterChanged(\cf4 DependencyObject\cf0  d, \cf4 DependencyPropertyChangedEventArgs\cf0  e)\par ??\tab \{\par ??\tab \tab ((\cf4 ExecuteCommandAction\cf0 )d).OnCommandParameterBindingChanged(e);\par ??\tab \}\par ??\par ??\tab \cf1 private\cf0  \cf1 void\cf0  OnCommandBindingChanged(\cf4 DependencyPropertyChangedEventArgs\cf0  e)\par ??\tab \{\par ??\tab \tab commandListener.Binding = (\cf4 Binding\cf0 )e.NewValue;\par ??\tab \}\par ??\par ??\tab \cf1 private\cf0  \cf1 void\cf0  OnCommandParameterBindingChanged(\cf4 DependencyPropertyChangedEventArgs\cf0  e)\par ??\tab \{\par ??\tab \tab commandParameterListener.Binding = (\cf4 Binding\cf0 )e.NewValue;\par ??\tab \}\par ??\par ??\tab \cf1 protected\cf0  \cf1 override\cf0  \cf1 void\cf0  OnAttached()\par ??\tab \{\par ??\tab \tab \cf1 base\cf0 .OnAttached();\par ??\par ??\tab \tab commandListener.Element = AssociatedObject;\par ??\tab \tab commandParameterListener.Element = AssociatedObject;\par ??\tab \}\par ??\par ??\tab \cf1 protected\cf0  \cf1 override\cf0  \cf1 void\cf0  OnDetaching()\par ??\tab \{\par ??\tab \tab \cf1 base\cf0 .OnDetaching();\par ??\par ??\tab \tab commandListener.Element = \cf1 null\cf0 ;\par ??\tab \tab commandParameterListener.Element = \cf1 null\cf0 ;\par ??\tab \}\par ??\par ??\tab \cf1 protected\cf0  \cf1 override\cf0  \cf1 void\cf0  Invoke(\cf1 object\cf0  parameter)\par ??\tab \{\par ??\tab \tab \cf1 var\cf0  command = commandListener.Value \cf1 as\cf0  \cf4 ICommand\cf0 ;\par ??\par ??\tab \tab \cf1 if\cf0 (command == \cf1 null\cf0 )\par ??\tab \tab \tab \cf1 return\cf0 ;\par ??\par ??\tab \tab \cf1 var\cf0  commandParameter = commandParameterListener.Value;\par ??\par ??\tab \tab \cf1 if\cf0 (command.CanExecute(commandParameter))\par ??\tab \tab \tab command.Execute(commandParameter);\par ??\tab \}\par ??\}} --></p>
<div class="code" style="background: white none repeat scroll 0% 0%; font-family: Courier New; font-size: 10pt; color: black; -moz-background-clip: -moz-initial; -moz-background-origin: -moz-initial; -moz-background-inline-policy: -moz-initial;">
<p style="margin: 0px;"><span style="color: blue;">public</span> <span style="color: blue;">class</span> <span style="color: #2b91af;">ExecuteCommandAction</span> : <span style="color: #2b91af;">TriggerAction</span>&lt;<span style="color: #2b91af;">FrameworkElement</span>&gt;</p>
<p style="margin: 0px;">{</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">private</span> <span style="color: blue;">readonly</span> <span style="color: #2b91af;">BindingListener</span> commandListener;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">private</span> <span style="color: blue;">readonly</span> <span style="color: #2b91af;">BindingListener</span> commandParameterListener;</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> ExecuteCommandAction()</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; commandListener = <span style="color: blue;">new</span> <span style="color: #2b91af;">BindingListener</span>();</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; commandParameterListener = <span style="color: blue;">new</span> <span style="color: #2b91af;">BindingListener</span>();</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: blue;">static</span> <span style="color: blue;">readonly</span> <span style="color: #2b91af;">DependencyProperty</span> CommandProperty =</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: #2b91af;">DependencyProperty</span>.Register(<span style="color: #a31515;">"Command"</span>, <span style="color: blue;">typeof</span>(<span style="color: #2b91af;">Binding</span>), <span style="color: blue;">typeof</span>(<span style="color: #2b91af;">ExecuteCommandAction</span>), <span style="color: blue;">new</span> <span style="color: #2b91af;">PropertyMetadata</span>(<span style="color: blue;">null</span>, OnCommandChanged));</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: blue;">static</span> <span style="color: blue;">readonly</span> <span style="color: #2b91af;">DependencyProperty</span> CommandParameterProperty =</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: #2b91af;">DependencyProperty</span>.Register(<span style="color: #a31515;">"CommandParameter"</span>, <span style="color: blue;">typeof</span>(<span style="color: #2b91af;">Binding</span>), <span style="color: blue;">typeof</span>(<span style="color: #2b91af;">ExecuteCommandAction</span>), <span style="color: blue;">new</span> <span style="color: #2b91af;">PropertyMetadata</span>(<span style="color: blue;">null</span>, OnCommandParameterChanged));</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: #2b91af;">Binding</span> Command</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">get</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">return</span> (<span style="color: #2b91af;">Binding</span>)GetValue(CommandProperty);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">set</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; SetValue(CommandProperty, <span style="color: blue;">value</span>);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: #2b91af;">Binding</span> CommandParameter</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">get</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">return</span> (<span style="color: #2b91af;">Binding</span>)GetValue(CommandParameterProperty);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">set</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; SetValue(CommandParameterProperty, <span style="color: blue;">value</span>);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">private</span> <span style="color: blue;">static</span> <span style="color: blue;">void</span> OnCommandChanged(<span style="color: #2b91af;">DependencyObject</span> d, <span style="color: #2b91af;">DependencyPropertyChangedEventArgs</span> e)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; ((<span style="color: #2b91af;">ExecuteCommandAction</span>)d).OnCommandBindingChanged(e);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">private</span> <span style="color: blue;">static</span> <span style="color: blue;">void</span> OnCommandParameterChanged(<span style="color: #2b91af;">DependencyObject</span> d, <span style="color: #2b91af;">DependencyPropertyChangedEventArgs</span> e)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; ((<span style="color: #2b91af;">ExecuteCommandAction</span>)d).OnCommandParameterBindingChanged(e);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">private</span> <span style="color: blue;">void</span> OnCommandBindingChanged(<span style="color: #2b91af;">DependencyPropertyChangedEventArgs</span> e)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; commandListener.Binding = (<span style="color: #2b91af;">Binding</span>)e.NewValue;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">private</span> <span style="color: blue;">void</span> OnCommandParameterBindingChanged(<span style="color: #2b91af;">DependencyPropertyChangedEventArgs</span> e)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; commandParameterListener.Binding = (<span style="color: #2b91af;">Binding</span>)e.NewValue;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">protected</span> <span style="color: blue;">override</span> <span style="color: blue;">void</span> OnAttached()</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">base</span>.OnAttached();</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; commandListener.Element = AssociatedObject;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; commandParameterListener.Element = AssociatedObject;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">protected</span> <span style="color: blue;">override</span> <span style="color: blue;">void</span> OnDetaching()</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">base</span>.OnDetaching();</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; commandListener.Element = <span style="color: blue;">null</span>;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; commandParameterListener.Element = <span style="color: blue;">null</span>;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">protected</span> <span style="color: blue;">override</span> <span style="color: blue;">void</span> Invoke(<span style="color: blue;">object</span> parameter)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">var</span> command = commandListener.Value <span style="color: blue;">as</span> <span style="color: #2b91af;">ICommand</span>;</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">if</span>(command == <span style="color: blue;">null</span>)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">return</span>;</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">var</span> commandParameter = commandParameterListener.Value;</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">if</span>(command.CanExecute(commandParameter))</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; command.Execute(commandParameter);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">}</p>
</div>
<p>The functionality we're going to provide is to list the available cocktails, then once a user selects one we show similar cocktails (similar being defined as sharing two or more ingredients). We've modified the Cocktails service to look like this.</p>
<p><!-- {\rtf1\ansi\ansicpg\lang1024\noproof65001\uc1 \deff0{\fonttbl{\f0\fnil\fcharset0\fprq1 Courier New;}}{\colortbl;??\red0\green0\blue255;\red255\green255\blue255;\red0\green0\blue0;\red43\green145\blue175;\red163\green21\blue21;}??\fs20 \cf1 public\cf0  \cf1 class\cf0  \cf4 CocktailService\par ??\cf0 \{\par ??\tab \cf1 private\cf0  \cf1 readonly\cf0  \cf4 List\cf0 &lt;\cf4 Cocktail\cf0 &gt; cocktails = \cf1 new\cf0  \cf4 List\cf0 &lt;\cf4 Cocktail\cf0 &gt;\par ??\tab \{\par ??\tab \tab \cf1 new\cf0  \cf4 Cocktail\par ??\cf0 \tab \tab \tab \{\par ??\tab \tab \tab \tab Id = 1,\par ??\tab \tab \tab \tab Name = \cf5 "Black Russian"\cf0 ,\par ??\tab \tab \tab \tab Ingredients = \cf1 new\cf0  [] \{ \cf5 "Vodka"\cf0 , \cf5 "Kahlua"\cf0  \}\par ??\tab \tab \tab \},\par ??\tab \tab \tab \cf1 new\cf0  \cf4 Cocktail\par ??\cf0 \tab \tab \tab \{\par ??\tab \tab \tab \tab Id = 2,\par ??\tab \tab \tab \tab Name = \cf5 "White Russian"\cf0 ,\par ??\tab \tab \tab \tab Ingredients = \cf1 new\cf0 [] \{ \cf5 "Vodka"\cf0 , \cf5 "Cream"\cf0 , \cf5 "Kahlua"\cf0  \}\par ??\tab \tab \tab \},\par ??\tab \tab \tab \cf1 new\cf0  \cf4 Cocktail\par ??\cf0 \tab \tab \tab \{\par ??\tab \tab \tab \tab Id = 3,\par ??\tab \tab \tab \tab Name = \cf5 "Gin and Tonic"\cf0 ,\par ??\tab \tab \tab \tab Ingredients = \cf1 new\cf0 [] \{ \cf5 "Gin"\cf0 , \cf5 "Tonic"\cf0  \}\par ??\tab \tab \tab \}\par ??\tab \};\par ??\par ??\tab \cf1 public\cf0  \cf4 IEnumerable\cf0 &lt;\cf4 Cocktail\cf0 &gt; GetCocktails()\par ??\tab \{\par ??\tab \tab \cf1 return\cf0  cocktails;\par ??\tab \}\par ??\par ??\tab \cf1 public\cf0  \cf4 IEnumerable\cf0 &lt;\cf4 Cocktail\cf0 &gt; GetCocktailsSimilarTo(\cf4 Cocktail\cf0  cocktail)\par ??\tab \{\par ??\tab \tab \cf1 return\cf0  \cf1 from\cf0  c \cf1 in\cf0  cocktails\par ??\tab \tab        \cf1 where\cf0  c.Ingredients.Intersect(cocktail.Ingredients).Count() &gt;= 2 &amp;&amp; c.Id != cocktail.Id\par ??\tab \tab        \cf1 select\cf0  c;\par ??\tab \}\par ??\}} --></p>
<div class="code" style="background: white none repeat scroll 0% 0%; font-family: Courier New; font-size: 10pt; color: black; -moz-background-clip: -moz-initial; -moz-background-origin: -moz-initial; -moz-background-inline-policy: -moz-initial;">
<p style="margin: 0px;"><span style="color: blue;">public</span> <span style="color: blue;">class</span> <span style="color: #2b91af;">CocktailService</span></p>
<p style="margin: 0px;">{</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">private</span> <span style="color: blue;">readonly</span> <span style="color: #2b91af;">List</span>&lt;<span style="color: #2b91af;">Cocktail</span>&gt; cocktails = <span style="color: blue;">new</span> <span style="color: #2b91af;">List</span>&lt;<span style="color: #2b91af;">Cocktail</span>&gt;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">new</span> <span style="color: #2b91af;">Cocktail</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Id = 1,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Name = <span style="color: #a31515;">"Black Russian"</span>,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Ingredients = <span style="color: blue;">new</span> [] { <span style="color: #a31515;">"Vodka"</span>, <span style="color: #a31515;">"Kahlua"</span> }</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; },</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">new</span> <span style="color: #2b91af;">Cocktail</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Id = 2,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Name = <span style="color: #a31515;">"White Russian"</span>,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Ingredients = <span style="color: blue;">new</span>[] { <span style="color: #a31515;">"Vodka"</span>, <span style="color: #a31515;">"Cream"</span>, <span style="color: #a31515;">"Kahlua"</span> }</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; },</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">new</span> <span style="color: #2b91af;">Cocktail</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Id = 3,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Name = <span style="color: #a31515;">"Gin and Tonic"</span>,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Ingredients = <span style="color: blue;">new</span>[] { <span style="color: #a31515;">"Gin"</span>, <span style="color: #a31515;">"Tonic"</span> }</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; };</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: #2b91af;">IEnumerable</span>&lt;<span style="color: #2b91af;">Cocktail</span>&gt; GetCocktails()</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">return</span> cocktails;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: #2b91af;">IEnumerable</span>&lt;<span style="color: #2b91af;">Cocktail</span>&gt; GetCocktailsSimilarTo(<span style="color: #2b91af;">Cocktail</span> cocktail)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">return</span> <span style="color: blue;">from</span> c <span style="color: blue;">in</span> cocktails</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp; <span style="color: blue;">where</span> c.Ingredients.Intersect(cocktail.Ingredients).Count() &gt;= 2 &amp;&amp; c.Id != cocktail.Id</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp; <span style="color: blue;">select</span> c;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">}</p>
</div>
<p>We also need an implementation of ICommand, I've gone with a very simple DelegateCommand which simply has an executed command invoke a method on the ViewModel.</p>
<p><!-- {\rtf1\ansi\ansicpg\lang1024\noproof1252\uc1 \deff0{\fonttbl{\f0\fnil\fcharset0\fprq1 Courier New;}}{\colortbl;??\red0\green0\blue255;\red255\green255\blue255;\red0\green0\blue0;\red43\green145\blue175;\red163\green21\blue21;}??\fs20 \cf1 public\cf0  \cf1 class\cf0  \cf4 DelegateCommand\cf0 &lt;T&gt; : \cf4 ICommand\par ??\cf0 \{\par ??\tab \cf1 public\cf0  \cf1 event\cf0  \cf4 EventHandler\cf0  CanExecuteChanged;\par ??\par ??\tab \cf1 private\cf0  \cf1 readonly\cf0  \cf4 Action\cf0 &lt;T&gt; action;\par ??\tab \cf1 private\cf0  \cf1 readonly\cf0  \cf4 Func\cf0 &lt;T, \cf1 bool\cf0 &gt; predicate;\par ??\par ??\tab \cf1 public\cf0  DelegateCommand(\cf4 Action\cf0 &lt;T&gt; action)\par ??\tab \{\par ??\tab \tab \cf1 if\cf0 (action == \cf1 null\cf0 )\par ??\tab \tab \tab \cf1 throw\cf0  \cf1 new\cf0  \cf4 ArgumentNullException\cf0 (\cf5 "action"\cf0 );\par ??\par ??\tab \tab \cf1 this\cf0 .action = action;\par ??\tab \tab predicate = t =&gt; \cf1 true\cf0 ;\par ??\tab \}\par ??\par ??\tab \cf1 public\cf0  DelegateCommand(\cf4 Action\cf0 &lt;T&gt; action, \cf4 Func\cf0 &lt;T, \cf1 bool\cf0 &gt; predicate)\par ??\tab \{\par ??\tab \tab \cf1 if\cf0 (action == \cf1 null\cf0 )\par ??\tab \tab \tab \cf1 throw\cf0  \cf1 new\cf0  \cf4 ArgumentNullException\cf0 (\cf5 "action"\cf0 );\par ??\par ??\tab \tab \cf1 if\cf0 (predicate == \cf1 null\cf0 )\par ??\tab \tab \tab \cf1 throw\cf0  \cf1 new\cf0  \cf4 ArgumentNullException\cf0 (\cf5 "predicate"\cf0 );\par ??\par ??\tab \tab \cf1 this\cf0 .action = action;\par ??\tab \tab \cf1 this\cf0 .predicate = predicate;\par ??\tab \}\par ??\par ??\tab \cf1 protected\cf0  \cf1 virtual\cf0  \cf1 void\cf0  OnCanExecuteChanged(\cf4 EventArgs\cf0  e)\par ??\tab \{\par ??\tab \tab \cf1 var\cf0  canExecuteChanged = CanExecuteChanged;\par ??\par ??\tab \tab \cf1 if\cf0 (canExecuteChanged != \cf1 null\cf0 )\par ??\tab \tab \tab canExecuteChanged(\cf1 this\cf0 , e);\par ??\tab \}\par ??\par ??\tab \cf1 public\cf0  \cf1 void\cf0  RaiseCanExecuteChanged()\par ??\tab \{\par ??\tab \tab OnCanExecuteChanged(\cf4 EventArgs\cf0 .Empty);\par ??\tab \}\par ??\par ??\tab \cf1 public\cf0  \cf1 bool\cf0  CanExecute(\cf1 object\cf0  parameter)\par ??\tab \{\par ??\tab \tab \cf1 return\cf0  predicate((T)parameter);\par ??\tab \}\par ??\par ??\tab \cf1 public\cf0  \cf1 void\cf0  Execute(\cf1 object\cf0  parameter)\par ??\tab \{\par ??\tab \tab action((T)parameter);\par ??\tab \}\par ??\}} --></p>
<div class="code" style="background: white none repeat scroll 0% 0%; font-family: Courier New; font-size: 10pt; color: black;">
<p style="margin: 0px;"><span style="color: blue;">public</span> <span style="color: blue;">class</span> <span style="color: #2b91af;">DelegateCommand</span>&lt;T&gt; : <span style="color: #2b91af;">ICommand</span></p>
<p style="margin: 0px;">{</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: blue;">event</span> <span style="color: #2b91af;">EventHandler</span> CanExecuteChanged;</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">private</span> <span style="color: blue;">readonly</span> <span style="color: #2b91af;">Action</span>&lt;T&gt; action;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">private</span> <span style="color: blue;">readonly</span> <span style="color: #2b91af;">Func</span>&lt;T, <span style="color: blue;">bool</span>&gt; predicate;</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> DelegateCommand(<span style="color: #2b91af;">Action</span>&lt;T&gt; action)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">if</span>(action == <span style="color: blue;">null</span>)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">throw</span> <span style="color: blue;">new</span> <span style="color: #2b91af;">ArgumentNullException</span>(<span style="color: #a31515;">"action"</span>);</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">this</span>.action = action;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; predicate = t =&gt; <span style="color: blue;">true</span>;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> DelegateCommand(<span style="color: #2b91af;">Action</span>&lt;T&gt; action, <span style="color: #2b91af;">Func</span>&lt;T, <span style="color: blue;">bool</span>&gt; predicate)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">if</span>(action == <span style="color: blue;">null</span>)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">throw</span> <span style="color: blue;">new</span> <span style="color: #2b91af;">ArgumentNullException</span>(<span style="color: #a31515;">"action"</span>);</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">if</span>(predicate == <span style="color: blue;">null</span>)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">throw</span> <span style="color: blue;">new</span> <span style="color: #2b91af;">ArgumentNullException</span>(<span style="color: #a31515;">"predicate"</span>);</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">this</span>.action = action;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">this</span>.predicate = predicate;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">protected</span> <span style="color: blue;">virtual</span> <span style="color: blue;">void</span> OnCanExecuteChanged(<span style="color: #2b91af;">EventArgs</span> e)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">var</span> canExecuteChanged = CanExecuteChanged;</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">if</span>(canExecuteChanged != <span style="color: blue;">null</span>)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; canExecuteChanged(<span style="color: blue;">this</span>, e);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: blue;">void</span> RaiseCanExecuteChanged()</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; OnCanExecuteChanged(<span style="color: #2b91af;">EventArgs</span>.Empty);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: blue;">bool</span> CanExecute(<span style="color: blue;">object</span> parameter)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">return</span> predicate((T)parameter);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: blue;">void</span> Execute(<span style="color: blue;">object</span> parameter)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; action((T)parameter);</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">}</p>
</div>
<p>Now we have all the pieces to build our new ViewModel. As before we expose our ObservableCollection of AvailableCocktails which is populated on construction, we'll now elso expose a similar collection of SimilarCocktails. Now the new stuff, we'll create a private method that takes a cocktail and fills the SimilarCocktails with ones similar to the parameter. We then wrap this method with a DelegateCommand and expose it via an ICommand property.</p>
<p><!-- {\rtf1\ansi\ansicpg\lang1024\noproof65001\uc1 \deff0{\fonttbl{\f0\fnil\fcharset0\fprq1 Courier New;}}{\colortbl;??\red0\green0\blue255;\red255\green255\blue255;\red0\green0\blue0;\red43\green145\blue175;}??\fs20 \cf1 public\cf0  \cf1 class\cf0  \cf4 CocktailsViewModel\cf0  : \cf4 ViewModelBase\cf0 &lt;\cf4 CocktailsViewModel\cf0 &gt;\par ??\{\par ??\tab \cf1 private\cf0  \cf1 readonly\cf0  \cf4 CocktailService\cf0  cocktailService = \cf1 new\cf0  \cf4 CocktailService\cf0 ();\par ??\par ??\tab \cf1 public\cf0  CocktailsViewModel()\par ??\tab \{\par ??\tab \tab AvailableCocktails = \cf1 new\cf0  \cf4 ObservableCollection\cf0 &lt;\cf4 Cocktail\cf0 &gt;();\par ??\tab \tab SimilarCocktails = \cf1 new\cf0  \cf4 ObservableCollection\cf0 &lt;\cf4 Cocktail\cf0 &gt;();\par ??\par ??\tab \tab GetSimilarCocktailsCommand = \cf1 new\cf0  \cf4 DelegateCommand\cf0 &lt;\cf4 Cocktail\cf0 &gt;(GetSimilarCocktails);\par ??\par ??\tab \tab AvailableCocktails.AddRange(cocktailService.GetCocktails());\par ??\tab \}\par ??\par ??\tab \cf1 public\cf0  \cf4 ObservableCollection\cf0 &lt;\cf4 Cocktail\cf0 &gt; AvailableCocktails\par ??\tab \{\par ??\tab \tab \cf1 get\cf0 ;\par ??\tab \tab \cf1 set\cf0 ;\par ??\tab \}\par ??\par ??\tab \cf1 public\cf0  \cf4 ObservableCollection\cf0 &lt;\cf4 Cocktail\cf0 &gt; SimilarCocktails\par ??\tab \{\par ??\tab \tab \cf1 get\cf0 ; \cf1 set\cf0 ;\par ??\tab \}\par ??\par ??\tab \cf1 public\cf0  \cf4 ICommand\cf0  GetSimilarCocktailsCommand\par ??\tab \{\par ??\tab \tab \cf1 get\cf0 ; \cf1 private\cf0  \cf1 set\cf0 ;\par ??\tab \}\par ??\par ??\tab \cf1 protected\cf0  \cf1 void\cf0  GetSimilarCocktails(\cf4 Cocktail\cf0  cocktail)\par ??\tab \{\par ??\tab \tab SimilarCocktails.Replace(cocktailService.GetCocktailsSimilarTo(cocktail));\par ??\tab \}\par ??\}} --></p>
<div class="code" style="background: white none repeat scroll 0% 0%; font-family: Courier New; font-size: 10pt; color: black;">
<p style="margin: 0px;"><span style="color: blue;">public</span> <span style="color: blue;">class</span> <span style="color: #2b91af;">CocktailsViewModel</span> : <span style="color: #2b91af;">ViewModelBase</span>&lt;<span style="color: #2b91af;">CocktailsViewModel</span>&gt;</p>
<p style="margin: 0px;">{</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">private</span> <span style="color: blue;">readonly</span> <span style="color: #2b91af;">CocktailService</span> cocktailService = <span style="color: blue;">new</span> <span style="color: #2b91af;">CocktailService</span>();</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> CocktailsViewModel()</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; AvailableCocktails = <span style="color: blue;">new</span> <span style="color: #2b91af;">ObservableCollection</span>&lt;<span style="color: #2b91af;">Cocktail</span>&gt;();</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; SimilarCocktails = <span style="color: blue;">new</span> <span style="color: #2b91af;">ObservableCollection</span>&lt;<span style="color: #2b91af;">Cocktail</span>&gt;();</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; GetSimilarCocktailsCommand = <span style="color: blue;">new</span> <span style="color: #2b91af;">DelegateCommand</span>&lt;<span style="color: #2b91af;">Cocktail</span>&gt;(GetSimilarCocktails);</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; AvailableCocktails.AddRange(cocktailService.GetCocktails());</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: #2b91af;">ObservableCollection</span>&lt;<span style="color: #2b91af;">Cocktail</span>&gt; AvailableCocktails</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">get</span>;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">set</span>;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: #2b91af;">ObservableCollection</span>&lt;<span style="color: #2b91af;">Cocktail</span>&gt; SimilarCocktails</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">get</span>; <span style="color: blue;">set</span>;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">public</span> <span style="color: #2b91af;">ICommand</span> GetSimilarCocktailsCommand</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span style="color: blue;">get</span>; <span style="color: blue;">private</span> <span style="color: blue;">set</span>;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: blue;">protected</span> <span style="color: blue;">void</span> GetSimilarCocktails(<span style="color: #2b91af;">Cocktail</span> cocktail)</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; {</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; SimilarCocktails.Replace(cocktailService.GetCocktailsSimilarTo(cocktail));</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; }</p>
<p style="margin: 0px;">}</p>
</div>
<p>Excellent, our ViewModel is complete, notice we've done all the functionality of the page without worrying about how the UI will look or feel. I'm going to assume that while I'm building this our designer is building a lovely interface for our application (hopefully using some of theSketchFlow and SampleData functionality in Blend). The screencast shows how to hook up the View to the ViewModel using Blend.</p>
<div id="silverlightControlHost">
<object width="600" height="450" data="data:application/x-silverlight," type="application/x-silverlight">
<param name="source" value="/clientbin/mediaplayer.xap" />
<param name="autoUpgrade" value="true" />
<param name="minRuntimeVersion" value="3.0.40624.0" />
<param name="enableHtmlAccess" value="true" />
<param name="enableGPUAcceleration" value="true" />
<param name="initparams" value="playerSettings =                          &lt;Playlist&gt;                             &lt;AutoLoad&gt;true&lt;/AutoLoad&gt;                             &lt;AutoPlay&gt;false&lt;/AutoPlay&gt;                             &lt;DisplayTimeCode&gt;false&lt;/DisplayTimeCode&gt;                             &lt;EnableCachedComposition&gt;true&lt;/EnableCachedComposition&gt;                             &lt;EnableCaptions&gt;true&lt;/EnableCaptions&gt;                             &lt;EnableOffline&gt;true&lt;/EnableOffline&gt;                             &lt;EnablePopOut&gt;true&lt;/EnablePopOut&gt;                             &lt;StartMuted&gt;false&lt;/StartMuted&gt;                             &lt;StretchMode&gt;None&lt;/StretchMode&gt;                             &lt;Items&gt; 								&lt;PlaylistItem&gt; 									&lt;AudioCodec&gt;&lt;/AudioCodec&gt; 									&lt;Description&gt;&lt;/Description&gt; 									&lt;FileSize&gt;2797299&lt;/FileSize&gt; 									&lt;FrameRate&gt;15.000015000015&lt;/FrameRate&gt; 									&lt;Height&gt;600&lt;/Height&gt; 									&lt;IsAdaptiveStreaming&gt;false&lt;/IsAdaptiveStreaming&gt; 									&lt;MediaSource&gt;http://compiledexperience.com/content/video/BlendableViewModel.2.wmv&lt;/MediaSource&gt; 									&lt;ThumbSource&gt;http://compiledexperience.com/content/video/BlendableViewModel.2_Thumb.jpg&lt;/ThumbSource&gt; 									&lt;Title&gt;Blendable%20View%20Model%20Databinding&lt;/Title&gt; 									&lt;VideoCodec&gt;VC1&lt;/VideoCodec&gt; 									&lt;Width&gt;960&lt;/Width&gt; 								&lt;/PlaylistItem&gt;                             &lt;/Items&gt;                         &lt;/Playlist&gt;" />
<div onmouseover="highlightDownloadArea(true)" onmouseout="highlightDownloadArea(false)"><img style="border-style: none; position: absolute; width: 100%; height: 100%;" src="/content/video/BlendableViewModel.2_Thumb.jpg" alt="" /> 
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
<p>If you want to download the source of the project you can find it <a href="/content/downloads/CompiledExperience.BlendableViewModel.zip">here</a>, (it does contain some of the code for the next post).</p>
<p><a rev="vote-for" href="http://dotnetshoutout.com/Blendable-MVVM-Commands-and-Behaviors-Blog-Compiled-Experience-Silverlight-Development"><img style="border:0px" src="http://dotnetshoutout.com/image.axd?url=http%3A%2F%2Fcompiledexperience.com%2Fblog%2Fposts%2FBlendable-MVVM-Commands-and-Behaviors" alt="Shout it" /></a> <a href="http://www.dotnetkicks.com/kick/?url=http%3a%2f%2fcompiledexperience.com%2fblog%2fposts%2fBlendable-MVVM-Commands-and-Behaviors"><img src="http://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fcompiledexperience.com%2fblog%2fposts%2fBlendable-MVVM-Commands-and-Behaviors" border="0" alt="kick it on DotNetKicks.com" /></a></p>
