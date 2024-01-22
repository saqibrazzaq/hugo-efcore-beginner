---
title: 0. Introduction - AddressBook – A small web app to manage Addresses
weight: 410
ShowToc: true
TocOpen: true
---

We are going to build a small app to manage Address Book. We will cover some complex scenarios which are used commonly in real world apps. It will not be a basic CRUD app like [Person](/person-api). It will contain

- Multiple tables with relations
- Image storage in 3rd party cloud service
- Transaction processing
- Import bulk data from json into SQL Server using ASP.NET
- Generate hundreds of thousands of records to test performance

It will consist of two layers. The API layer will be built with ASP.NET Core 7, based on Repository using Entity Framework Core 7. The UI layer will be built with React 18, using Chakra UI, Formik, Axios etc.

If you are new to ASP.NET C#, Entity Framework, React etc. then it is recommended to read the basic series first at [Person API](/person-api). In the basic series we covered each step in detail. But for the AddressBook app, we will not cover each and every thing, we will just focus on the core design, complex queries, all those things which were not in the beginner series.

If you already know some ASP.NET and Entity Framework and want to work on a larger app, some real world app, then following this tutorial series is recommended for you.