---
title: 3. Repositories for Contact and Related tables
weight: 730
ShowToc: true
TocOpen: true
---

After creating entities and dtos, the next step is to create repository classes for our entities. We will create the repositories for the 13 new entities created in this part.

## Repositories for Label Definitions

We have 6 entities for different labels. The label entities are the simplest. Each label entity contains an ID (primary key) and label (text value). Lets start with creating repository for the PhoneLabel entity.

To add a repository for an entity, we create 3 files as follows.

1. IPhoneLabelRepository – interface which inherits from IRepositoryBase
2. PhoneLabelRepository – implements the RepositoryBase and IPhoneLabelRepository
3. PhoneLabelRepositoryExtensions – contains specific methods for search and sort

In ASP.NET web API project’s Repository folder, create a new interface IPhoneLabelRepository and add the following code. It inherits from IRepositoryBase, so all CRUD methods are already inherited from parent. We just need to add definition for Search method.

```cs
public interface IPhoneLabelRepository : IRepositoryBase<PhoneLabel>
{
  PagedList<PhoneLabel> Search(PhoneLabelReqSearch dto, bool trackChanges);
}
```

Now create a new class PhoneLabelRepository in the same Repository folder and write the code for implementing this interface as follows. The CRUD methods are already implemented in the RepositoryBase class, which are inherited here. We only need to implement the Search method as below. 

```cs
public class PhoneLabelRepository : RepositoryBase<PhoneLabel>, IPhoneLabelRepository
{
  public PhoneLabelRepository(AppDbContext context) : base(context)
  {
  }

  public PagedList<PhoneLabel> Search(PhoneLabelReqSearch dto, bool trackChanges)
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
    return new PagedList<PhoneLabel>(entities, count,
        dto.PageNumber, dto.PageSize);
  }
}
```

Create a new file PhoneLabelRepositoryExtensions.cs. We will add two extension methods for Search and Sort here. Copy the code as follows. The Search extension method is very simple, it searches Label field with the searchText and applies where condition to the query.

```cs
public static class PhoneLabelRepositoryExtensions
{
  public static IQueryable<PhoneLabel> Search(this IQueryable<PhoneLabel> items,
      PhoneLabelReqSearch searchParams)
  {
    var itemsToReturn = items
        .AsQueryable();

    if (string.IsNullOrWhiteSpace(searchParams.SearchText) == false)
    {
        itemsToReturn = itemsToReturn.Where(
            x => x.Label.Contains(searchParams.SearchText) 
        );
    }

    return itemsToReturn;
  }
  public static IQueryable<PhoneLabel> Sort(this IQueryable<PhoneLabel> items,
      string? orderBy)
  {
    if (string.IsNullOrWhiteSpace(orderBy))
        return items.OrderBy(e => e.Label);

    var orderQuery = OrderQueryBuilder.CreateOrderQuery<PhoneLabel>(orderBy);

    if (string.IsNullOrWhiteSpace(orderQuery))
        return items.OrderBy(e => e.Label);

    return items.OrderBy(orderQuery);
  }
}
```

Go ahead and create the repository interface, class and extension methods like above, for the remaining label repositories. For reference you may check the link https://github.com/saqibrazzaq/efcorebeginner/blob/main/AddressBook/AddressBook/Repository

- IAddressLabelRepository, AddressLabelRepository, AddressLabelRepositoryExtensions
- IChatLabelRepository, ChatLabelRepository, ChatLabelRepositoryExtensions
- IEmailLabelRepository, EmailLabelRepository, EmailLabelRepositoryExtensions
- ILabelRepository, LabelRepository, LabelRepositoryExtensions
- IPhoneLabelRepository, PhoneLabelRepository, PhoneLabelRepositoryExtensions
- IWebsiteLabelRepository, WebsiteLabelRepository, WebsiteLabelRepositoryExtensions

## Repository for Contact

This is the main repository to do CRUD on Contacts. Like all other repositories, we will have an interface, an implementation class and extension methods for searching and sorting. Create a new interface in the Repository folder named IContactRepository and add the following code.

```cs
public interface IContactRepository : IRepositoryBase<Contact>
{
  PagedList<Contact> Search(ContactReqSearch dto, bool trackChanges);
}
```

Now add a new class ContactRepository, which will implement the RepositoryBase class and IContactRepository interface. We only have to implement the Search method, the rest are already implemented in the RepositoryBase class. Add the following code in the ContactRepository class.

```cs
public class ContactRepository : RepositoryBase<Contact>, IContactRepository
{
  public ContactRepository(AppDbContext context) : base(context)
  {
  }

  public PagedList<Contact> Search(ContactReqSearch dto, bool trackChanges)
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
    return new PagedList<Contact>(entities, count,
      dto.PageNumber, dto.PageSize);
  }
}
```

The above code will give errors on Search and Sort methods, because they do not exist yet. Create a new file ContactRepositoryExtensions.cs and add the following code to add the extension methods.

```cs
public static class ContactRepositoryExtensions
{
  public static IQueryable<Contact> Search(this IQueryable<Contact> items,
      ContactReqSearch searchParams)
  {
    var itemsToReturn = items
        .Include(x => x.ContactPhones.Take(1))
        .Include(x => x.ContactEmails.Take(1))
        .AsQueryable();

    if (searchParams.LabelId != null)
    {
        itemsToReturn = itemsToReturn.Where(
            x => x.ContactLabels.Any(y => y.LabelId == searchParams.LabelId));
    }

    if (string.IsNullOrWhiteSpace(searchParams.SearchText) == false)
    {
        itemsToReturn = itemsToReturn.Where(
            x => x.FirstName.Contains(searchParams.SearchText) ||
            x.LastName.Contains(searchParams.SearchText) ||
            x.MiddleName.Contains(searchParams.SearchText) ||
            x.JobTitle.Contains(searchParams.SearchText) ||
            x.Company.Contains(searchParams.SearchText) ||
            x.Department.Contains(searchParams.SearchText) ||
            x.ContactPhones.Any(ph => ph.Phone.Contains(searchParams.SearchText)) ||
            x.ContactEmails.Any(em => em.Email.Contains(searchParams.SearchText)) ||
            x.ContactChats.Any(ch => ch.Chat.Contains(searchParams.SearchText)) ||
            x.ContactAddresses.Any(ad => ad.Line1.Contains(searchParams.SearchText) 
                || ad.Line2.Contains(searchParams.SearchText)
                || ad.City.Name.Contains(searchParams.SearchText)
                || ad.City.State.Name.Contains(searchParams.SearchText)
                || ad.City.State.Country.Name.Contains(searchParams.SearchText)
                || ad.City.State.Country.iso2.Equals(searchParams.SearchText)
                || ad.City.State.Country.Iso3.Equals(searchParams.SearchText)
            )
        );
    }

    return itemsToReturn;
  }
  public static IQueryable<Contact> Sort(this IQueryable<Contact> items,
      string? orderBy)
  {
      if (string.IsNullOrWhiteSpace(orderBy))
          return items.OrderBy(e => e.FirstName);

      var orderQuery = OrderQueryBuilder.CreateOrderQuery<Contact>(orderBy);

      if (string.IsNullOrWhiteSpace(orderQuery))
          return items.OrderBy(e => e.FirstName);

      return items.OrderBy(orderQuery);
  }
}
```

Now this implementation of Search method looks very different than the other methods we implemented so far. Lets discuss one by one.

### Include Data from Related Tables in Repository

We have called Include method, to get data from child tables of Contact. Include is called two times, once for ContactPhone and then for ContactEmail table. Also note that Include method is chained with Take method like Take(1).

.Include(x => x.ContactPhones.Take(1)) means Get only first record from the child table ContactPhone. If we remove .Take(1), then the Include method will return all the phone numbers for this contact.

Why Take(1)? Lets have a look at the following screenshot first, to see the requirement.

![contact search include related data](/images/blog/contact-search-include-related-data.jpg "contact search include related data")

Our Contact search screen will look like above. It is implemented in React. React will call Axios API helper method to call /contacts/search web API, which will call the controller, then ContactService, then the request will finally reach the ContactRepository.Search extension method. ProfilePicture Url, First Name and Last Name are available in the ContactRepository. A contact may have 0 or many email addresses, 0 or many phone numbers. But we only need one email and one phone to display in the search page. So that is why we chain Include method with Take, to get the first record only.

### Where condition with Fields from Related Tables

Note the following code in the Search method.

```cs
if (searchParams.LabelId != null)
{
  itemsToReturn = itemsToReturn.Where(
      x => x.ContactLabels.Any(y => y.LabelId == searchParams.LabelId));
}
```

If search params have LabelId, then we match it with the label Id in the ContactLabel table. The LabelId is not part of Contact table, it is in the child table ContactLabel. And also note that we have used Any() method instead of ==, because one contact may have 0 or more labels. We cannot use x.ContactLabels.ContactLabelId, because x.ContactLabels is an array, having 0 or more items. So in this case, we use Any, and then match the LabelId again with LINQ.

Have a look at the database relation diagram again.

![contact label match labelid](/images/blog/Contact-Label-match-labelId.jpg "contact label match labelid")

If we were using SQL and we would like to get all Contacts whose labelId is 1, the SQL would have been like below.

```sql
SELECT *
FROM Contact c
JOIN ContactLabel cl ON cl.ContactId = c.ContactId
WHERE c.ContactId = 7 AND cl.LabelId = 1
```

Its equivalent in LINQ would be

```cs
_repositoryManager.ContactRepository.FindByCondition(
  x => x..ContactLabels.Any(y => y.LabelId == searchParams.LabelId),
  false
)
```

The LINQ way is better because we get data in object format in our entity. We can transform easily in dto. We can send the Dto to React or any other client in Json format. The React can read Json and can easily transform Json into TypeScript dto. This way we also have data in object format in both server side (ASP.NET web API) and client side(React/Angular etc).

Then note how we match the searchText with text fields in the main Contacts table, as well as other related tables.

If we want to add where condition which matches with fields of same table, it is very easy, just use x => x.FieldName.Contains(text).

To get clear idea, see the gif animation below.

![contact search phone email](/images/blog/contact-search-phone-email.gif "contact search phone email")

First we search 7000 in the search field. It matches with ContactPhone.Phone field with x.ContactPhones.Any(), because the child table has multiple phone numbers, so we first use Any() method, then match with ph => ph.Phone field.

We can also add where condition with distant related table e.g. we typed California, which is state name. The Contact has multiple addresses, so first we use Any() with ContactAddress. Then again in Any() method, we can match ad.City.State.Name field.

LINQ is very powerful once you learn how to use it. With SQL, the query may become complex, the SQL also returns the data in tabular format, it is hard to transform tabular data in object format. But with LINQ, our data is readily available in Object format when we use Repository and Entity Framework.

```cs
if (string.IsNullOrWhiteSpace(searchParams.SearchText) == false)
{
  itemsToReturn = itemsToReturn.Where(
    x => x.FirstName.Contains(searchParams.SearchText) ||
    x.LastName.Contains(searchParams.SearchText) ||
    x.MiddleName.Contains(searchParams.SearchText) ||
    x.JobTitle.Contains(searchParams.SearchText) ||
    x.Company.Contains(searchParams.SearchText) ||
    x.Department.Contains(searchParams.SearchText) ||
    x.ContactPhones.Any(ph => ph.Phone.Contains(searchParams.SearchText)) ||
    x.ContactEmails.Any(em => em.Email.Contains(searchParams.SearchText)) ||
    x.ContactChats.Any(ch => ch.Chat.Contains(searchParams.SearchText)) ||
    x.ContactAddresses.Any(ad => ad.Line1.Contains(searchParams.SearchText) 
        || ad.Line2.Contains(searchParams.SearchText)
        || ad.City.Name.Contains(searchParams.SearchText)
        || ad.City.State.Name.Contains(searchParams.SearchText)
        || ad.City.State.Country.Name.Contains(searchParams.SearchText)
        || ad.City.State.Country.iso2.Equals(searchParams.SearchText)
        || ad.City.State.Country.Iso3.Equals(searchParams.SearchText)
    )
  );
}
```

## Repository for Contact Phone

As usual create 3 files, including the interface, implementation class and extension methods files for the ContactPhone repository. CRUD methods are inherited from RepositoryBase class, so we will discuss the Search method implementation here.

```cs
public static class ContactPhoneRepositoryExtensions
{
  public static IQueryable<ContactPhone> Search(this IQueryable<ContactPhone> items,
      ContactPhoneReqSearch searchParams)
  {
    var itemsToReturn = items
        .Include(x => x.Country)
        .Include(x => x.PhoneLabel)
        .AsQueryable();

    if (searchParams.ContactId != null)
    {
        itemsToReturn = itemsToReturn.Where(
            x => x.ContactId == searchParams.ContactId);
    }

    if (string.IsNullOrWhiteSpace(searchParams.SearchText) == false)
    {
        itemsToReturn = itemsToReturn.Where(
            x => x.Phone.Contains(searchParams.SearchText)
        );
    }

    return itemsToReturn;
  }
  public static IQueryable<ContactPhone> Sort(this IQueryable<ContactPhone> items,
      string? orderBy)
  {
    // sort method
  }
}
```

It includes two related tables, Country and PhoneLabel. A phone may have an optional country code and label. Below screenshot displays two phone numbers. One phone number is saved with country code and label. The other record just has the phone number, there is no country code or label selected, because they are optional. ContactPhone has IDs of both country and label, so when we do search query, we have to include both Country and PhoneLabel related tables, so that they are included in the dtos.

The search params has ContactId required field, we match it with ContactPhone.ContactId in the Where condition.

![phone country code label](/images/blog/phone-country-code-label.jpg "phone country code label")

Just like we created the interface, class and extension methods for the Contact Phone repository, you can go ahead and create the repository classes for the rest of the properties. These are listed below.

- IContactLabelRepository, ContactLabelRepository, ContactLabelRepositoryExtensions
- IContactEmailRepository, ContactEmailRepository, ContactEmailRepositoryExtensions
- IContactPhoneRepository, ContactPhoneRepository, ContactPhoneRepositoryExtensions
- IContactAddressRepository, ContactAddressRepository, ContactAddressRepositoryExtensions
- IContactWebsiteRepository, ContactWebsiteRepository, ContactWebsiteRepositoryExtensions
- IContactChatRepository, ContactChatRepository, ContactChatRepositoryExtensions

You can find all the repositories in this [GitHub folder](https://github.com/saqibrazzaq/efcorebeginner/tree/main/AddressBook/AddressBook/Repository).

## Update RepositoryManager

Whenever we add a new repository, it must be added to the RepositoryManager. We can access any repository from a single instance of RepositoryManager, so it must contain all the repositories. Now the IRepositoryManager interface should have 18 member variables for all the repositories as follows.

```cs
public interface IRepositoryManager
{
  ICountryRepository CountryRepository { get; }
  IStateRepository StateRepository { get; }
  ICityRepository CityRepository { get; }
  ITranslationRepository TranslationRepository { get; }
  ITimezoneRepository TimezoneRepository { get; }
  IContactRepository ContactRepository { get; }
  ILabelRepository LabelRepository { get; }
  IContactLabelRepository ContactLabelRepository { get; }
  IContactEmailRepository ContactEmailRepository { get; }
  IEmailLabelRepository EmailLabelRepository { get; }
  IContactPhoneRepository ContactPhoneRepository { get; }
  IPhoneLabelRepository PhoneLabelRepository { get; }
  IContactAddressRepository ContactAddressRepository { get; }
  IAddressLabelRepository AddressLabelRepository { get; }
  IContactWebsiteRepository ContactWebsiteRepository { get; }
  IWebsiteLabelRepository WebsiteLabelRepository { get; }
  IContactChatRepository ContactChatRepository { get; }
  IChatLabelRepository ChatLabelRepository { get; }
  void Save();
}
```

These new members must also be defined in the RepositoryManager class. Update the RepositoryManager as follows. We have not included all the 18 instance members, because the code would grow large. We have only initialized 3 members. You can write the code for the rest of the repositories in the similar manner. The complete code is available on [GitHub](https://github.com/saqibrazzaq/efcorebeginner/tree/main/AddressBook/AddressBook/Repository).

```cs
public class RepositoryManager : IRepositoryManager
{
  private readonly AppDbContext _context;
  private readonly Lazy<ICountryRepository> _countryRepository;
  private readonly Lazy<IContactPhoneRepository> _contactPhoneRepository;
  private readonly Lazy<IPhoneLabelRepository> _phoneLabelRepository;
  // other repositories
  public RepositoryManager(AppDbContext context)
  {
      _context = context;

      // Initialize all the repositories
      _countryRepository = new Lazy<ICountryRepository>(() =>
          new CountryRepository(context));
      _contactPhoneRepository = new Lazy<IContactPhoneRepository>(() =>
          new ContactPhoneRepository(context));
      _phoneLabelRepository = new Lazy<IPhoneLabelRepository>(() =>
          new PhoneLabelRepository(context));
      // initialize the rest
    }

    public ICountryRepository CountryRepository => _countryRepository.Value;
    public IContactPhoneRepository ContactPhoneRepository => _contactPhoneRepository.Value;
    public IPhoneLabelRepository PhoneLabelRepository => _phoneLabelRepository.Value;
    // define the rest

    public void Save()
    {
        _context.SaveChanges();
    }
}
```