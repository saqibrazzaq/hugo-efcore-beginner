---
title: 5. Transactions with RepositoryManager
weight: 60
ShowToc: true
TocOpen: true
---

This is the 5th article in the EF Core beginner series. So far we created base repository classes which uses Entity Framework’s DbContext to do CRUD operations. In this article, you will learn how to add support for transactions and how to get reference to all the repositories.

## IRepositoryManager Interface

Why we need another generic interface?

We created **IRepositoryBase<T>**, which is the base class for all the repositories.

Then we created **IPersonRepository**, it inherited from **IRepositoryBase**. It is just a repository for Person table, it only does CRUD operations on Person.

IRepository interface serves two purposes

1. Get instance of the any repository in a simple way.
2. Handle transactions with Save method, even if we do CRUD operations on multiple repositories

In the **Repository** folder, create a new interface **IRepositoryManager**. It will contain a Save method. It will also contain all repositories as its public properties. See the code below.

```cs
public interface IRepositoryManager
{
  IPersonRepository PersonRepository { get; }
  void Save();
}
```

It looks really simple. Our services will use IRepositoryManager. Look how simple it is to initialize just IRepositoryManager. Then you can access all the repositories. All this effort so far was done to make the repositories as generic and as reusable as possible. Lets move to its implementation.

## RepositoryManager class

Create a new class **RepositoryManager** in the **Repository** folder. It will implement the **IRepositoryManager** interface. Use Visual Studio to implement the interface with the generated methods.

Here we will also need the AppDbContext.

**Important:** Whenever we want to do any operation on database, we will always use the IRepositoryManager interface.

So we will initialize all the repositories in this class. Hence we require the AppDbContext here.

See the complete code below for the RepositoryManager class.

```cs
public class RepositoryManager : IRepositoryManager
{
  private readonly AppDbContext _context;

  // List all the repositories
  private readonly Lazy<IPersonRepository> _personRepository;
  public RepositoryManager(AppDbContext context)
  {
    _context = context;

    // Initialize all the repositories
    _personRepository = new Lazy<IPersonRepository>(() =>
      new PersonRepository(context));
  }

  public IPersonRepository PersonRepository => _personRepository.Value;

  public void Save()
  {
    _context.SaveChanges();
  }
}
```

Spend some time reading it in your Visual Studio, we will try to explain what is happening here. It all seems too complex. But this complexity allow us greater reusability and simpler way to use repositories.

Note how we use **lazy loading** for in the constructor. We will always use the **IRepositoryManager** for any database operation, that means the constructor will always get called. But the Repository will be initialized ONLY when we access the repository using the public property.

Imagine there are 10 repositories in the RepositoryManager. And we only need to access Person repository. It would be a bad idea if we initialize all 10 repositories in the constructor!! That is why we use lazy loading. Constructor initialization will be very fast. The initialization will only take place when we access IRepositoryManager.PersonRepository.

Build the project, it should build now without any errors. In case of any error please read the article again if you missed anything. You can also check the GitHub repository for reference code.

So now are we ready to use CRUD operation on the Person. Yes we are!! Finally… yes!! we are ready to go.

In the next article, we will create service for managing Person.