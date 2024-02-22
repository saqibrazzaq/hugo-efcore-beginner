---
title: 4. DTOs and Repository classes for all Entities
weight: 550
ShowToc: true
TocOpen: true
---

## Country DTOs

In general we create three DTOs for an entity.

- EntityRes – for sending response data to the API client
- EntityReqEdit – for create and update, data is received from client in request body
- EntityReqSearch – for searching, paging and sorting, data received from client in get request

When we created [DTOs for Person in the beginner series](/person-api/service-uses-repository/#dtos-for-the-person-entity), we created these 3 DTOs. Person entity just had 5 fields. But Country entity has 18 fields and 3 collections for child rows. We will start with the three basic DTOs, but we will be needing more DTOs as well when we build some pages, we will get there soon. For now, lets create the basic three DTOs for Country as follows in the Dtos folder. You can get the source of Country.cs from https://github.com/saqibrazzaq/efcorebeginner/blob/main/AddressBook/AddressBook/Dtos/Country.cs.

```cs
public class CountryRes
{
  public int CountryId { get; set; }
  public string? Name { get; set; }
  public string? Iso3 { get; set; }
  public string? iso2 { get; set; }
  public string? NumericCode { get; set; }
  public string? PhoneCode { get; set; }
  public string? Capital { get; set; }
  public string? Currency { get; set; }
  public string? CurrencyName { get; set; }
  public string? CurrencySymbol { get; set; }
  public string? Tld { get; set; }
  public string? Native { get; set; }
  public string? Region { get; set; }
  public string? SubRegion { get; set; }
  public double Latitude { get; set; }
  public double Longitude { get; set; }
  public string? Emoji { get; set; }
  public string? EmojiU { get; set; }


  public IEnumerable<Timezone>? Timezones { get; set; }
  public IEnumerable<Translation>? Translations { get; set; }
  public IEnumerable<State>? States { get; set; }
}

public class CountryReqEdit
{
  [Required]
  public string? Name { get; set; }
  [Required, MaxLength(3)]
  public string? Iso3 { get; set; }
  [Required, MaxLength(2)]
  public string? iso2 { get; set; }
  public string? NumericCode { get; set; }
  public string? PhoneCode { get; set; }
  public string? Capital { get; set; }
  public string? Currency { get; set; }
  public string? CurrencyName { get; set; }
  public string? CurrencySymbol { get; set; }
  public string? Tld { get; set; }
  public string? Native { get; set; }
  public string? Region { get; set; }
  public string? SubRegion { get; set; }
  public double Latitude { get; set; }
  public double Longitude { get; set; }
  public string? Emoji { get; set; }
  public string? EmojiU { get; set; }
}

public class CountryReqSearch : PagedReq
{

}
```

CountryRes contains same fields as Country entity. We have no confidential information, so we included all fields from the entity. CountryReqEdit has all fields, except the CountryId.

CountryReqSearch inherits from PagedReq. PagedReq is generic class for requesting search data. We will also need Dtos for sending response of search/paged result, so we will create response DTOs as well. We covered these in very detail in https://efcorebeginner.com/person-api/searching-paging-and-sorting/ article. Here we will just mention the names of request and response DTOs with their respective links on GitHub.

- PagedReq – Generic class for search, sort and paging query
- ApiOkPagedResponse – Generic response Dto for any entity/class
- PagedList – Ordered List of entity having a single page
- MetaData – Contains metadata about the paged result like current page, total count etc.
- OrderQueryBuilder – For creating ordered query

These classes are created in the Common folder, you can find all these on [GitHub repository](https://github.com/saqibrazzaq/efcorebeginner). These should be easy to create and understand, if not, then I would strongly recommend you to follow the [beginner series of ASP.NET Core Web API](/person-api/), where we built an API from scratch, explaining everything in detail.

## Country Repository

We create **one repository for each entity**. All repositories are inherited from the base repository class. All basic select, insert, update and delete methods are implemented in the base repository class. Any repository can call base classs’ methods to do simple CRUD. Searching, paging and sorting is different, it cannot be added in the base class. So each repository implements its own paging, sorting and searching.

Creating Repository for each entity is also same as [we already did for the Person entity](/person-api/person-repository/). We will not go into details here. Lets create our first repository for Country as follows.

```cs
public interface ICountryRepository : IRepositoryBase<Country>
{
  PagedList<Country> Search(CountryReqSearch dto, bool trackChanges);
}

public class CountryRepository : RepositoryBase<Country>, ICountryRepository
{
  public CountryRepository(AppDbContext context) : base(context)
  {
  }

  public PagedList<Country> Search(CountryReqSearch dto, bool trackChanges)
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
    return new PagedList<Country>(entities, count,
      dto.PageNumber, dto.PageSize);
  }
}

public static class CountryRepositoryExtensions
{
  public static IQueryable<Country> Search(this IQueryable<Country> items,
      CountryReqSearch searchParams)
  {
    var itemsToReturn = items
      .AsQueryable();

    if (string.IsNullOrWhiteSpace(searchParams.SearchText) == false)
    {
      itemsToReturn = itemsToReturn.Where(
        x => x.Name.Contains(searchParams.SearchText) ||
        x.iso2.Contains(searchParams.SearchText) ||
        x.Iso3.Contains(searchParams.SearchText) ||
        x.PhoneCode.Contains(searchParams.SearchText)
      );
    }

    return itemsToReturn;
  }
  public static IQueryable<Country> Sort(this IQueryable<Country> items,
      string? orderBy)
  {
    if (string.IsNullOrWhiteSpace(orderBy))
      return items.OrderBy(e => e.Name);

    var orderQuery = OrderQueryBuilder.CreateOrderQuery<Country>(orderBy);

    if (string.IsNullOrWhiteSpace(orderQuery))
      return items.OrderBy(e => e.Name);

    return items.OrderBy(orderQuery);
  }
}
```

We implemented own search for Country Repository. Each repository will have its own search implementation in extension class, as you can see that Country search works on Name, iso2, iso3 and phone codes. Other repositories will do search on their own columns and may have different conditions.

To get detailed explanation of the Search and sort implementation, please see [this article](/person-api/searching-paging-and-sorting/) in detail.

At this stage, we have created the following files/folders in the API project.

![address book structure after creating country repository](/images/blog/address-book-structure-after-creating-country-repository.jpg "address book structure after creating country repository")

You can check and verify from https://github.com/saqibrazzaq/efcorebeginner/tree/main/AddressBook.

In Person basic project, we had only one small entity, now we have 5 entities so far (some more will be added). Now the entities also contain more fields of different types. We just created Country repository with its search/sort extension methods. By following the same approach, you may continue to create repositories for other entities.

## State DTOs

As usual, we will create three DTOs for state.

- StateRes – Contains all properties of the State entity
- StateReqEdit – All properties of the State entity except the StateId
- StateReqSearch – Inherits from PagedReq. It has its own property CountryId. PagedReq contains searchText, which is available to all. We specifically need CountryId here, so that we can search states in a specific country.

The source code can be viewed at GitHub.

## State Repository

We create Repository interface, class and extension methods for State in the same way as we created for Country. The code is same, except the Search extension method. Country’s search text queries on Name, iso2, iso3 and phone codes, whereas State search text queries on Name and Code fields. The state search method also has where condition for country id. See the code snippet below for just the Search extension method.

```cs
public static IQueryable<State> Search(this IQueryable<State> items,
            StateReqSearch searchParams)
{
  var itemsToReturn = items
      .AsQueryable();

  if (string.IsNullOrWhiteSpace(searchParams.SearchText) == false)
  {
    itemsToReturn = itemsToReturn.Where(
        x => x.Name.Contains(searchParams.SearchText) ||
        x.Code.Contains(searchParams.SearchText)
    );
  }

  if (searchParams.CountryId != null)
  {
    itemsToReturn = itemsToReturn.Where(
        x => x.CountryId == searchParams.CountryId);
  }

  return itemsToReturn;
}
```

You can view the complete source code in [GitHub repository](https://github.com/saqibrazzaq/efcorebeginner).

## City, Timezone and Translation DTOs

We will add three DTOs for City, Timezone and Translation each as follows.

- CityRes
- CityReqEdit
- CityReqSearch
- TimezoneRes
- TimezoneReqEdit
- TimezoneReqSearch
- TranslationRes
- TranslationReqEdit
- TranslationReqSearch

The pattern is same as City DTOs. You can find the source code at https://github.com/saqibrazzaq/efcorebeginner/tree/main/AddressBook/AddressBook/Dtos.

## City, Timezone and Translation Repositories

Like we add repository for Country, we will add the repository interface, class and extension methods for search and sort for City, Timezone and Translation entities. We will follow the same pattern as used by Country Repository. Create the new classes in the Repository folder as follows.

- ICityRepository
- CityRepository
- ITimezoneRepository
- TimezoneRepository
- ITranslationRepository
- TranslationRepository

The source code for the above repositories is available at https://github.com/saqibrazzaq/efcorebeginner/tree/main/AddressBook/AddressBook/Repository.

## Mapping between Entities and DTOs

We added AutoMapper NuGet package for converting entities into DTOs and vice versa. Why we need DTOs and conversion, to read [this article from the beginner series](/person-api/service-uses-repository/) for details. Open MappingProfile.cs file in project root, in the constructor add the following code for the mapping.

```cs
public class MappingProfile : Profile
{
  public MappingProfile()
  {
    // Country
    CreateMap<Country, CountryRes>();
    CreateMap<CountryReqEdit, Country>();

    // State
    CreateMap<State, StateRes>();
    CreateMap<StateReqEdit, State>();

    // City
    CreateMap<City, CityRes>();
    CreateMap<CityReqEdit, City>();

    // Timezone
    CreateMap<Timezone, TimezoneRes>();
    CreateMap<TimezoneReqEdit, Timezone>();

    // Translation
    CreateMap<Translation, TranslationRes>();
    CreateMap<TranslationReqEdit, Translation>();
  }
}
```

