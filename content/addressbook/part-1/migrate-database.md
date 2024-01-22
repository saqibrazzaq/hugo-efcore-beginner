---
title: 3. Database context, Base Repository, Extension Methods etc.
weight: 540
ShowToc: true
TocOpen: true
---

We have created entities, now we will generate database and tables from these entities. The process is same what we already used in the beginner series. You can follow [person-api/generate-database/](/person-api/generate-database/) to generate the database. We will not go in details here, we will just summarize how to generate database from entity classes.

## Generate database from Entity classes

- Create .env file and add connection string in variable SQLSERVER
- Create static property SecretUtility.SqlServer and load this environment variable
- Add extension method ConfigureEnvironmentVariables to load variables from .env using DotNetEnv library
- Add AppDbContext class, inherit it from DbContext, pass options in constructor
- Add ConfigureSqlContext extension method, register database context with connection string loaded from SecretUtility.SqlServer
- Add other extension methods for AutoMapper, CORS, Exception handler and database auto migration
- Call these extension methods in Program.cs
- Add tables in AppDbContext

Now you should be able to add migration and update database using Entity Framework commands in package manager console.

All these steps were already explained in the following articles, so no need to repeat here.

- [Generate database and tables](/person-api/generate-database/)
- [Generic Repository Base class](/person-api/generic-repository/)

You can get the full source code of this project from GitHub repository https://github.com/saqibrazzaq/efcorebeginner/tree/main/AddressBook.

So far, this project is same as the Person web API project, it just contains more entity classes, rest is 100% same so far. But we are in the beginning phase, we will create repositories for each entity and then services, they will be different.