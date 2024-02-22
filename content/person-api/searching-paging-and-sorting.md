---
title: 9. Searching, paging and sorting in Entity Framework 7
weight: 100
ShowToc: true
TocOpen: true
---

This is the 9th article in the beginning series of the Entity Framework Core tutorial. In the previous articles, we built base classes for repository and built services and controllers to do simple CRUD operations on Person table.

Searching, paging and sorting are more complex than basic Insert, Update and Delete methods. But it is very common to use in any project. Any project that has more than 20 records in a table, must implement searching, paging and sorting.

These three operations are related, so we will use single method to implement them. Lets start with creating a common base class that will be used by all repositories.

In Common folder, create a new folder Paging. In **Common\paging** folder create a new class **PagedReq** as follows.

```cs
public class PagedReq
{
  const int maxPageSize = 50;
  public int PageNumber { get; set; } = 1;
  private int _pageSize = 10;
  public int PageSize
  {
      get
      {
          return _pageSize;
      }
      set
      {
          _pageSize = (value > maxPageSize) ? maxPageSize : value;
      }
  }
  public string? OrderBy { get; set; }
  public string? SearchText { get; set; }
}
```

For searching, we have the SearchText property, we will add filters (Where conditions) and search text in the required fields.

For paging, we have PageNumber (the page which user wants to see) and PageSize property.

For sorting, we have the OrderBy property, it will contain column name to sort.

Create Paging folder in the Common folder. In the Paging folder, add a PagedList class with following code.

```cs
public class PagedList<T> : List<T>
{
  public MetaData MetaData { get; set; }
  public PagedList(List<T> items, int count, int pageNumber, int pageSize)
  {
    MetaData = new MetaData
    {
      TotalCount = count,
      PageSize = pageSize,
      CurrentPage = pageNumber,
      TotalPages = (int)Math.Ceiling(count / (double)pageSize)
    };
    AddRange(items);
  }

  public PagedList(List<T> items, MetaData metaData)
  {
    this.MetaData = metaData;
    AddRange(items);
  }

  public static PagedList<T> ToPagedList(IEnumerable<T> source, int pageNumber,
      int pageSize)
  {
    var count = source.Count();
    var items = source
      .Skip((pageNumber - 1) * pageSize)
      .Take(pageSize)
      .ToList();
    return new PagedList<T>(items, count, pageNumber, pageSize);
  }
}
```

You will get errors on MetaData, add a new class MetaData in the Paging folder.

```cs
public class MetaData
{
  public int CurrentPage { get; set; }
  public int TotalPages { get; set; }
  public int PageSize { get; set; }
  public int TotalCount { get; set; }

  public bool HasPrevious => CurrentPage > 1;
  public bool HasNext => CurrentPage < TotalPages;
}
```

The PagedList<T> class is a generic class, of type List<T>. It seems a bit complex. We will pass the result of FindAll() method to PagedList class. PagedList will use Skip and Take methods to return only those records, which are on the required page number and page size.

Add a new class OrderQueryBuilder in the Paging folder and update it as follows.

```cs
public static class OrderQueryBuilder
{
  public static string CreateOrderQuery<T>(string orderBy)
  {
    var orderParams = orderBy.Trim().Split(',');
    var propertyInfos = typeof(T).GetProperties(BindingFlags.Public |
      BindingFlags.Instance);
    var orderByQueryBuilder = new StringBuilder();
    foreach (var param in orderParams)
    {
      if (string.IsNullOrWhiteSpace(param))
        continue;

      var propertyFromQueryName = param.Split(" ")[0];
      var objectProperty = propertyInfos.FirstOrDefault(pi =>
        pi.Name.Equals(propertyFromQueryName, StringComparison.InvariantCultureIgnoreCase));

      if (objectProperty == null)
        continue;

      var direction = param.EndsWith(" desc") ? "descending" : "ascending";

      orderByQueryBuilder.Append($"{objectProperty.Name.ToString()} {direction}, ");
    }

    var orderQuery = orderByQueryBuilder.ToString().TrimEnd(',', ' ');

    return orderQuery;
  }
}
```

This is for sorting. The orderBy string can contain multiple columns separate with comma. It also supports sorting in both ascending and descending order.

Finally the last class in the Common\Paging folder is ApiOkPagedResponse.

```cs
public sealed class ApiOkPagedResponse<ResultList, ResultMetaData>
{
  public ResultList PagedList { get; set; }
  public ResultMetaData MetaData { get; set; }
  public ApiOkPagedResponse(ResultList pagedList, ResultMetaData metadata)
  {
    PagedList = pagedList;
    MetaData = metadata;
  }
}
```

All methods that implement searching, paging and sorting will return PagedList and MetaData. MetaData contains information about the current page, total pages, count etc. Based on MetaData the client can handle paging previous, next, forward search parameters.

## Add search, sort and paging in PersonRepository

The RepositoryBase class does almost 80% of the work. Open the PersonRepository class. Currently it does not have its own methods, it inherits from RepositoryBase and calls the methods of base class for CRUD operations.

Searching and sorting are unique to each repository. They cannot be implemented in the base class. Searching in each repository will be different, we search text in first name, last name in the Person table. In Country table, we will search text in country name. In product table, we will search text in product name, description etc. So it cannot be generalized.

Same is the case with sorting, it works on different columns, each repository will implement its own search, paging and sorting.

We also created a generic PagedReq class. Each repository will have its own Dto for the search, page, sort request. Lets create this Dto for Person.

Open Dtos\Person.cs file and add the PersonReqSearch class as follows.

```cs
public class PersonReqSearch : PagedReq 
{
  [MaxLength(1)]
  public string? Gender { get; set; }
}
```

This is the request dto for searching, paging and sorting Person. We name the class just PersonReqSearch, as by default every search will include paging and sorting.

PersonReqSearch inherits from PagedReq, which includes default searchText, page and orderBy parameters. On top of these, it has its own member Gender. So user can search by free text and gender.

Open IPersonRepository interface and add new Search method as follows.

```cs
public interface IPersonRepository : IRepositoryBase<Entities.Person>
{
  PagedList<Entities.Person> Search(PersonReqSearch dto, bool trackChanges);
}
```

The search method takes the search request dto in parameter. It returns PagedList<Person>

Lets implement this method. Open PersonRepository class and add the Search method as below.

```cs
public PagedList<Entities.Person> Search(PersonReqSearch dto, bool trackChanges)
{
  var entities = FindAll(trackChanges)
    .Search(dto)
    .Sort(dto.OrderBy)
    .Skip((dto.PageNumber - 1) * dto.PageSize)
    .Take(dto.PageSize)
    .ToList();
  var count = FindAll(trackChanges)
    .Search(dto)
    .Count();
  return new PagedList<Entities.Person>(entities, count,
    dto.PageNumber, dto.PageSize);
}
```

The search method in repository calls FindAll to return all records, then call search to filter the records. Then sort the records. Then call Skip and Take methods for paging. Finally ToList() is called. The Entity Framework will execute the SQL on ToList() method, so the final SQL will contain all the where conditions with paging and sorting.

We also need to send total count for the search results. So we need to execute the count on the search again.

You will get errors because there is no Search and Sort methods for IQueryable<Person> returned by FindAll(). We need to add these methods. It is best to add these as extension methods, to keep the PersonRepository clean.

Create a new class **PersonRepositoryExtensions** in the Repository folder and update it as follows.

```cs
public static class PersonRepositoryExtensions
{
  public static IQueryable<Entities.Person> Search(this IQueryable<Entities.Person> items,
      PersonReqSearch searchParams)
  {
    var itemsToReturn = items
      .AsQueryable();

    if (string.IsNullOrWhiteSpace(searchParams.SearchText) == false)
    {
      itemsToReturn = itemsToReturn.Where(
        x => x.FirstName.Contains(searchParams.SearchText) ||
        x.LastName.Contains(searchParams.SearchText)
      );
    }
    if (string.IsNullOrWhiteSpace(searchParams.Gender) == false)
    {
      itemsToReturn = itemsToReturn.Where(
        x => x.Gender == searchParams.Gender);
    }
    return itemsToReturn;
  }
  public static IQueryable<Entities.Person> Sort(this IQueryable<Entities.Person> items,
      string? orderBy)
  {
    if (string.IsNullOrWhiteSpace(orderBy))
      return items.OrderBy(e => e.FirstName);

    var orderQuery = OrderQueryBuilder.CreateOrderQuery<Entities.Person>(orderBy);

    if (string.IsNullOrWhiteSpace(orderQuery))
      return items.OrderBy(e => e.FirstName);

    return items.OrderBy(orderQuery);
  }
}
```

Build the project, you will still get errors on OrderBy method. For this to work, we need to install System.Linq.Dynamic.Core by ZZZ.


![system linq dynamic core](/images/blog/system.linq_.dynamic.core_-1024x96.jpg "system linq dynamic core")

After installing this NuGet package, add its reference on top.

```cs
using System.Linq.Dynamic.Core;
```

Now build the project, it should build without any errors this time.

We have done the hard work of creating the base classes and adding the Search and Sort method in the PersonRepository. Now we can add the Search method in the service.

Open IPersonService interface and add the search method in it as follows.

```cs
ApiOkPagedResponse<IEnumerable<PersonRes>, MetaData>
            Search(PersonReqSearch dto);
```

Open PersonService class to implement it as follows.

```cs
public ApiOkPagedResponse<IEnumerable<PersonRes>, MetaData>
    Search(PersonReqSearch dto)
{
  var pagedEntities = _repositoryManager.PersonRepository.
    Search(dto, false);
  var dtos = _mapper.Map<IEnumerable<PersonRes>>(pagedEntities);
  return new ApiOkPagedResponse<IEnumerable<PersonRes>, MetaData>(dtos,
    pagedEntities.MetaData);
}
```

Nothing really special here. The service calls the Search method in PersonRepository, passes the dto to it. The service transforms the entity to Person response dto and prepares the result in ApiOkPagedResponse class, which consists of PagedList and MetaData.

Now open PersonsController and add the search method as follows.

```cs
[HttpGet("search")]
public IActionResult Search([FromQuery] PersonReqSearch dto)
{
  var res = _personService.Search(dto);
  return Ok(res);
}
```

The method in controller is really simple as other methods we wrote previously. This time we used [FromQuery] in search dto parameter. As we want this dto to be set via URL querystring.

Run the project, it will open the default Swagger UI. Now you should see total 7 methods, including the new Search method.

![search api](/images/blog/search-api-1024x474.jpg "search api")

Click on search, try it out, try different parameters in the Search and see how it works.

That’s good enough for getting started with building a basic API. We will write another series for writing a frontend application to work with the API we wrote in these tutorials.