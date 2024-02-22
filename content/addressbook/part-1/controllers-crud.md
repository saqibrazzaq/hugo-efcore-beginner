---
title: 6. Controllers for City, State and Country – CRUD and Search operations
weight: 570
ShowToc: true
TocOpen: true
---

We have written entity, repository and service classes for our 5 entities. Now we will add controllers for these basic operations. Controllers are the endpoints that make the URL. All the business related functions are implemented in the Service classes. The controllers should be dumb. The role of the controllers should be to handle the requests between client and Services.

Add a new class named CountryController in Controllers folder. Add ICountryService interface as private member and initialize it in the constructor (dependency injection). Add the http methods as follows.

```cs
public class CountriesController : ControllerBase
{
  private readonly ICountryService _countryService;

  public CountriesController(ICountryService countryService)
  {
    _countryService = countryService;
  }

  [HttpGet]
  public IActionResult Default()
  {
    return Search(new CountryReqSearch());
  }

  [HttpGet("search")]
  public IActionResult Search([FromQuery] CountryReqSearch dto)
  {
    var res = _countryService.Search(dto);
    return Ok(res);
  }

  [HttpGet("count")]
  public IActionResult Count()
  {
    var res = _countryService.Count();
    return Ok(res);
  }

  [HttpGet("{countryId}")]
  public IActionResult Get(int countryId)
  {
    var res = _countryService.Get(countryId);
    return Ok(res);
  }

  [HttpPost]
  public IActionResult Create(CountryReqEdit dto)
  {
    var res = _countryService.Create(dto);
    return Ok(res);
  }

  [HttpPut("{countryId}")]
  public IActionResult Update(int countryId, CountryReqEdit dto)
  {
    var res = _countryService.Update(countryId, dto);
    return Ok(res);
  }

  [HttpDelete("{countryId}")]
  public IActionResult Delete(int countryId)
  {
    _countryService.Delete(countryId);
    return NoContent();
  }
}
```

These are very basic controllers for Country. We will not go in to much details. If you don’t know how URLs are mapped with methods, it is recommended to read in detail, how to create a [basic controller in Controller to create new Person](/person-api/controller-create-person/) article.

Follow the same pattern and create other controllers for city, state etc. You can see the code for the controllers in [this folder on GitHub](https://github.com/saqibrazzaq/efcorebeginner/tree/main/AddressBook/AddressBook/Controllers).

Debug or Run the project, it should build successfully and open the default Swagger UI on localhost. You can add new country, state, city, translation and timezones from the Swagger UI.

![address book swagger](/images/blog/address-book-swagger-1024x611.jpg "address book swagger")

So far, we have one repository, one service and one controller for an entity. It is very simple. But in real world we have summary and detail reports, which includes reading data from multiple tables/entities. We have not created any summary or detail queries or reports yet. Lets first create UI for country, state and city, then we will create use cases where we will need data from multiple tables in our UI.