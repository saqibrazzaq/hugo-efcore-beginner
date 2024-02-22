---
title: 4. Repository class for Person Entity
weight: 50
ShowToc: true
TocOpen: true
---

You are reading the 4th article in the beginning EF Core series with ASP.NET Web API. So far we have setup the base repository class that uses the Entity Framework’s DbContext and deals with DbSet<Entity>.

**RepositoryBase<Person>** creates **DbSet<Person>**, which is very generic. One generic class cannot handle all operations done on an entity. So we create separate Repository classes for each entity.

That means if we have 10 tables, we will have 10 Entities and 10 Repository classes. It seems a lot, but 80% of these classes will be very simple, they will just use the inherited Create, Delete, FindAll and FindByCondition methods. We will modify each repository only when needed.

So lets begin by creating our first repository class for the entity.

## PersonRepository class

Create a new interface **IPersonRepository** in the **Repository** folder. This interface will inherit **IRepositoryBase<Person>** interface.

```cs
public interface IPersonRepository : IRepositoryBase<Entities.Person>
{
}
```

Then create a new class **PersonRepository** in the same **Repository** folder. It will inherit from **RepositoryBase<Person>** and also implement **IPersonRepository**. Generate the constructor using Visual Studio, the class will become like below.

```cs
public class PersonRepository : RepositoryBase<Entities.Person>, IPersonRepository
{
  public PersonRepository(AppDbContext context) : base(context)
  {
  }
}
```

That is all we need to do CRUD operations on the Person table. Finally… we did it.

## But how do we use this PersonRepository class?

We are building an ASP.NET Web API application, which gives is the default WeatherForecaseController. The APIs are meant to be accessed via http. Right now, we can add a new PersonsController and use the PersonRepository class right inside the controller. But that is a bad approach. Our controllers will be dependent on the Repository, which will be a bad practice.

Remember the layered pattern diagram?

![Repository pattern](/images/blog/Repository-pattern.jpg "Repository pattern")

We will write Services, which will use our Repository classes.

Anyways, back to the Repository, it is not complete yet. We are missing one last thing in our generic repository classes. Not one, but few.

- How do we handle transactions?
- How do services get the reference of Repository classes?

In the next article, we will solve the above issues.