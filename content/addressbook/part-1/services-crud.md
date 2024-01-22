---
title: 5. Service classes for City, state and country entities – CRUD operations
weight: 560
ShowToc: true
TocOpen: true
---

So far we have written the repositories for all 5 tables that implements basic CRUD operations. Now we will write service classes for all 5 entities, including city, state and country.

## Service for Country

Add a new interface **ICountryService** in **Services** folder. Add the following methods in this interface.

```cs
public interface ICountryService
{
  CountryRes Create(CountryReqEdit dto);
  CountryRes Update(int countryId, CountryReqEdit dto);
  void Delete(int countryId);
  CountryRes Get(int countryId);
  int Count();
  ApiOkPagedResponse<IEnumerable<CountryRes>, MetaData>
      Search(CountryReqSearch dto);
}
```

There are 6 methods, 4 for basic CRUD and 2 for Search and count. First we will implement the basic methods as we did in the [beginner ASP.NET tutorial series](/person-api/), we will add complex queries involving related data later.

Create a new class **CountryService** in the **Services** folder. Add the method implementation as follows.

```cs
public class CountryService : ICountryService
{
  private readonly IRepositoryManager _repositoryManager;
  private readonly IMapper _mapper;

  public CountryService(IMapper mapper, 
      IRepositoryManager repositoryManager)
  {
    _mapper = mapper;
    _repositoryManager = repositoryManager;
  }

  public int Count()
  {
    return _repositoryManager.CountryRepository.FindAll(false)
        .Count();
  }

  public CountryRes Create(CountryReqEdit dto)
  {
    var entity = _mapper.Map<Country>(dto);
    _repositoryManager.CountryRepository.Create(entity);
    _repositoryManager.Save();
    return _mapper.Map<CountryRes>(entity);
  }

  public void Delete(int countryId)
  {
    var entity = FindCountryIfExists(countryId, true);
    _repositoryManager.CountryRepository.Delete(entity);
    _repositoryManager.Save();
  }

  private Country FindCountryIfExists(int countryId, bool trackChanges)
  {
    var entity = _repositoryManager.CountryRepository.FindByCondition(
        x => x.CountryId == countryId,
        trackChanges)
        .FirstOrDefault();
    if (entity == null) { throw new Exception("No country found with id " + countryId); }

    return entity;
  }

  public CountryRes Get(int countryId)
  {
    var entity = FindCountryIfExists(countryId, false);
    var dto = _mapper.Map<CountryRes>(entity);
    return dto;
  }

  public ApiOkPagedResponse<IEnumerable<CountryRes>, MetaData> Search(CountryReqSearch dto)
  {
    var pagedEntities = _repositoryManager.CountryRepository.
        Search(dto, false);
    var dtos = _mapper.Map<IEnumerable<CountryRes>>(pagedEntities);
    return new ApiOkPagedResponse<IEnumerable<CountryRes>, MetaData>(dtos,
        pagedEntities.MetaData);
  }

  public CountryRes Update(int countryId, CountryReqEdit dto)
  {
    var entity = FindCountryIfExists(countryId, true);
    _mapper.Map(dto, entity);
    _repositoryManager.Save();
    return _mapper.Map<CountryRes>(entity);
  }
}
```

## Services for City, State, Translation and Timezone

We will now write services for other entities, in similar way we created CountryService. The pattern is same, so we will not write all the code here. To get the code, you may refer or download the code from [GitHub repository](https://github.com/saqibrazzaq/efcorebeginner).

Below are the interface and classes that we created. Each class is linked to the respective code in GitHub.

- IStateService
- StateService
- ICityService
- CityService
- ITranslationService
- TranslationService
- ITimezoneService
- TimezoneService

## Register Services in extension method

We already added empty method **ConfigureServices** in **ServicesExtensions** class. Here we register all services. Update this method with the code below to register our services.

```cs
public static void ConfigureServices(this IServiceCollection services)
{
  services.AddScoped<ICountryService, CountryService>();
  services.AddScoped<IStateService, StateService>();
  services.AddScoped<ICityService, CityService>();
  services.AddScoped<ITranslationService, TranslationService>();
  services.AddScoped<ITimezoneService, TimezoneService>();
}
```