---
title: 1. Introduction to Blazor
weight: 1
ShowToc: true
TocOpen: true
---

## What is Blazor

Blazor is Microsoft's solution for the single page web application developoment frameworks e.g. React, Vue, Angular etc. 

## Why you should even think of adopting Blazor?

If you want to do web client side development in C#, then Blazor is an ideal solution. You don't need to learn JavaScript.

It is not that simple. It actually goes way beyond JavaScript. Blazor is an SPA framework for web development. It is an alternate of libraries like React, Vue and the likes.

If you have experience in React, you know that it is not just about React and JavaScript. You need to learn

- Formik for Form handling
- Axios for sending http requests to API
- Yup for form validation
- ChakraUI for layouts, UI controls
- react-select for search textbox
- react-dropzone for file upload drag and drop
- react-router-dom for routing
- grid like component that can do searching, paging and sorting
- TypeScript if you want to see type safety errors at compile time

The list may go on and on. There are a lot of good libraries for each of the above requirement. Almost all are open source, with tons of users and support. It is really great and fun to work on all these, but it takes time to learn.

If you have past experience of development in ASP.NET Forms and MVC, and you want to do client side web in 2024, then you can try Blazor. It is quick, it is Microsoft, means most of the features are built in.

## Blazor project types

Now within Blazor, you can choose from 3 types of projects.

**Blazor Server App**

By using this type your project will run like classic web frameworks e.g. PHP, ASP.NET MVC, Web Forms etc. All the code is executed on the server. The server sends HTML to the browser.

When you open www.website.com/page1, the code is executed on the server, server converts to HTML, HTML sent to the browser.

Then you open www.website.com/page2, the code of page2 is again executed on the server, translated to HTML and then HTML sent to the browser.

This is the old classic model. Whenever you visit a new page, the whole page will be executed on the server. The whole HTML will be sent to the browser. The whole page will be downloaded from server to the browser. This architecture results in more network traffic betweek server and the browser. To solve this problem, many JavaScript based SPA frameworks were introducted like React, Next, Vue, Angular etc.

**Blazor WebAssembly Standalone App**

This is just like React and Angular. The code is executed on the browser. Microsoft runtime is downloaded on the browser, that's why the C# code works on the user's browser. The .NET runtime is actually downloaded to the browser.

When you visit www.website.com/page1, first the .NET runtime will be downloaded to the browser. Then page1 will be executed locally on the browser.

Then you visit www.website.com/page2, you already have the runtime, so the execution will be really fast. It will like SPA (single page application).

**Blazor Web App**

This is the generic type and recommended to be used in general. You have the control at page level, whether it executes on the server or on the browser. It is the combination of the Blazor server and WebAssembly project types.

Whenever in doubt, use the Blazor web app type.