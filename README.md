# mvc-5-routing-localization
Simple localization and translation for both conventional and attribute routing. 

# Features

Normal `Route("~/invest")` automatically gets a language prefix, such as `/invest` for default language, and e.g. `/de/invest` for german language.  
You can achieve the same for conventional routing.

For more control use LocalizeRoute("..."), so you can get `/de/investieren`, or disable route localization.

The effect is that you get Thread.CurrentThread.CurrentCulture set appropriately during controller initialization.

# Setup

Let's say we have a couple of languages we want to support, stored in application config, accessible like his:  
ConfigurationManager.AppSettings["SupportedCultures"] -> **"en,de"**

We have a default culture like so:  
ConfigurationManager.AppSettings["DefaultCulture"] -> **"en"**'

Have a resource files for these languages containing url path segments translated, e.g. for key 'invest' have it with default value 'invest', and in german resource file as 'investieren'

Add global filter:  
```
filters.Add(new CultureFilter(ConfigurationManager.AppSettings["DefaultCulture"]);
```

# Conventional routing

```
routes.MapRoute(
  name: "DefaultWithCulture",
  url: "{culture}/{controller}/{action}/{id}",
  defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional },
  constraints: new { culture = new CultureConstraint(defaultCulture: ConfigurationManager.AppSettings["DefaultCulture"]) }
  );

routes.MapRoute(
	name: "Default",
	url: "{controller}/{action}/{id}",
	defaults: new { culture = DefaultCulture, controller = "Home", action = "Index", id = UrlParameter.Optional }
);
```

Do this in same order in areas route registration too.

# Attribute routing

In order for conventional and attribute routing to work together, initialize them in this order in global.asax.cs :

```
RouteTable.Routes.MapLocalizedMvcAttributeRoutes();
AreaRegistration.RegisterAllAreas();
RouteConfig.RegisterRoutes(RouteTable.Routes);
```

By default, all normal routes are localized by adding a culture prefix e.g. for
```
[Route("~/invest")]
```
you have 2 routes generated:  
~/{culture}/invest  
~/invest (this one has a default route value of {culture} -> 'en')

To have more control, use **LocalizedRoute**, usually for 2 scenarios:
### 1. Translate url path
```
[LocalizedRoute("~/invest")]
```
generates these routes:  
~/de/investieren  
~/invest

### 2. Have a route that does not get neither prefix-localized or translated, e.g. to permanent-redirect old urls to new ones for SEO reasons.
For example if you need to redirect /investing to /invest
```
[LocalizeRoute("~/investing", explicitCulture: "en")
ActionResult Index_Old()
{
	return RedirectToActionPermanent("Index");
}
```

or just the english url:
```
[LocalizeRoute("~/investing", translateUrl: false, explicitCulture: "en")
ActionResult Index_Old_en()
{
	return RedirectToActionPermanent("Index");
}
```

Ideas from post https://stackoverflow.com/questions/32764989/asp-net-mvc-5-culture-in-route-and-url