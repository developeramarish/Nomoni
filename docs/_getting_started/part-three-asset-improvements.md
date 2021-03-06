---
layout: doc
title:  "Part 3: Asset Improvements"
date:   2018-12-07 22:00:00
categories: beginner
published: true
order: 3

---

## Prerequisites

To start this tutorial you will need to have completed parts 1 & 2: 


- [Part 1 : Basic Web App with single module](/docs/getting-started/part-one-basic-web-app-with-single-module)
- [Part 2 : Adding a Second Module](/docs/getting-started/part-two-adding-a-second-module)

The result of which is a basic MVC .net core app with a single module that looks and plays just like the standard .net core template app.

## Outcome

The aim of this tutorial is to improve the rendering of the scripts from modules so they appear at the bottom of the page. Improve the placement of stylesheets from the modules so they render in the head of the webpage, and to make the navigation bar dynamic based upon loaded modules.


## Step 1 : Add a new .net standard Class Library

This will form the basic of our new module.

![Shared project](../images/standard-lib.png "Shared project")


## Step 2 : Add a new BasePageViewModel Class

Add a class to the shared project that looks like this:

```
public class BasePageViewModel
{
    public BasePageViewModel()
    {
        PageScripts = new List<string>();
        PageStyles = new List<string>();
    }

    public string PageTitle { get; set; }

    public List<string> PageScripts { get; set; }

    public List<string> PageStyles { get; set; }

}
```

## Step 3 : Add a BasePageViewModelExtensions Class

Add a class to the shared project that looks like this:

```
public static class BasePageViewModelExtensions
{

    public static T AddPageScript<T>(this T viewModel, string url) where T : BasePageViewModel
    {
        viewModel.PageScripts.Add(url);

        return viewModel;
    }

    public static T AddPageStyles<T>(this T viewModel, string url) where T : BasePageViewModel
    {
        viewModel.PageScripts.Add(url);

        return viewModel;
    }

}
```

## Step 4 : Update ManagmentViewModel.cs

Update the ManagmentViewModel.cs to inherit from the BasePageViewModel class:

```
public class ManagmentViewModel : BasePageViewModel
{
    public string PageContent { get; set; }
}
```


## Step 6 : Update ManagementController.cs

Update the Management Controller to use the BasePageModelExtensions to register the styles and scripts.

```
using Microsoft.AspNetCore.Mvc;
using Nomoni.Examples.Basic.AdminModule.Models;
using Nomoni.Examples.Basic.Shared;

namespace Nomoni.Examples.Basic.AdminModule.Controllers
{
    public class ManagementController : Controller
    {
        public IActionResult Index()
        {

            var model = new ManagmentViewModel()
            {
                PageContent = "I'm some page content",
                PageTitle = "Management Page"
            };

            model.AddPageStyles("/css/admin-styles.css");
            model.AddPageStyles("/js/admin-scripts.js");   

            return View(model);
        }
    }
}
```


## Step 7 : Remove Scripts and Styles from the Management Index View

The View should now look like this : 

```
@model Nomoni.Examples.Basic.AdminModule.Models.ManagmentViewModel

@{
    ViewData["Title"] = Model.PageTitle;
}

<h2 class="admin-heading">@Model.PageTitle</h2>

@Model.PageContent

<a href="#" id="button" class="btn btn-primary">Click Me</a>
```


## Step 8 : Update _Layout.cshtml 

The _Layout.cshtml will need amending to expect a model that inherits from BasePageViewModel.

Add a reference to the shared module in the basic module and update _Layout.cshtml to the following:

```
@model Nomoni.Examples.Basic.Shared.BasePageViewModel

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@Model.PageTitle</title>

    <environment include="Development">
        <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
        <link rel="stylesheet" href="~/css/site.css" />
    </environment>
    <environment exclude="Development">
        <link rel="stylesheet" href="https://ajax.aspnetcdn.com/ajax/bootstrap/3.3.7/css/bootstrap.min.css"
              asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
              asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute" />
        <link rel="stylesheet" href="~/css/site.min.css" asp-append-version="true" />
    </environment>

    @foreach (var url in Model.PageStyles)
    {
        <script src="@url"></script>
    }

</head>
<body>
    <nav class="navbar navbar-inverse navbar-fixed-top">
        <div class="container">
            <div class="navbar-header">
                <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                    <span class="sr-only">Toggle navigation</span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </button>
                <a asp-area="" asp-controller="Home" asp-action="Index" class="navbar-brand">Nomoni.Examples.Basic.Module</a>
            </div>
            <div class="navbar-collapse collapse">
                <ul class="nav navbar-nav">
                    <li><a asp-area="" asp-controller="Home" asp-action="Index">Home</a></li>
                    <li><a asp-area="" asp-controller="Home" asp-action="About">About</a></li>
                    <li><a asp-area="" asp-controller="Home" asp-action="Contact">Contact</a></li>
                </ul>
            </div>
        </div>
    </nav>

    <partial name="_CookieConsentPartial" />

    <div class="container body-content">
        @RenderBody()
        <hr />
        <footer>
            <p>&copy; 2018 - Nomoni.Examples.Basic.Module</p>
        </footer>
    </div>

    <environment include="Development">
        <script src="~/lib/jquery/dist/jquery.js"></script>
        <script src="~/lib/bootstrap/dist/js/bootstrap.js"></script>
        <script src="~/js/site.js" asp-append-version="true"></script>
    </environment>
    <environment exclude="Development">
        <script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-3.3.1.min.js"
                asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
                asp-fallback-test="window.jQuery"
                crossorigin="anonymous"
                integrity="sha384-tsQFqpEReu7ZLhBV2VZlAu7zcOV+rXbYlF2cqB8txI/8aZajjp4Bqd+V6D5IgvKT">
        </script>
        <script src="https://ajax.aspnetcdn.com/ajax/bootstrap/3.3.7/bootstrap.min.js"
                asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.min.js"
                asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal"
                crossorigin="anonymous"
                integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa">
        </script>
        <script src="~/js/site.min.js" asp-append-version="true"></script>
    </environment>

    @foreach (var url in Model.PageScripts)
    {
        <script src="@url"></script>
    }


    @RenderSection("Scripts", required: false)
</body>
</html>

```

## Step 8 : Update the Home Controller in the Basic Module

The HomeController.cs file will need amending to pass a BasePageViewModel to the view as follows :

```
public class HomeController : Controller
{
    public IActionResult Index()
    {
        var model = new BasePageViewModel();

        return View(model);
    }

    public IActionResult About()
    {

        var model = new BasePageViewModel();

        ViewData["Message"] = "Your application description page.";

        return View(model);
    }

    public IActionResult Contact()
    {

        var model = new BasePageViewModel();

        ViewData["Message"] = "Your contact page.";

        return View(model);
    }

    public IActionResult Privacy()
    {
        var model = new BasePageViewModel();

        return View(model);
    }

    [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
    public IActionResult Error()
    {
        return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
    }
}

```

## Step 9 : Launch App

Upon inspecting the source code for the page for "/admin/management" the web app should now show the stylesheet in the head of the web page and the script at the bottom of the page. This now means that the mosdule scripts could depend on JQuery or any other library framework loaded before the module scripts in _layout.cshtml. 

## The Source Code for this Tutorial can be found

[https://github.com/treefishuk/nomoni/tree/master/examples/Nomoni.Examples.AssetImprovements](https://github.com/treefishuk/nomoni/tree/master/examples/Nomoni.Examples.AssetImprovements)

## Next Steps

In the next tutorial we will look at improving the navigation menu

[Part 4 : Navigation](/nomoni/docs/getting-started/part-four-nav-improvements)

