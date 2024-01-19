---
title: Build Person API, very basic using ASP.NET Core Web API and EF Core
---

If you want to learn the basics of Entity Framework, but use best design and architectural patterns, start from this page.

GitHub Repository: https://github.com/saqibrazzaq/efcorebeginner

You will learn how to do CRUD operations on a single table with EF Core (using SQL Server as the backend database). In this beginner series, we will keep just one table with 4-5 fields. You will learn to

- Create entity classes, which represents database schema
- Use Entity Framework DbContext to represent the entities
- Code first approach, design classes first, update database later
- Use repository pattern to keep the EF Core related stuff together
- Use service classes for business logic and domain classes
- Create controllers to expose web APIs
- Create DTOs for API clients
- Do transformation between DTOs and entity classes
- Validate data in service layer
- Better and generalized exception handling
- Search, sort and paging using EF Core LINQ

You can learn and practice Entity Framework by just initializing DbContext with database connection string, then create DbSet<Entity> and do CRUD operations. This approach is good for learning Entity Framework, but not good for using in a large scale system.

For medium sized system, you may have

- 20 or more tables
- millions of records in database
- hundred or more concurrent users
- thousands or more transactions per day
- 10 ore more developers

In such a system using EF Core classes directly in your application classes is not a good idea. You need to follow good software development architecture, so that the code is reusable and easy to maintain.

In this beginning series, we will tell you how to build a large scale system with the best practices, so that the system scale well, and it also performs well.

We will use the standard layers pattern to build the ASP.NET based backend. We will use only single table in the beginner series. Later on we will add more features in the system and make it a large scale system by adding more features, tables, data etc.

## What is the story behind this website and article series?

I started my first job back in 2003, with classic ASP using VBScript, JavaScript/HTML. We used to write SQL and stored procedures. I used this stack for many years. Then came the .NET. I moved to .NET when version 3 was released. Entity Framework was either new or did not exist at that time, I don’t remember. We did not use any framework for database and used SQL queries, because we had so many years of experience in writing SQL and optimizing these queries.

Many times I tried to adopt the Entity Framework, but failed due to many reasons, including

- It had entity classes, the concept of data classes was new, I was comfortable with SQL and dataset, data readers they return.
- Why use EF when SQL is working just fine?
- EF is slow as compared to plain SQL, which is the native language of the database.
- The performance was the main reason I quit on EF Core. LINQ was not mature at that time. I admit that I did not give much time to learn LINQ and Entity Framework.

Then job nature changed, I did not do much database related development for a long time, so forgot about datasets, SQL, EF etc.

After a long time I decided to work again on a pet project, that used databases. A lot of things changed. Entity Framework was much more evolved. I decided to give it a try.

I tried and it was very easy, just create Entity classes, setup DbContext and you are good to go.

I tried on complex queries and compared the performance, it was very slow. I learned more and found out that I was abusing Include to fetch related data, which was using un-necessary outer joins. I knew it was I who was doing something wrong.

I learned about LINQ and I loved it. I loved how it beautifully provided the alternate of WHERE and SELECT in SQL. I also learned how to use LINQ and Entity Framework properly, so that it has nearly equal performance to the plain SQL.

I learned how to build medium scaled software system using EF Core with good design patterns.

It took me an iteration of building test projects around 10 times. Yes!! 10 times I repeated similar projects and learned new things every time I built. It took 3-4 months on how to setup EF Core with repository, services, use authentication using Identity Framework, expose APIs, use DTOs, consume APIs in React, use better error handling, validation, searching, paging, sorting etc.

I learned it from many places including books, medium articles, stackoverflow, several videos on YouTube, Udemy courses and many other sites which I don’t even remember.

So I decided to create one basic project (single table) and one medium scale project (multiple tables) using the best practices I learned. These projects will include common SQL related problems, which took a lot of time to learn with Entity Framework using LINQ.

Based on these projects and article series, you can also build your next project in a better way.