# ASP.NET

```cs
using System.Collections.Generic;
namespace SimpleApp.Models {
	public enum Color {
		Red, Green, Yellow, Purple
	};
	
	class Votes {
		private static Dictionary<Color, int> votes = new Dictionary<Color, int>();
		
		public static void RecordVote(Color color) {
			votes[color] = votes.ContainsKey(color) ? votes[color] + 1 : 1;
		}
		
		public static void ChangeVote(Color newColor, Color oldColor) {
			if (votes.ContainsKey(oldColor)) {
				votes[oldColor]--;
			}	
			RecordVote(newColor);
		}
		
		public static int GetVotes(Color color) {
			return votes.ContainsKey(color) ? votes[color] : 0;
		}
	}
}
```

```cs
namespace SimpleApp.Controllers {
	public class HomeController : Controller {
		public ActionResult Index() {
			return View();
		}
		
		[HttpPost]
		public ActionResult Index(Color color) {
			Color? oldColor = Session["color"] as Color?;
			if (oldColor != null) {
				Votes.ChangeVote(color, (Color)oldColor);
			} else {
				Votes.RecordVote(color);
			}
			ViewBag.SelectedColor = Session["color"] = color;
			return View();
		}
	}
}
```

```aspx
@using SimpleApp.Models
@{ Layout = null; }
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width" />
<title>Vote</title>
</head>
<body>
@if (ViewBag.SelectedColor == null) {
<h4>Vote for your favorite color</h4>
} else {
<h4>Change your vote from @ViewBag.SelectedColor</h4>
}
@using (Html.BeginForm()) {
@Html.DropDownList("color",
new SelectList(Enum.GetValues(typeof(Color))), "Choose a Color")
<div>
<button type="submit">Vote</button>
</div>
}
<div>
<h5>Results</h5>
<table>
<tr><th>Color</th><th>Votes</th></tr>
@foreach (Color c in Enum.GetValues(typeof(Color))) {
<tr><td>@c</td><td>@Votes.GetVotes(c)</td></tr>
}
</table>
</div>
</body>
</html>
```


## 
```cs

```


### The Contents of the Global.asax.cs File
```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;
using System.Web.Routing;
namespace SimpleApp {
	public class MvcApplication : System.Web.HttpApplication {
		protected void Application_Start() {
			AreaRegistration.RegisterAllAreas();
			RouteConfig.RegisterRoutes(RouteTable.Routes);
		}
	}
}
```



### Breaking the Debugger in the Global.asax.cs File
```cs

namespace SimpleApp {
	public class MvcApplication : System.Web.HttpApplication {
		protected void Application_Start() {
			AreaRegistration.RegisterAllAreas();
			RouteConfig.RegisterRoutes(RouteTable.Routes);
			System.Diagnostics.Debugger.Break();
		}
		
		protected void Application_End() {
			System.Diagnostics.Debugger.Break();
		}
	}
}

```

### Handling Request life-Cycle Events in the Global.asax.cs File
```cs
namespace SimpleApp {
	public class MvcApplication : System.Web.HttpApplication {
		protected void Application_Start() {
			AreaRegistration.RegisterAllAreas();
			RouteConfig.RegisterRoutes(RouteTable.Routes);
		}
		
		protected void Application_BeginRequest() {
			RecordEvent("BeginRequest");
		}
		
		protected void Application_AuthenticateRequest() {
			RecordEvent("AuthenticateRequest");
		}
		
		protected void Application_PostAuthenticateRequest() {
			RecordEvent("PostAuthenticateRequest");
		}
		
		private void RecordEvent(string name) {
			List<string> eventList = Application["events"] as List<string>;
			if (eventList == null) {
				Application["events"] = eventList = new List<string>();
			}
			eventList.Add(name);
		}
	}
}
```


### Displaying the Event Information
```cs
public class HomeController : Controller {
	public ActionResult Index() {
		return View(HttpContext.Application["events"]);
	}
}
```

```aspx
<div class="panel panel-primary">
	<h5 class="panel-heading">Events</h5>
	<table class="table table-condensed table-striped">
		@foreach (string eventName in Model) {
		<tr><td>@eventName</td></tr>
	}
	</table>
</div>
```

### Using C# Events in the Global.asax.cs File
```cs
namespace SimpleApp {
	public class MvcApplication : System.Web.HttpApplication {
	
		public MvcApplication() {
			PostAcquireRequestState += (src, args) => CreateTimeStamp();
		}
		
		protected void Application_Start() {
			AreaRegistration.RegisterAllAreas();
			RouteConfig.RegisterRoutes(RouteTable.Routes);
			CreateTimeStamp();
		}
		
		private void CreateTimeStamp() {
			string stamp = Context.Timestamp.ToLongTimeString();
			if (Session != null) {
				Session["request_timestamp"] = stamp;
			} else {
				Application["app_timestamp"] = stamp;
			}
		}
	}
}
```


```cs
namespace SimpleApp.Controllers {
	public class HomeController : Controller {
		public ActionResult Index() {
			return View(GetTimeStamps());
		}

		private List<string> GetTimeStamps() {
			return new List<string> {
				string.Format("App timestamp: {0}",
					HttpContext.Application["app_timestamp"]),
				string.Format("Request timestamp: {0}", Session["request_timestamp"]),
			};
		}
```

## Module

### ASP.NET Modules		
#### Contents of the TimerModule.cs File
```cs
namespace SimpleApp.Infrastructure {
	public class TimerModule : IHttpModule {
	
		private Stopwatch timer;
	
		//Setting Up the Event Handlers
		public void Init(HttpApplication app) {
			app.BeginRequest += HandleEvent;
			app.EndRequest += HandleEvent;
		}
		
		//Handling the BeginRequest & EndRequest Event
		private void HandleEvent(object src, EventArgs args) {
			HttpContext ctx = HttpContext.Current;
			
			if (ctx.CurrentNotification == RequestNotification.BeginRequest) {
				timer = Stopwatch.StartNew();
			} else {
				ctx.Response.Write(string.Format(
					"<div class='alert alert-success'>Elapsed: {0:F5} seconds</div>",
					((float) timer.ElapsedTicks) / Stopwatch.Frequency));
			}
		}
		
		public void Dispose() {
		// do nothing - no resources to release
		}
	}
}
```		
		
#### Registering a Module	
```xml	
//Registering a Module in the Web.config File
<?xml version="1.0" encoding="utf-8"?>
<configuration>
	<system.webServer>
	<modules>
		<add name="Timer" type="SimpleApp.Infrastructure.TimerModule"/>
	</modules>
	</system.webServer>
</configuration>
```

### Creating Self-registering Modules


#### Creating the Module
```cs
namespace CommonModules {
	public class InfoModule : IHttpModule {
	
		public void Init(HttpApplication app) {
			app.EndRequest += (src, args) => {
				HttpContext ctx = HttpContext.Current;
					ctx.Response.Write(string.Format(
						"<div class='alert alert-success'>URL: {0} Status: {1}</div>",
						ctx.Request.RawUrl, ctx.Response.StatusCode));
			};
		}
		public void Dispose() {
			// do nothing - no resources to release
		}
	}
}


#### Creating the Registration Class
```cs
using System.Web;
[assembly: PreApplicationStartMethod(typeof(CommonModules.ModuleRegistration),"RegisterModule")]
namespace CommonModules {
	public class ModuleRegistration {
		public static void RegisterModule() {
			HttpApplication.RegisterModule(typeof(CommonModules.InfoModule));
		}
	}
}
```

### Using Module Events
#### Defining the Module Event

### Understanding the Built-in Modules

## Handler
```cs
//DayOfWeekHandler.cs
namespace SimpleApp.Infrastructure {
	public class DayOfWeekHandler: IHttpHandler {
		public void ProcessRequest(HttpContext context) {
		
			string day = DateTime.Now.DayOfWeek.ToString();
			
			if (context.Request.CurrentExecutionFilePathExtension == ".json") {
				context.Response.ContentType = "application/json";
				context.Response.Write(string.Format("{{\"day\": \"{0}\"}}", day));
			} else {
				context.Response.ContentType = "text/html";
				context.Response.Write(string.Format("<span>It is: {0}</span>", day));
			}
		}
		
		public bool IsReusable {
			get { return false; }
		}
	}
}
```

### Registering a Handler Using URL Routing
```cs
//Setting Up a Route for a Custom Handler in the RouteConfig.cs File
namespace SimpleApp {
	public class RouteConfig {
		public static void RegisterRoutes(RouteCollection routes) {
			routes.IgnoreRoute("{resource}.axd/{*pathInfo}");
			
			routes.Add(new Route("handler/{*path}",
			new CustomRouteHandler {HandlerType = typeof(DayOfWeekHandler)}));
			
			routes.MapRoute(
				name: "Default",
				url: "{controller}/{action}/{id}",
				defaults: new { controller = "Home", action = "Index",
				id = UrlParameter.Optional }
				);
			}
		}
		
	class CustomRouteHandler : IRouteHandler {
		public Type HandlerType { get; set; }
		
		public IHttpHandler GetHttpHandler(RequestContext requestContext) {
			return (IHttpHandler)Activator.CreateInstance(HandlerType);
		}
	}
}
```

### Registering a Handler Using the Configuration File
```cs
//Registering a Handler in the Web.config File
<?xml version="1.0" encoding="utf-8"?>
<configuration>
	<system.webServer>
	<handlers>
		<add name="DayJSON" path="/handler/*.json" verb="GET" type="SimpleApp.Infrastructure.DayOfWeekHandler"/>
		<add name="DayHTML" path="/handler/day.html" verb="*" type="SimpleApp.Infrastructure.DayOfWeekHandler"/>
	</handlers>
	</system.webServer>
</configuration>
```

### Ignoring Requests for the Custom handler in the RouteConfig.cs File
```cs
namespace SimpleApp {
	public class RouteConfig {
		public static void RegisterRoutes(RouteCollection routes) {
			routes.IgnoreRoute("{resource}.axd/{*pathInfo}");
			
			//routes.Add(new Route("handler/{*path}",
			// new CustomRouteHandler {HandlerType = typeof(DayOfWeekHandler)}));
			
			routes.IgnoreRoute("handler/{*path}");
			routes.MapRoute(
				name: "Default",
				url: "{controller}/{action}/{id}",
				defaults: new { controller = "Home", action = "Index",
				id = UrlParameter.Optional }
				);
			}
		}
		
		class CustomRouteHandler : IRouteHandler {
			public Type HandlerType { get; set; }
		
			public IHttpHandler GetHttpHandler(RequestContext requestContext) {
				return (IHttpHandler)Activator.CreateInstance(HandlerType);
			}
		}
}
```

### Creating Asynchronous Handlers

```cs
//The Contents of the SiteLengthHandler.cs File
using System.Net.Http;
using System.Threading.Tasks;
using System.Web;
namespace SimpleApp.Infrastructure {
	public class SiteLengthHandler : HttpTaskAsyncHandler {
		public override async Task ProcessRequestAsync(HttpContext context) {
			string data = await new HttpClient().GetStringAsync("http://www.apress.com");
			context.Response.ContentType = "text/html";
			context.Response.Write(string.Format("<span>Length: {0}</span>", data.Length));
		}
	}
}
```

```xml
//Registering the Asynchronous Handler in the Web.config File
<?xml version="1.0" encoding="utf-8"?>
<configuration>
	<system.webServer>
		<handlers>
			<add name="SiteLength" path="/handler/site" verb="*" type="SimpleApp.Infrastructure.SiteLengthHandler"/>
		</handlers>
	</system.webServer>
</configuration>
```

### Creating Modules That Provide Services to Handlers

```cs
Contents of the DayModule.cs File
using System;
using System.Web;
namespace SimpleApp.Infrastructure {
	public class DayModule : IHttpModule {
		public void Init(HttpApplication app) {
			app.BeginRequest += (src, args) => {
				app.Context.Items["DayModule_Time"] = DateTime.Now;
			};
		}
		public void Dispose() {
			// nothing to do
		}
	}
}
```

```xml
//Registering the Module in the Web.config File
<?xml version="1.0" encoding="utf-8"?>
	<configuration>
	<system.webServer>
		<modules>
			<add name="DayPrep" type="SimpleApp.Infrastructure.DayModule"/>
		</modules>
	</system.webServer>
</configuration>
```

### Consuming the Items Data
```cs
namespace SimpleApp.Infrastructure {
	public class DayOfWeekHandler : IHttpHandler {
		public void ProcessRequest(HttpContext context) {
		
			if (context.Items.Contains("DayModule_Time")
				&& (context.Items["DayModule_Time"] is DateTime)) {
					string day = ((DateTime)context.Items["DayModule_Time"]).DayOfWeek.ToString();
		
				if (context.Request.CurrentExecutionFilePathExtension == ".json") {
					context.Response.ContentType = "application/json";
					context.Response.Write(string.Format("{{\"day\": \"{0}\"}}", day));
				} else {
					context.Response.ContentType = "text/html";
					context.Response.Write(string.Format("<span>It is: {0}</span>",	day));
				}
			} else {
				context.Response.ContentType = "text/html";
				context.Response.Write("No Module Data Available");
			}
		}
		
		public bool IsReusable {
			get { return false; }
		}
	}
}
```

### Custom Handler Factories



### Detecting Device Capabilities

#### The Contents of the Browser.cshtml File

```aspx
//The ASP.NET browser capabilities properties for the iPhone
@model IEnumerable<Tuple<string, string>>
@{
Layout = null;
}
<!DOCTYPE html>
<html>
<head>
	<meta name="viewport" content="width=device-width" />
	<title>Device Capabilities</title>
	<link href="~/Content/bootstrap.min.css" rel="stylesheet" />
	<link href="~/Content/bootstrap-theme.min.css" rel="stylesheet" />
</head>
<body class="container">
	<div class="panel panel-primary">
		<div class="panel-heading">Capabilities</div>
		<table class="table table-striped table-bordered">
			<tr><th>Property</th><th>Value</th></tr>
			<tr><td>Browser</td><td>@Request.Browser.Browser</td></tr>
			<tr><td>IsMobileDevice</td><td>@Request.Browser.IsMobileDevice</td></tr>
		<tr>
			<td>MobileDeviceManufacturer</td>
			<td>@Request.Browser.MobileDeviceManufacturer</td>
		</tr>
		<tr>
			<td>MobileDeviceModel</td>
			<td>@Request.Browser.MobileDeviceModel</td>
		</tr>
		<tr>
			<td>ScreenPixelsHeight</td>
			<td>@Request.Browser.ScreenPixelsHeight</td>
		</tr>
		<tr>
			<td>ScreenPixelsWidth</td>
			<td>@Request.Browser.ScreenPixelsWidth</td>
		</tr>
		<tr><td>Version</td><td>@Request.Browser.Version</td></tr>
		</table>
	</div>
</body>
</html>
```

#### Creating a Capability Provider

```cs
//The Contents of the KindleCapabilities.cs File
using System.Web;
using System.Web.Configuration;
namespace Mobile.Infrastructure {
	public class KindleCapabilities : HttpCapabilitiesProvider {
		public override HttpBrowserCapabilities
			GetBrowserCapabilities(HttpRequest request) {

			HttpCapabilitiesDefaultProvider defaults =
				new HttpCapabilitiesDefaultProvider();
				
			HttpBrowserCapabilities caps = defaults.GetBrowserCapabilities(request);
			
			if (request.UserAgent.Contains("Kindle Fire")) {
				caps.Capabilities["Browser"] = "Silk";
				caps.Capabilities["IsMobileDevice"] = "true";
				caps.Capabilities["MobileDeviceManufacturer"] = "Amazon";
				caps.Capabilities["MobileDeviceModel"] = "Kindle Fire";
				if (request.UserAgent.Contains("Kindle Fire HD")) 
				{
					caps.Capabilities["MobileDeviceModel"] = "Kindle Fire HD";
				}
			}
			return caps;
		}
	}
}
```

```cs
//Registering the Capabilities Provider in the Global.asax.cs File
using System.Web.Configuration;
using System.Web.Mvc;
using System.Web.Routing;
using Mobile.Infrastructure;
namespace Mobile {
	public class MvcApplication : System.Web.HttpApplication {
		protected void Application_Start() {
			AreaRegistration.RegisterAllAreas();
			RouteConfig.RegisterRoutes(RouteTable.Routes);
			HttpCapabilitiesBase.BrowserCapabilitiesProvider = new KindleCapabilities();
		}
	}

}
```


