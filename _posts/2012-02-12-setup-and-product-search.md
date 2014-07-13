---
layout: post
title: Setup and Product Search
tags: windows-phone tutorial
category: windows-phone-tutorial
permalink: '/windows-phone/tutorials/setup-and-product-search'
---

<span class="alignleft"><img src="/content/images/tutorials/setup-and-product-search.png" alt="Setup and Product Search"/></span>
An overview of the series, creating the initial skeleton for the project and building product search functionality. This includes building the nuget packages, setting up Caliburn.Micro and using Reactive Extensions.

###Introduction
For a while now I've wanted to push a lot of the code I use in Windows Phone apps into a publicly available library, this tutorial is the start of that process. One thing I didn't want to do was create another control library but rather a slightly opinionated (my opinion) approach to building Windows Phone applications. It will be built to take hard dependencies on certain libraries and also try to remove the developer friction in creating applications. During the process we'll create an actual application so you can also see some of the things you'll need to consider building a full application. Also in some the tutorials I'll cover the workflow I use when approaching a new page in the application.

One note about this tutorial series is that I'm going to cover a lot of concepts very quickly or just assume you know something, for instance this tutorial covers dependency injection using the container provided in Caliburn Micro. I highly recommend reading the supporting documentation for any library mentioned here as I'd like to focus on what it takes to put together a full market ready application.

The foundation of this toolkit is the fantastic framework [Caliburn Micro][1], we’ll also make heavy use of [Reactive Extensions][3] and for our REST operations [Hammock][2] and [Json.NET][5]. In order to reduce some friction in development we'll use the AOP toolkit [PostSharp][4].

The toolkit will be available through [NuGet][6] and the source code available on [Bit Bucket][7] including the example application, since the repository will contain the latest version of the application the source code may not line up perfectly with the tutorial, I will however endeavor to tag it when each tutorial was released. The toolkit will be broken up into multiple assemblies, the main reason for this is due to the Windows Phone capabilities model. One thing I really don't like is creating a dependency on a third party assembly that ends up adding capabilities to the application that could possibly give an incorrect view of what the application does.

<span class="alignleft"><img src="/content/images/tutorials/toolkit/yealands.png" alt="Yealands"/></span>

###What are we building?
I spent a couple of years living in Ontario, Canada (and in fact married a lovely Canadian girl). One of the interesting things I found about the province is that all alcohol sales is controlled through the Liquor Control Board of Ontario, they sell to the public through two different chains of stores known as "The Beer Store" and the "LCBO". While I'm not exactly a fan of such government control is done provide one interesting benefit, they can now provide all the data you need for the purchase of liquor in Ontario. Frustratingly they don't actually provide that data in a consumable format. Thankfully Carsten Nielson from Unspace Interactive has put together the LCBO REST API which you can browse at the web site www.lcboapi.com. The app we'll be building will provide a metro style user interface over the functionality exposed by the API.

What we're going to cover in this tutorial is setting up the project and creating a simple product search page, we'll be revisiting the pages we create early on in later tutorials to expand or refine them.

###Lets get started
Let's fire up Visual Studio and create a new **Windows Phone Application Project**, my usual step here is pretty much clear out the project. I delete the following:

 - MainPage.xaml
 - Everything in App.xaml except the Application element and it's namespace declarations.
 - All the code in App.xaml.cs except the constructor and the call to InitiaizeComponent.

We should now have a bare bones Windows Phone application, with no pages and no set up code. Fire up NuGet and add the following packages (if you haven't used NuGet before I suggest the following [tutorial][1]):

 - **Caliburn Micro** - I usually delete everything the package adds to the project by default, we'll be creating these ourselves.
 - **Hammock** - This is the REST client library we'll be using to talk to the LCBO API.
 - **Json.NET** - We'll be using this to deserialise requests from the API.
 - **Compiled Experience Phone Toolkit** - Wouldn't be much of a toolkit if we didn't need it ...
 - **Compiled Experience Phone Toolkit REST** - The REST Client capabilities for the CX Phone Toolkit.

### Setting up the Services
For most projects like this I like to work from the inside out, building the services first, then the view model and finally the view. In the project create a Services folder that will hold all the functionality necessary to interact with the API. We'll most likely break this down into difference services for different pieces of functionality, however first I need to create the types that will map to the returned json from the API. To be able to search products we'll need two, a generic ServiceResult type and a Product type. To cut down on friction we'll only map the types we actually need, and we'll use attributes from Json.NET to handle the differences in naming conventions between the API and the Product class.

``` csharp
public class ServiceResult<T> 
{     
	public int Status     
	{         
		get; set;     
	}

	public string Message     
	{      
		get; set;     
	}       

	public T Result     
	{         
		get; set;     
	} 
} 
```

``` csharp
public class Product 
{     
	public int Id     
	{         
		get; set;     
	}       

	public string Name     
	{         
		get; set;     
	}       

	public string Category     
	{         
		get         
		{             
			var categories = new[]             
			{                 
				PrimaryCategory, SecondaryCategory             
			};               

			return String.Join(" ", categories.Where(c => !String.IsNullOrEmpty(c)));         
		}     
	}       

	[JsonProperty("primary_catgegory")]     
	public string PrimaryCategory     
	{         
		get; set;     
	}       

	[JsonProperty("secondary_category")]     
	public string SecondaryCategory     
	{         
		get; set;     
	} 
}
```

Over the course of the tutorials we'll be creating a number of services, but for the moment we'll only need a ProductService, I have however created a ServiceBase type which will handle the setup of our RestClient (if you'd like to read more about Hammock check it out on [GitHub][2]). The product search is pretty simple, but we're taking advantage of a feature in the toolkit to turn the request into an Observable and then a projection on the result to just return the list of products that match the search. To better separate our view models from the services I'll extra an interface from the Product Service. This will also help if you want to unit test your view models.

``` csharp
public class ServiceBase
{
    private readonly RestClient client = new RestClient
    {
        Authority = "http://lcboapi.com",
        Deserializer = new JsonDeserializer()
    };
 
    protected RestClient Client
    {
        get { return client; }
    }
}
```

``` csharp
public class ProductService : ServiceBase, IProductService
{
    public IObservable<IEnumerable<Product>> Search(string query)
    {
        var request = new RestRequest
        {
            Path = "products"
        };
 
        request.AddParameter("q", query);
        request.AddParameter("where_not", "is_dead");
 
        return Client.ToObservable<ServiceResult<IEnumerable<Product>>>(request)
            .Select(r => r.Result);
    }
 
    public IObservable<IEnumerable<Product>> Latest(int limit)
    {
        var request = new RestRequest
        {
            Path = "products"
        };
 
        request.AddParameter("per_page", limit.ToString());
        request.AddParameter("where_not", "is_dead");
        request.AddParameter("order", "released_on.desc");
 
        return Client.ToObservable<ServiceResult<IEnumerable<Product>>>(request)
            .Select(r => r.Result);
    }
}
```

###Caliburn Micro and creating our View Models
While I'll be endeavoring to explain a little bit about [Caliburn Micro][1] as I go this won't be an in-depth discussion on how it works. For a longer series of tutorials I'd recommend the [documentation][1].  The configuration of the Caliburn is driven by the Bootstrapper object, which is added to the application's resource collection in App.xaml. Normally we'd need to have to put some boilerplate code into our Bootstrapper for the setup of the Phone Container and configuration of the Phone Services. The toolkit contains one already setup so inheriting from *CompiledExperience.Phone.Toolkit.Framework.ApplicationBootstrapper* speeds you up a bit. All we need to do at the moment is to add the Product Service to the container. Once we've done that we'll also need to add an instance of the type to App.xaml.

``` csharp
public class Bootstrapper : ApplicationBootstrapper 
{     
	protected override void Configure()    
	{         
		base.Configure();           

		Container.Singleton<IProductService, ProductService>();           
		Container.PerRequest<ProductSearchViewModel>();     
	} 
}
```

``` xml
<Application
   x:Class="CompiledExperience.Phone.Demo.App"
   xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"      
   xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
   xmlns:sys="clr-namespace:System;assembly=mscorlib"
   xmlns:app="clr-namespace:CompiledExperience.Phone.Demo">
 
    <Application.Resources>
        <app:Bootstrapper x:Key="Bootstrapper"/>
    </Application.Resources>
 
</Application>
```

So now Caliburn is set up we can create our first view model, create a ViewModels folder in the project and to that add a new class ProductSeachViewModel, inherit this from *CompiledExperience.Phone.Toolkit.Framework.ViewModelBase*. This adds functionality such as a co-routine for OnViewReady which will make for some much more readable and testable code.

First thing we'll do is create a method to execute our search and populate the results. We'll create this as a co-routine to easily lay out our code. Initially we'll clear out any existing search results, in order to show the user that we're doing some work we first return a ProgessIndicatorResult, this is always a good practise because if the service is over a slow connection the user may not think anything is happening. We then perform the search and get the observable for the search, we then pass this to an Observable result, what this IResult does is take the custom subscription method, and then waits till the observable sequence as been completed for passing on to the next IResult. The subscribe method on the result takes the values and adds them to the view model. Once that's complete we clear our Progress Indicator and we're done.

``` csharp
public class ProductSearchViewModel : ViewModelBase
{     
	private readonly IProductService productService;       

	public ProductSearchViewModel(IProductService productService)     
	{         
		this.productService = productService;           

		Products = new BindableCollection<Product>();     
	}       

	public IObservableCollection<Product> Products     
	{         
		get; private set;     
	}       

	public IEnumerable<IResult> Search(string query)     
	{         
		Products.Clear();           

		yield return new ProgressResult(true, "Searching");           

		var products = productService.Search(query).SelectMany(p => p);           

		yield return new ObservableResult<Product>(products, p => Products.Add(p));           

		yield return new ProgressResult(visible: false);     
	} 
}
```


Now that our view model is complete we need to create our view. In the project create a Views folder and add ProductSearchView.xaml. By convention Caliburn will use the naming to bind together the view and view model. The view will be a fairly simple one at the moment; we'll be revisiting design during a later tutorial. We'll use a Grid to split the view into two sections, the search text box and list of results. One of the important things is to give the TextBox an InputScope of Search, this will mean the keyboard style the phone displays has the submit button (which is really just the enter key).

``` xml
<Grid x:Name="LayoutRoot" Background="Transparent">
     <Grid.RowDefinitions>
         <RowDefinition Height="Auto"/>
         <RowDefinition Height="*"/>
     </Grid.RowDefinitions>

      <StackPanel x:Name="TitlePanel" Grid.Row="0" Margin="12,17,0,28">
         <TextBlock x:Name="ApplicationTitle" Text="{StaticResource ApplicationName}" Style="{StaticResource PhoneTextNormalStyle}"/>
         <TextBlock x:Name="PageTitle" Text="product search" Margin="9,-7,0,0" Style="{StaticResource PhoneTextTitle1Style}"/>
     </StackPanel>

	 <Grid x:Name="ContentPanel" Grid.Row="1" Margin="12,0,12,0">
         <Grid.RowDefinitions>
         	<RowDefinition Height="Auto"/
         	<RowDefinition/>
         </Grid.RowDefinitions>

         <TextBox InputScope="Search" AcceptsReturn="False" />
         <ListBox x:Name="Products" Grid.Row="1" Margin="0,0,-12,0" ItemTemplate="{StaticResource ProductListTemplate}" />
     </Grid>
</Grid>
```

One of the beauties of Caliburn's action system is that it's ultimately built on the Expression Blend Trigger, Action and Behavior system, this gives you the flexibility to create custom triggers to actions, this is something we'll need to do here as there's no easy way to trigger an action from when the user submits the search using the keyboard. Ultimately it's a pretty simple trigger, we attach to the KeyDown even on the TextBox and if that key is Enter then we trigger the attached actions. The only thing unique here is we also focus back on the Root Visual of the Application. What this does is hide the keyboard after the user hits submit, otherwise they're left with the keyboard hiding the search results. Wiring this trigger into Caliburn means using expanded syntax, but it isn't very complicated at all.

``` csharp
public class SearchTriggerAction : TriggerBase<TextBox> 
{
	protected override void OnAttached()     
	{         
		AssociatedObject.KeyDown += OnKeyDown;     
	}       

	protected override void OnDetaching()     
	{         
		AssociatedObject.KeyDown -= OnKeyDown;     
	}       

	private void OnKeyDown(object sender, KeyEventArgs e)     
	{         
		if(e.Key != Key.Enter)             
			return;           

		e.Handled = true;           
		
		InvokeActions(AssociatedObject.Text);           

		if(!(Application.Current.RootVisual is PhoneApplicationFrame))             
			return;           

		var frame = Application.Current.RootVisual as PhoneApplicationFrame;           
		frame.Focus();     
	} 
}
```

``` xml
<TextBox InputScope="Search" AcceptsReturn="False">     
	<i:Interaction.Triggers>         
		<cx:SearchTriggerAction>             
			<caliburn:ActionMessage MethodName="Search">                 
				<caliburn:Parameter Value="$eventArgs" />             
			</caliburn:ActionMessage>         
		</cx:SearchTriggerAction>     
	</i:Interaction.Triggers> 
</TextBox>
```


We now need to customise the ListBox for the results, there are a couple of ways to do this, the first is to create a view (ProductView.xaml) and let Caliburn use its conventions to locate it. I prefer to use explicit data templates for things like this and use Blends sample data feature (I'll cover this in a later tutorial). We're most likely going to be reusing this template through the application to it makes sense to create this as an application wide resource, this would normally mean adding it App.xaml, but I've found that if we just dump everything into there it'll get cluttered really quickly. What we'll do is create a number of resource dictionaries (essentially separate resource files) that will be combined into App.xaml, create a Resources folder. Unfortunately Visual Studio doesn't have a Resource Dictionary item template (I believe Expression Blend does), so just create an xml file with the .xaml file extension and replace the contents with the following.

We can then modify App.xaml to include our Templates.xaml file, in large apps I have a number of these external resource dictionaries to make it easier to divide up shared resources. The Product data template will be pretty simple at moment (we'll be revisiting it in later tutorials) binding to the Name and a computed Category property. 

``` xml
<Application.Resources>     
	<ResourceDictionary>         
		<ResourceDictionary.MergedDictionaries>             
			<ResourceDictionary Source="/CompiledExperience.Phone.Toolkit;component/Resources/Styles.xaml"/>             
			<ResourceDictionary Source="Resources/Templates.xaml"/>        
		</ResourceDictionary.MergedDictionaries>

		<app:Bootstrapper x:Key="Bootstrapper"/>
		<sys:String x:Key="ApplicationName">LCBO SEARCH</sys:String>
	</ResourceDictionary>
</Application.Resources>
```

The last thing we need to do is modify the WPManifiest.xml file to change the start page to Views/ProductSearchView.xaml.

We didn't cover a huge amount of functionality in this tutorial but the foundations are now in place to build a whole lot more very quickly. Hope you enjoyed this introduction.

### Download the Code

The code for all the tutorials in this series as well as [Compiled Experience Phone Toolkit][7] are available on Bitbucket.
[1]: http://caliburnmicro.codeplex.com/
[2]: https://github.com/danielcrenna/hammock
[3]: http://msdn.microsoft.com/en-us/library/ff431792(v=vs.92).aspx
[4]: http://www.sharpcrafters.com/
[5]: http://json.codeplex.com/
[6]: http://nuget.codeplex.com/wikipage?title=Getting%20Started
[7]: https://bitbucket.org/nigel.sampson/compiled-experience-toolkit/
