---
title: Generic Repository Base Class
weight: 40
ShowToc: true
TocOpen: true
---

This is the 3rd article on the beginning EF Core series. In the previous articles we created a new Web API project, entity class, DbContext class, used DotNetEnv to keep connection string in env file, initialized the EF Core in Program.cs and then finally created the database using our first migration command.

![layer](/images/layer.jpg "layer")
*Photo by Pixabay: https://www.pexels.com/photo/asphalt-balance-blur-close-up-268018/*

Now it is time to do CRUD operations with the database using the Entity Framework. You can directly call the EF Core classes for create, read, update and delete operations, but that is not recommended approach. We use the layers architecture and keep the database related operations in the Repository layer.

## IRepositoryBase Interface

Add a new folder **Repository**. In this folder, add a new interface **IRepositoryBase**. This interface will have only 4 methods, which will do the 80% of the database related work. Sounds too good to be true? We will find out soon enough.

```cs
public interface IRepositoryBase<T>
{
  IQueryable<T> FindAll(bool trackChanges);
  IQueryable<T> FindByCondition(
    Expression<Func<T, bool>> expression,
    bool trackChanges,
    Func<IQueryable<T>, IIncludableQueryable<T, object>>? include = null
    );
  void Create(T entity);
  void Delete(T entity);
}
```

The above code requires reference to the following two namespaces. Make sure that you are using the right ones. In case of Visual Studio 2022 latest version with .NET Core 7, these are added automatically if you copy the code and have already added EF Core NuGet packages.

- Microsoft.EntityFrameworkCore.Query
- System.Linq.Expressions

Lets get introduced to each of the 4 methods.

- FindAll – Returns all items in the collection or all records from the table
- FindByCondition – Filters items in the collection by applying conditions. It also includes the data from the related tables.
- Create – Add a new item to the collection, means add a new record in table.
- Delete – Delete an item/record.

**Where is the Update method?**

There is no update method. Note the trackChanges parameter in the FindAll and FIndByCondition methods. We will use this parameter to perform updates. It is very easy, you will see it soon.

## RepositoryBase class

Lets implement the IRepositoryBase interface now. Create **RepositoryBase** class in the **Repository** folder. Edit the class so that it looks like the code snippet below.

```cs
public class RepositoryBase<T> : IRepositoryBase<T> where T : class
{
}
```

RepositoryBase is a generic class of type T, where T is a class. We will use the repository with the entity classes. We will see it soon when we do CRUD operations on the Person entity.

You will get errors at this point. Use Visual Studio to implement interface, so that it will add the correct method signatures automatically.

Add a protected member in the **RepositoryBase** class of type AppDbContext. Remember **AppDbContext**? This is our database context class, which inherits Entity Framework’s DbContext class. AppDbContext has list of all entities/tables.

Also add a constructor to initialize the db context. See the below code snippet.

```cs
protected AppDbContext _context;
public RepositoryBase(AppDbContext context)
{
  _context = context;
}
```

Now we need to define the methods. We will start with Create and Delete methods, which have the simplest implementation with a single line.

```cs
public void Create(T entity)
{
  _context.Set<T>().Add(entity);
}

public void Delete(T entity)
{
  _context.Set<T>().Remove(entity);
}
```

Both methods uses the DbContext.Set<T> method. We use it to create a DbSet or type T (our Entity class). Once we get the DbSet<Entity>, we can call Add, Remove and other methods. Add method will add a new entity in the DbSet. Remove method will remove the entity from the DbSet. And the DbSet is mapped to the Entity class. Entity class is mapped to our database table. That is how it works with the EF Core, using Object Oriented design approach.

Next, we define the FindAll method. It accepts trackChanges as a parameter. We use **AsNoTracking()** option for **ready only queries**, where we only want to READ the data. AsNoTracking() options is very fast, as the Entity Framework does not keep track of the tables and fields. Without it, EF Core framework keeps track of every field and tables that are used in DbSet.

With AsNoTracking(), we can update the DbSet.

Without AsNoTracking(), we cannot update. The DbSet is readonly.

```cs
public IQueryable<T> FindAll(bool trackChanges)
{
  return !trackChanges ?
    _context.Set<T>()
      .AsNoTracking() :
    _context.Set<T>();
}
```

FindAll method will just return the whole DbSet. Yes, all the records in a table. Which is equivalent to the following SQL.

```bat
SELECT * FROM Table
```

We will use FindAll method in simple scenarios like loading data in dropdowns or loading data from tables where number of records are really low. For larger tables, we can’t just return ALL the records.

So we have a generic FindByCondition method. Lets define this as shown below.

```cs
public IQueryable<T> FindByCondition(Expression<Func<T, bool>> expression, 
  bool trackChanges, 
  Func<IQueryable<T>, IIncludableQueryable<T, object>>? include = null)
{
  // Get query
  IQueryable<T> query = _context.Set<T>();

  // Apply filter
  query = query.Where(expression);

  // Include
  if (include != null)
  {
    query = include(query);
  }

  // Tracking
  if (!trackChanges)
    query.AsNoTracking();

  return query;
}
```

FindByCondition has 3 parameters.

1. expression – we will use LINQ expressions to filter the DbSet. We will learn later how we do it.
2. trackChanges – for update in DbSet
3. include – we include other related tables. Not used in this beginner series, but we will cover it later when we created multiple tables having relationships.

We will not go into more details now. If you are seeing LINQ expressions and generic types first time, it will be difficult. But it is a very good practice to generalize the Repository, so that EF Core’s DbContext is used from only RepositoryBase class.

Are we ready now to do CRUD on the Person table. Not yet. Actually we can initialize RepositoryBase<Person>, but we will not do it this way. We will create separate repository classes for each entity.

![wait](/images/wait.jpg "wait")
*Photo by Andrea Piacquadio: https://www.pexels.com/photo/young-annoyed-female-freelancer-using-laptop-at-home-3808008/*

This is really too long time to do a simple CRUD on a table with 4-5 fields. But we will in future have a slightly bigger application, with 5-6 related tables, having hundreds of thousands of records. Once we setup the Repository right, we will be able to create entity’s separate repository classes very quickly.

Have a little patience and stay with us. We will create a bigger application with best practices using Entity Framework.

In the next article, we will write about creating repository class for the entity.