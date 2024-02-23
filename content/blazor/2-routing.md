---
title: 2. Routing
weight: 2
ShowToc: true
TocOpen: true
---

## Link Url Path with Razor Components

Routing means which component will run when you open a URL in the browser. For example if you open www.website.com or www.website.com/counter, which actual razor components will run.

In React, we use React Router library, you can define routes with paths. Similar method is available in ASP.NET MVC, you can link paths with controller classes.

In Blazor, it is very easy. You define routing in each razor component.

Check Home.razor and Counter.razor pages. You will see @page on top. @page defines the url path.

In home.razor, @page "/" is defined, which means www.website.com will run the home.razor component.

In counter.razor, @page "/counter" is defined, which means www.website.com/counter will run the counter.razor component.

## NavLink by Microsoft

NavLink is the builtin ASP.NET class for routing in Blazor. You can also see NavLink used in Layout/NavMenu.razor component. It is being used for links in the navigation menu.

Basic usage is very easy.

```html
<NavLink href="counter">Link with NavLink</NavLink>
```

## a HTML tag

You can also use the standard a HTML tag for links. In href, use url path, it will work.

```html
<a href="counter">Link with a</a>
```

## Open Page on Button Click

If you want to open a page on button click event, use NavigationManager built in class. To use this class in Razor component, you have to inject it. And don't forget to set render mode as InteractiveServer for code block to work.

```html
@inject NavigationManager NavManager
@rendermode InteractiveServer
<button @onclick="OpenCounter">Button</button>
```

Define the OpenCounter function in the code block.

```cs
@code {
    protected void OpenCounter()
    {
        NavManager.NavigateTo("counter");
    }
}
```

I used all 3 methods to create links in the homepage. All open the counter page. 

![navigation methods](/images/blazor/navigation-methods.jpg "navigation methods")

