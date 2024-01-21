---
title: 8. Update, Delete and List all Persons
weight: 90
ShowToc: true
TocOpen: true
---

You are now reading the 8th article in the beginning of EF Core series. In the last article, we created a new controller for Person and added a method to handle HttpPost request to create a new person. In this article you will learn how to do the rest of the CRUD, which are update, delete and Read all records.

## List All Persons

To list all persons, we need to update two methods.

1. PersonService.GetAll
2. HttpGet method in PersonsController

No need to add anything new in the repository. As I shared previously, 80% of the queries related to CRUD will all be done by the RepositoryBase class. So lets edit the Service method now in PersonService class.

```cs
public IEnumerable<PersonRes> GetAll()
{
  var persons = _repositoryManager.PersonRepository.FindAll(
      false);
  return _mapper.Map<IEnumerable<PersonRes>>(persons);
}
```

The service calls the PersonRepository.FindAll method. TrackChanges is false, because we are only reading the data. Repository returns entity, so we convert entity to Dto using AutoMapper.

Open PersonsController and add a new method below.

```cs
[HttpGet]
public IActionResult GetAll()
{
  var res = _personService.GetAll();
  return Ok(res);
}
```

Since we are reading data, we will create new method with HttpGet annotation.

HttpGet methods can be accessed from Url request. However for HttpPost, we need to send data in request body.

Controller methods will be the simplest, they act as only endpoint for Web API. They just call the service methods. All business logic code will be in the service classes.

Run the project, it will open the Swagger default UI. Now you should see two methods in the Swagger page as follows.

![swagger get all](/images/swagger-getall-1024x596.jpg "swagger get all")

Open the **Get /api/Persons**, click on **Try it out** button and click **Execute**. You should now see 200 Ok response from server. And in response body, you should see the list of all the persons which exist in the database.

![swagger get all response](/images/swagger-getall-response-1024x446.jpg "swagger get all response")

You can also copy the URL and open it in any browser. The browser will call the API and load the Json response, containing all the persons.

![get all browser json](/images/getall-browser-json.jpg "get all browser json")

## Get Single Person

To get a single person, we need to edit code at two places, just like we did previously for GetAll

Open PersonService class and edit the Get method as follows.

```cs
public PersonRes Get(int personId)
{
    var entity = FindPersonIfExists(personId, false);
    return _mapper.Map<PersonRes>(entity);
}
```

Get method has a parameter, person Id. We will use the PersonRepository.FindByContidion to filter the records. So we will create another helper method FindPersonIfExists, which will

- Filter records by person Id, read the first record
- If no record found, throw Exception
- If record found, return the entity

This helper method will be reused again in Update and Delete methods later. Because Get, Update and Delete methods of RepositoryBase requires the entity. We need to find the entity first, for all 3 methods. We also need to check whether a person is found or not, so need to throw Exception if not found.

Instead of repeating the above in Get, Update and Delete separately, it is better to create a new helper method as below.

```cs
private Entities.Person FindPersonIfExists(int personId, bool trackChanges)
{
  var entity = _repositoryManager.PersonRepository.FindByCondition(
    x => x.PersonId == personId,
    trackChanges)
    .FirstOrDefault();
  if (entity == null) throw new Exception("No person found with id " + personId);

  return entity;
}
```

Note the trackChanges parameter. We need the helper method FindPersonIfExists to be generic. Get will pass false in trackChanges. Update and Delete will pass true in trackChanges.

Now open the PersonsController class and create a new get method as below.

```cs
[HttpGet("{personId}")]
public IActionResult Get(int personId)
{
  var res = _personService.Get(personId);
  return Ok(res);
}
```

Note the annotation, we used “{personId}”, this means personId is a variable, it can have any text value. The variable name in method parameter must match the variable in annotation “{variable}”. Don’t miss the curly braces.

“personId” is just hard coded string.

“{personId}” is a variable

In this Get method, we use **“{personId}”** in annotation and then used **int personId** in parameter.

Run the project, it will open the Swagger UI. Now you will see the 3rd method

![swagger get](/images/swagger-get-1024x233.jpg "swagger get")

Click on **Get /api/Persons/{personId}**, click on **Try it out**. Note the text box now. Swagger UI has provided a textbox to set the personId variable value. Try giving different values and execute the method.

If a Person with a valid personId is found, you will get Json response. If not found, you will get the error message with Exception.

You can also try this method using the URL directly in the browser. Try the following URLs and note the response.

- https://localhost:7277/api/Persons/1
- https://localhost:7277/api/Persons/2
- https://localhost:7277/api/Persons/333

## Update Person

For update, we will write code at 2 places, like we did previously.

Open PersonService class and edit the Update method as below.

```cs
public PersonRes Update(int personId, PersonReqEdit dto)
{
  var entity = FindPersonIfExists(personId, true);
  _mapper.Map(dto, entity);
  _repositoryManager.Save();
  return _mapper.Map<PersonRes>(entity);
}
```

Update method requires 2 parameters, which will be provided by the client.

1. personId
2. Request Dto

For update operation on repository, we need to get the entity first. We have reused the FindPersonIfExists helper method. This time we passed true, because we want to track changes and we want to modify the repository.

First we get the entity. The filter (where condition) and not found exception is already handled in the helper method, so we get the entity in one line code.

Then, we transfer the Dto to the entity. User provided values will be updated in the existing database values, using AutoMapper.Map method.

Then we called RepositoryManager.Save method. This save method updates all the tracked data. We passed true in trackChanges, so we can update it. If we pass false to the trackChanges, the Save method will not update. That is why there is no Update method in the RepositoryManager or RepositoryBase class. It just saves all data which is tracked.

## RepositoryManager.Save method works as Transaction

We fetched entity from repository with trackChanges set to true.

We updated entity from the user request dto.

Then we called the Save method.

This works like a Transaction or Unit Of Work.

If we fetch 3 different records with trackChanges set to true. Then modify these 3 records. And call the Save method in the last. These 3 updates will be saved altogether. If any one of the update fails, all 3 tracked changes will fail.

## Update Person Controller

The controller will be the same, 2 line code as usual. Open the PersonsController class and add new Update method as below.

```cs
[HttpPut("{personId}")]
public IActionResult Update(int personId, PersonReqEdit dto)
{
  var res = _personService.Update(personId, dto);
  return Ok(res);
}
```

For update, we use HttpPut method in Web APIs. The person Id is present in the annotation as “{personId}”, which works as a variable. The PersonReqEdit dto will be an extra data that the client needs to send.

Run the project, it will open the Swagger API. Now you will see 4th method, which is Put. Click on it.

![swagger put](/images/swagger-put-1024x305.jpg "swagger put")

Now the Swagger UI will ask for input data for both the variables.

1. personId in a textbox, because it is part of Url, we used in annotation
2. PersonReqEdit dto in Json format in request body

Set personId to 1, we already created 1 person.

The default template value for the request dto is already provided by Swagger UI. Update it and press Execute. You will see that changes will be saved in the database.

Pass some wrong id in personId, and you will get the exception message.

![swagger update](/images/swagger-update-1024x374.jpg "swagger update")

You can verify the results by querying to SQL Server or by using the Get method from the Swagger UI.

Now lets create the last basic method, which is Delete.

## Delete Person

Open PersonService class and add the Delete method as follows.

```cs
public void Delete(int personId)
{
  var entity = FindPersonIfExists(personId, true);
  _repositoryManager.PersonRepository.Delete(entity);
  _repositoryManager.Save();
}
```

To delete an entity, we first need to find it. We have reused the helper method FindPersonIfExists 3rd time here in the Delete method. We have passed true for trackChanges, because we want to modify the repository.

Once we get the entity, call the Delete method of PersonRepository and then call Save method to commit the tracked changes to the database.

Open PersonsController class and add the delete method as follows.

```cs
[HttpDelete("{personId}")]
public IActionResult Delete(int personId)
{
  _personService.Delete(personId);
  return NoContent();
}
```

The Delete method requires personId in parameter, just like Get and Update methods previously. It is a dumb controller and just calls the Service.Delete method.

The Delete method has nothing to return, so we just return NoContent() as response.

Run the project, Swagger default UI will open. You will see the 5th Delete method is now added in the method list.

![swagger delete](/images/swagger-delete-1024x353.jpg "swagger delete")

Click on it and then Try it out. You will see only one textbox for personId parameter. Enter 1 in personId and Execute, the record will be deleted from the database. And if you enter 1 again and Execute, it will throw the exception that no person found.

## Count all Persons

We also added Count method in the service, so lets define it as follows.

```cs
public int Count()
{
    return _repositoryManager.PersonRepository.FindAll(
        false)
        .Count();
}
```

We are counting all the records in the database. We first called FindAll method. trackChanges is false, because we only want to read. We used method chaining here to call two methods

1. FindAll – returns all records
2. Count – count the records, returned from previous method

EF Core will not SELECT all records and return in the service. Entity Framework will execute the query only when you use it. In the Count method, we are calling FindAll() and then Count() methods. We are actually USING the result of the last method in the chain. We are NOT using FindAll anywhere. So this way the final SQL that will be executed by the Entity Framework will be

```sql
SELECT COUNT(*)
FROM [Person] AS [p]
```

This is the exact SQL that we want, no overhead extra SQL added by the Entity Framework. Lets edit Count method as below

```cs
public int Count()
{
  var persons = _repositoryManager.PersonRepository.FindAll(false);
  return persons.Count();
}
```

Here we are not using method chaining. Will it first execute SELECT * FROM Person. And then execute SELECT Count(*) again? No. the EF Core again translates this code to SELECT COUNT(*) from [Person]. No problem if we use this way to count.

Lets update the Count method in another way.

```cs
public int Count()
{
  var persons = _repositoryManager.PersonRepository.FindAll(false);
  var usePersonsInSomeWay = persons.ToList();
  return usePersonsInSomeWay.Count();
}
```

Now this is the WRONG way to just get Count of all records. This code will be executed as follows.

- FindAll method will be called, result of type IQueryable<Person> will be stored in the persons variable. No SQL will be executed yet.
- persons.ToList() will convert IQueryable<Person> to List<Person>. We are calling ToList on all records, this will be translated to the following SQL

```sql
SELECT [p].[PersonId], [p].[FirstName], [p].[Gender], [p].[LastName], [p].[PhoneNumber]
FROM [Person] AS [p]
```

If we only want the count, above code would be bad. We don’t really want to convert IQueryable<Person> to List<Person> and then call Count. Revert back to the following code.

```cs
public int Count()
{
  var count = _repositoryManager.PersonRepository.FindAll(false)
    .Count();
  return count;
}
```

We are calling FindAll, but not using it. We are calling Count() and are storing in variable and returning it, so we are only using the Count(). EF Core will intelligently transform it into SELECT COUNT(*) query, which is what we need.

## Is Entity Framework good in Performance?

If you know how to use it, to get the proper SQL you want. Then it is as good as the generated SQL!! If you don’t know how to use the Entity Framework and are calling and using methods like FindAll un-necessarily, it would be really bad. But you can’t blame Entity Framework for it.

You could do the same SELECT * FROM Table in pure SQL, store results in some array and then count the array to get the result, if you don’t know about SELECT COUNT(*). You can’t blame SQL is slow. You really need to learn SQL. The better you know raw SQL, you will be better at Entity Framework.

In the next tutorial you will learn how to do searching, paging and sorting.