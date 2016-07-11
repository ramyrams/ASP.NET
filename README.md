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
