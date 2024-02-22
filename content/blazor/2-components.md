---
title: 2. Blazor Components
weight: 2
ShowToc: true
TocOpen: true
---

## Create Blazor Project

Open Visual Studio 2022 and make sure that you have the latest version installed. At current date, the latest version is 17.9.

Create a new project, select "Blazor Web App" from the project types. Make sure that .NET Framework selected is .NET 8 Long Term Support. Leave all the options as default and create the project.

![Blazor folder structure](/images/blazor/blazor-folder-structure.jpg "Blazor folder structure")

Check the "Pages" folder, which contains 4 Razor files. These are components.

Also have a look at the "Layout" folder, which has 2 razor files. These are also components.

Open Home.razor, it shows Hello World in h1 heading and after that a welcome message in a new line.

## Add New Component

Add a new component in Pages folder as below.

![Add new component](/images/blazor/blazor-add-new-component.jpg "Add new component")

Name it "WelcomeMessage". It will create a new WelcomeMessage.razor file in Pages folder. It contains Welcome text in h3 tag and also an empty code block. This is a very basic component, which can be used in other components.

Open Home.razor. Delete "Welcome to your new app" line. Add the new welcome component to display the message.

```html
@page "/"

<PageTitle>Home</PageTitle>

<h1>Hello, world!</h1>

<WelcomeMessage />
```

Run the project, you should see the welcome message coming from the component.

![welcome component](/images/blazor/welcome-component.jpg "welcome component")

In similar way you can also include Counter and Weather components from the Home page. The component can work independently as a separate page and can also be included in another page.