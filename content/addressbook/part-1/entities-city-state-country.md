---
title: 2. Create Entities for City, State and Country
weight: 530
ShowToc: true
TocOpen: true
---

Start with creating entity classes for city, state and zip. We will use the data from the Json file hosted on https://github.com/dr5hn/countries-states-cities-database repository. The schema will also be based on the same Json file.

Below is an extract from the Json file for Qatar, where FIFA 2022 world cup took place.

```json
{
  "id": 179,
  "name": "Qatar",
  "phone_code": "974",
  "capital": "Doha",
  "currency": "QAR",
  "latitude": "25.50000000",
  "longitude": "51.25000000"
  "timezones": [
      {
          "zoneName": "Asia\/Qatar",
          "gmtOffset": 10800,
          "gmtOffsetName": "UTC+03:00",
          "abbreviation": "AST",
          "tzName": "Arabia Standard Time"
      }
  ],
  "translations": {
      "kr": "카타르",
      "pt-BR": "Catar"
  },
  "states": [
      {
          "id": 3183,
          "name": "Al Khor",
          "state_code": "KH",
          "latitude": "25.68040780",
          "longitude": "51.49685020",
          "type": null,
          "cities": [
              {
                  "id": 89856,
                  "name": "Al Ghuwayrīyah",
                  "latitude": "25.82882000",
                  "longitude": "51.24567000"
              },
              {
                  "id": 89858,
                  "name": "Al Khawr",
                  "latitude": "25.68389000",
                  "longitude": "51.50583000"
              }
          ]
      }
  ]
}
```

Country may have multiple states, translations and time zones. A state may have multiple cities.

We can create the following entity classes using the above design.

```cs
[Table("Country")]
public class Country
{
  [Key]
  public int CountryId { get; set; }
  [Required]
  public string? Name { get; set; }
  // Other members.....
  
  // Child tables
  public ICollection<Timezone>? Timezones { get; set; }
  public ICollection<Translation>? Translations { get; set; }
  public ICollection<State>? States { get; set; }
}

[Table("Translation")]
public class Translation
{
  [Key]
  public int TranslationId { get; set; }
  [Required]
  public string? Code { get; set; }
  // Other members...
  
  // Foreign keys
  public int? CountryId { get; set; }
  [ForeignKey(nameof(CountryId))]
  public Country? Country { get; set; }
}

[Table("Timezone")]
public class Timezone
{
  [Key]
  public int TimezoneId { get; set; }
  [Required]
  public string? Name { get; set; }
  // Other members...

  // Foreign keys
  public int? CountryId { get; set; }
  [ForeignKey(nameof(CountryId))]
  public Country? Country { get; set; }
}

[Table("State")]
public class State
{
  [Key]
  public int StateId { get; set; }
  [Required]
  public string? Name { get; set; }
  // Other members...

  // Foreign keys
  public int? CountryId { get; set; }
  [ForeignKey(nameof(CountryId))]
  public Country? Country { get; set; }

  // Child tables
  public ICollection<City>? Cities { get; set; }
}

[Table("City")]
public class City
{
  [Key]
  public int CityId { get; set; }
  [Required]
  public string? Name { get; set; }
  // Other members...
  
  // Foreign keys
  public int? StateId { get; set; }
  [ForeignKey(nameof(StateId))]
  public State? State { get; set; }
}
```

The tables listed above have some fields omitted here, otherwise the page will grow too long. Only the important fields like Id, Name, foreign keys and collections for the related tables are given.