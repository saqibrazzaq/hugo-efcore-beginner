---
title: 6. Use Repository in Service, and DTOs
weight: 70
ShowToc: true
TocOpen: true
---

You are now reading the 6th article in the beginner series of EF Core beginner series. In the previous articles we created the RepositoryManager for getting any repository. We also worked on the RepositoryBase class that handles CRUD operations on entities using EF Core’s DbContext class. Now we are finally ready to use the Repository in our service classes.

## IPersonService interface

Add a new folder **Services** in the project. In this folder add a new interface **IPersonService**. Here we add contracts what we need to do with Person. Lets add some basic CRUD operations.

```cs
public interface IPersonService
{
  IEnumerable<Entities.Person> GetAll();
  int Count();
  Entities.Person Get(int personId);
  Entities.Person Create(Entities.Person person);
  Entities.Person Update(int personId, Entities.Person person);
  void Delete(int personId);
}
```

There is nothing special in it, only basic create, update, read and delete operations.

This service will be used by the web api controller. And the controller will act as a dummy method, which just acts as a bridge between client and the service. Which means the controller will have the same return type and parameters. This is really wrong!!

## What is wrong with our Service interface?

The GetAll method here is returning the list of Person entity to the service. The entity is our database schema. We should never return our schema to the client, browser or outside world.

Imagine having a User service, that returns the User entity. The user entity will contain confidential information like password hash. We should never expose such information to the outside world. In fact, we should never even expose any Entity to the outside world.

The Repositories should work on the entities.

The services should work on the DTOs.

### What are DTOs?

DTO stands for Data Transfer Objects. Dtos should be used to send/receive data to/from the client/browser.

Dtos should exclude the confidential information, which is present in the entity.

Dtos can also include any extra information, which is not part of the entity.

Dtos are meant to be used by the end user, browser, client, anyone who is using the web api.

So our IPersonService interface should use Dtos. Service should not use entities in return type and parameters.

## DTOs for the Person entity

We have to create the Dtos for the Person now. Any entity will have at least 3 Dtos.

- CreateDto – client will send this to create a new record.
- UpdateDto – client will send this to update an existing record.
- ResponseDto – We will send this to the client instead of entity.

Create a new folder **Dtos** in the project. In this folder create a new class **Person**. The default filename will be Person.cs. We will use one file to create all Dtos for the person.

In our case, we don’t have anything to hide from user, we can safely use the same properties as entity. Update Person.cs and add the following Dtos in it. Below is the complete code snippet.

```cs
public class PersonRes
{
  public int PersonId { get; set; }
  public string? FirstName { get; set; }
  public string? LastName { get; set; }
  public string PhoneNumber { get; set; } = "";
  public string Gender { get; set; } = "";
}
public class PersonReqEdit
{
  [Required, MaxLength(100)]
  public string? FirstName { get; set; }
  [Required, MaxLength(100)]
  public string? LastName { get; set; }
  [MaxLength(20)]
  public string PhoneNumber { get; set; } = "";
  [MaxLength(1)]
  public string Gender { get; set; } = "";
}
```

**PersonRes** class is the Response Dto, which will be sent to the client. We will use it as return type in **GetAll** and **Get** methods. As a convention, we will use Res postfix with the Response Dtos.

**PersonReqEdit** is the Dtos for create and edit methods. **Req** postfix is used for any request Dto. The client will send the Request Dto to our controller/service.

Note the **PersonRes** class, it has exact same properties as the Person entity. There is no confidential information. But we removed the data annotations, we don’t need in the response Dtos.

Note the **PersonReqEdit** Dto. The same Dto will be used for create and edit operation. There is no PersonId in it, as it is not required for creating new Person. For update, we keep the Id in url param, which is outside the Dto. The request Dtos also have the validation related annotations. We keep these so that we also validate the data at the service level.

Now lets use our Dtos in the Person service.

## IPersonService interface with the Dtos

Open **IPersonService** class, previously we used entities, now we will update and use Dtos as follows.

```cs
public interface IPersonService
{
  IEnumerable<PersonRes> GetAll();
  int Count();
  PersonRes Get(int personId);
  PersonRes Create(PersonReqEdit person);
  PersonRes Update(int personId, PersonReqEdit person);
  void Delete(int personId);
}
```

Now our Get methods return Response Dtos. And Create/Update methods take Request Dtos as parameters. No more entities in service classes.

IPersonService seems like a good contract. Lets implement it now.

## PersonService class

Create **PersonService** class in **Services** folder. This class will implement the IPersonService interface. Use Visual Studio to implement interface, so that all methods are generated with the boilerplate code.

We will use IRepositoryManager and initialize it using the dependency injection, in the constructor. The code will look like below.

```cs
public class PersonService : IPersonService
{
  private readonly IRepositoryManager _repositoryManager;

  public PersonService(IRepositoryManager repositoryManager)
  {
    _repositoryManager = repositoryManager;
  }

  public int Count()
  {
    throw new NotImplementedException();
  }

  public PersonRes Create(PersonReqEdit person)
  {
    throw new NotImplementedException();
  }

  public void Delete(int personId)
  {
    throw new NotImplementedException();
  }

  public PersonRes Get(int personId)
  {
    throw new NotImplementedException();
  }

  public IEnumerable<PersonRes> GetAll()
  {
    throw new NotImplementedException();
  }

  public PersonRes Update(int personId, PersonReqEdit person)
  {
    throw new NotImplementedException();
  }
}
```

This is quite good now. Service methods taking Dtos in parameters and returning Dtos.

But the IRepositoryManager knows nothing about the Dtos. It only works with the entities. The RepositoryBase class works with the entities. EF Core is designed to work with the entities. But we created Dtos, because we had our reasons.

Take Create method as an example. It takes a RequestDto as a parameter, and returns ResponseDto. But the repository needs entity!! Try the following code, which will not work.

```cs
public PersonRes Create(PersonReqEdit person)
{
  _repositoryManager.PersonRepository.Create(person);
  _repositoryManager.Save();
  return person;
}
```

First look at how easily we used _repositoryManager.PersonRepository.Create() method. And then called the Save method to actually insert in the database.

But it does not work, as the RepositoryManager requires an entity. But we got Dto from the client.

We need to convert RequestDto to our entity class, PersonReqCreate to Person. Then it will work with the repository.

After Save method, we also need to convert entity to ResponseDto, Person to PersonRes.

We will use [AutoMapper](https://automapper.org/) for automatically converting one class to another. Since the property names in our entity, Request and Response Dtos are same, the transformation will work automatically, with minimal code.

Add the following NuGet package in your project.

```bat
AutoMapper.Extensions.Microsoft.DependencyInjection
```

Below is the screenshot of this NuGet package. Make sure that you install the correct package with Dependency Injection.

![automapper](/images/blog/automapper-1024x106.jpg "automapper")

There are 129M downloads of this package. It is very popular and commonly use for transformation of Dtos into entities and vice versa.

## AutoMapper configuration

To use the AutoMapper with dependency injection, we need to

- Initialize it, so that we can use it anywhere
- Configure our Dtos and entities

In project root, add a new class MappingProfile. It will inherit AutoMapper.Profile class. Add the default constructor. We configure the transformations in this default constructor. Below is the complete code snippet for MappingProfile.

```cs
public class MappingProfile : Profile
{
  public MappingProfile()
  {
    // Person
    CreateMap<Entities.Person, PersonRes>();
    CreateMap<PersonReqEdit, Entities.Person>();
  }
}
```

We added 3 lines in the constructor.

- Convert Person entity to PersonRes Dto
- Convert PersonReqEdit Dto to entity

Open Extensions\ServiceExtensions.cs and add the 3rd static method here.

```cs
public static void ConfigureAutoMapper(this IServiceCollection services)
{
  services.AddAutoMapper(typeof(MappingProfile));
}
```

To use this extension method, we need to call it in Program.cs like below.

```bat
// Add services to the container.
builder.Services.ConfigureEnvironmentVariables();
builder.Services.ConfigureSqlContext();
builder.Services.ConfigureAutoMapper();
```

So far we got 3 extension methods in total.

Our configuration for AutoMapper is complete. Now back to the Create method of the **PersonService** class.

```cs
public class PersonService : IPersonService
{
  private readonly IRepositoryManager _repositoryManager;
  private readonly IMapper _mapper;
  public PersonService(IRepositoryManager repositoryManager, 
      IMapper mapper)
  {
    _repositoryManager = repositoryManager;
    _mapper = mapper;
  }

  public int Count()
  {
      throw new NotImplementedException();
  }

  public PersonRes Create(PersonReqEdit dto)
  {
    var entity = _mapper.Map<Entities.Person>(dto);
    _repositoryManager.PersonRepository.Create(entity);
    _repositoryManager.Save();
    return _mapper.Map<PersonRes>(entity);
  }

  public void Delete(int personId)
  {
      throw new NotImplementedException();
  }

  public PersonRes Get(int personId)
  {
      throw new NotImplementedException();
  }

  public IEnumerable<PersonRes> GetAll()
  {
      throw new NotImplementedException();
  }

  public PersonRes Update(int personId, PersonReqEdit person)
  {
      throw new NotImplementedException();
  }
}
```

PersonService has now 2 private members

- IRepositoryManager – for database related operations
- IMapper – for converting Dtos to entity and vice versa

Both members are initialized in the constructor, using dependency injection pattern.

Also note the Create method. It receives the RequestDto in parameter. We convert the RequestDto to entity, using IMapper.

Then add the entity to the repository. Save changes.

Then finally convert the entity to the ResponseDto.

In the next article, we will write controller methods to create a new Person.