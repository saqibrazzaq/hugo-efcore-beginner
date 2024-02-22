---
title: Massively Improve Search Query Performance by Index Columns
ShowToc: true
TocOpen: true
cover:
  image: "/images/blog/performance-search-index.jpg"
  caption: "Photo by Marc-Olivier Jodoin on Unsplash"
---

While making the [AddressBook app](/addressbook/part-2/), I wanted to work on more than one million rows in a database. The goal is to make it work on a cheap 2 core VPS, under $15 monthly budget. After adding 1M+ rows, the search queries were not performing well. On a fresh install the response time ranged from 1 to 10 seconds. But I got some other sites hosted on the same VPS, so when I checked back the AddressBook after some time, the Search was not working at all. I was getting timeout errors.

**What is the first thing to do when you get slow performance on queries?**

Check Indexes on columns.

There are many other things to optimize the database and query performance. But the easiest and straightforward method to improve the performance, is to check indexes on columns.

There are 4 main search screens, which large number of data.

- Contacts search – https://addressbook-web.efcorebeginner.com/contacts
- Countries search – https://addressbook-web.efcorebeginner.com/countries
- State and City search

Only Countries search was working under 1 second, as there are only 250 rows. But search queries on other tables were getting timeout. So I checked the search query in Repository classes and checked which column is used in sorting and searching. I found out that all searches were sorted default on Name column. Below is an example of Country Search query.

```cs
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

Even on default search, this sort method is called, which sorts the search query by Name column. If there are a few rows in the table, it does not matter. But if there are thousands of rows and sorting is done on every query, then it is a must to add an index on the sort column.

## How to Add Index with EF Core?

Just add the Index using data annotation on entity class, the EF Core will create index. The Country entity below has Index on Name column.

```cs
[Table("Country")]
[Index(nameof(Name))]
public class Country
{
    [Key]
    public int CountryId { get; set; }
    [Required]
    public string? Name { get; set; }
    // other columns
}
```

To apply the index, add the migration and update the database with the following commands.

```bat
add-migration country-index
update-database
```

## Performance after adding the Index

The performance increased a lot after adding indexes to all the sort columns. There were no timeouts. Most queries response time was 1-2 seconds.