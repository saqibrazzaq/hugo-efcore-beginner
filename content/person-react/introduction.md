---
title: 0. Introduction - React frontend app for Person API
weight: 210
ShowToc: true
TocOpen: true
---

In this tutorial series you will learn to create a React app, which will call the Person API that we built in https://efcorebeginner.com/person-api. The web API was built using ASP.NET Core 6. All communication is done via http protocol, so we can choose our favorite language and platform for building frontend client application. We choose React because it is easy to get started with it. React is one of the most used frontend library for building client applications and developer support is widely available.

Since the API we built is very simple, based on single table with 5 columns, having basic CRUD operations. The React frontend app would also be very basic, with CRUD UI. We will also use best practices to write React app which consumes API services. In our React frontend app, we will use

- TypeScript language for strong typing
- Define Dtos at client side, so that we get strongly typed objects
- Chakra UI for styling
- Axios for writing services that call APIs
- Formik with Yup and Yup password for forms and validation

Lets continue in the next page to create and setup our React client app.