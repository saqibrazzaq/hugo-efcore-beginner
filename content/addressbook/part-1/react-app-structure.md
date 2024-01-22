---
title: 7. Create React app, basic structure, API helper classes, DTOs
weight: 580
ShowToc: true
TocOpen: true
---

## Basic Structure – layout, header, footer, top menu

We have developed the API for managing city, state and country. Lets create the UI for it now, using React. To create the basic structure of the React app, read the following articles, which are already part of the beginner React UI series.

- [Create React app](/person-react/create-react-app/)
- [Add layout, header, footer, navigation menu using Chakra UI](/person-react/layout-menu/)
- [Setup Axios to call Persons web API](/person-react/setup-axios/)

## API Helper classes

When you are at the 3rd article, it says to create persionApi.js in src folder. Instead of creating personApi.js, create **countryApi.ts** in **src** folder. Previously we used JavaScript code for personApi, in this series, we will take advantage of TypeScript and write all APIs in TypeScript.

Add the following code.

```cs
import { CountryReqEdit, CountryReqSearch } from "../dtos/Country";
import { api } from "./axiosconfig"

export const CountryApi = {
  search: async function (searchParams: CountryReqSearch) {
    const response = await api.request({
      url: "/countries/search",
      method: "GET",
      params: searchParams,
    })

    return response.data
  },
  get: async function (countryId?: string) {
    if (!countryId) return {};
    const response = await api.request({
      url: `/countries/` + countryId,
      method: "GET",
    })

    return response.data
  },
  create: async function (country: CountryReqEdit) {
    const response = await api.request({
      url: `/countries`,
      method: "POST",
      data: country,
    })

    return response.data
  },
  update: async function (countryId?: string, country?: CountryReqEdit) {
    await api.request({
      url: `/countries/` + countryId,
      method: "PUT",
      data: country,
    })
  },
  delete: async function (countryId?: string) {
    const response = await api.request({
      url: `/countries/` + countryId,
      method: "DELETE",
    })

    return response.data
  },
}
```

In the TypeScript file, note that all functions have typed arguments e.g. search: async function (searchParams: CountryReqSearch). In JavaScript we could pass any object to the search function. If it were invalid or of wrong type, we would not be able to detect it. But in TypeScript, we have to pass strongly typed objects, the compiler will throw error in code if we pass the wrong object.

Add helper services for city, state and others following the same pattern. File names with links to GitHub are given below.

- [stateApi.ts](https://github.com/saqibrazzaq/efcorebeginner/blob/main/AddressBook/react-client/src/api/stateApi.ts)
- [cityApi.ts](https://github.com/saqibrazzaq/efcorebeginner/blob/main/AddressBook/react-client/src/api/cityApi.ts)
- [translationApi.ts](https://github.com/saqibrazzaq/efcorebeginner/blob/main/AddressBook/react-client/src/api/translationApi.ts)
- [timezoneApi.ts](https://github.com/saqibrazzaq/efcorebeginner/blob/main/AddressBook/react-client/src/api/TimezoneApi.ts)

## DTOs for city, state, country

Before we start to build the UI and call the API helper classes to do CRUD operations, we need to create the DTOs. The DTOs are used to send and receive data between API and the client.

Create a new folder in src, name it dtos. Add PagedReq.ts and PagedRes.ts files. These are generic DTOs for paging request and response. To get the details on these two dtos, refer to the article [Searching, Paging and Sorting in React](/person-react/searching-paging/). It was also explained in detail in the beginner React series, we won’t cover details here.

Now add a new file Country.ts in src\dtos folder. We created three dtos in our API (response, edit and search request), you can find source at https://github.com/saqibrazzaq/efcorebeginner/blob/main/AddressBook/AddressBook/Dtos/Country.cs. We will create the same DTOs in React, so that we send/receive the response using strongly typed variables. Add the following DTOs in Country.ts file.

```cs
export interface CountryRes {
  countryId?: string;
  name?: string;
  iso3?: string;
  iso2?: string;
  numericCode?: string;
  phoneCode?: string;
  capital?: string;
  currency?: string;
  currencyName?: string;
  currencySymbol?: string;
  tld?: string;
  native?: string;
  region?: string;
  subRegion?: string;
  latitude?: number;
  longitude?: number;
  emoji?: string;
  emojiU?: string;

  states?: StateRes[];
  translations?: TranslationRes[];
  timezones?: TimezoneRes[];
}

export class CountryReqEdit {
  name?: string = "";
  iso3?: string = "";
  // all other properties same as CountryRes
}

export class CountryReqSearch extends PagedReq {
  constructor(
    {
      pageNumber = 1,
      pageSize = Common.DEFAULT_PAGE_SIZE,
      orderBy = "",
      searchText = "",
    }: PagedReq,
    {}
  ) {
    super({
      pageNumber: pageNumber,
      pageSize: pageSize,
      orderBy: orderBy,
      searchText: searchText,
    });
  }
}
```

The CountryRes dto has three arrays for states, translations and timezones. In the beginner series, we had PersonRes dto, which just had 5 fields, no array. But now the Country entity has child tables with foreign keys, so we have arrays to load child records. We will do that in part two, not now. In part one, we are just creating multiple entities, multiple services, multiple dtos, UI pages, not handling related data. Part one is simple, we will work with the related data in part two and make things more real.

Country is parent table, it may contain multiple states. State may contain multiple cities. Lets create State Dtos now. Create a new file **State.ts** in **src\dtos** folder. Add the three dtos as below.

```cs
export interface StateRes {
  stateId?: string;
  name?: string;
  code?: string;
  latitude?: number;
  longitude?: number;

  countryId?: string;
  country?: CountryRes;

  cities?: CityRes[];
}

export class StateReqEdit {
  name?: string = "";
  code?: string = "";
  latitude?: number = 0;
  longitude?: number = 0;

  countryId?: string= "";
}

export class CountryReqSearch extends PagedReq {
  countryId?: string;
  constructor(
    {
      pageNumber = 1,
      pageSize = Common.DEFAULT_PAGE_SIZE,
      orderBy = "",
      searchText = "",
    }: PagedReq,
    {countryId = ""}
  ) {
    super({
      pageNumber: pageNumber,
      pageSize: pageSize,
      orderBy: orderBy,
      searchText: searchText,
    });
    this.countryId = countryId;
  }
}
```

ets talk how we have define relationship in the State dto. A state may have multiple cities, so it has an array of CityRes. A state belongs to a country, so it also has Country as its member. In state search dto, we also have countryId as member. Also note the constructor which has two portions in curly braces, in first part we have default initializers for PagedReq which is parent class, in second part we have initialized countryId which is specific to CountryReqSearch.

### Why primary key are of type string instead of number?

In React frontend app, we have set the type of primary keys, Ids as string. Previously we have used number, which makes sense, because in ASP.NET backend entity, dto etc. we have all Id, primary keys of type int. So why use string type in React?

We are dealing with three main languages/technologies here.

1. ASP.NET Core Web API – We use int for all primary keys, foreign keys
2. React – We use string for all primary keys, foreign keys
3. Json – When we call we API from React, the data is sent from the web API to the React app. The API converts dto into Json. Through the network, the data travels in the form of Json. The React app reads data in Json format, then converts Json to Dto.

In ASP.NET, if we have optional foreign key in the entity, we declare it as

```react
public int? StateId { get; set; }
```

And in React, we declare the same nullable number type as

```react
stateId?: number;
```

If we pass a null integer from ASP.NET to React, it will be converted to null in Json. When it Reaches React Typescript dto, it will be converted to null. We have few problems with this null in React.

- In React nullable number has default value of undefined, instead of null. When we receive null from ASP.NET, its equivalent in TypeScript is undefined. So either we convert all null to undefined, or in conditions we handle both null and undefined.
- We pass Ids in url params e.g. /states?countryId=4. If no country is selected, what will be our URL? It can be just /states. But we use object for search, which has countryId as member. It will be passed to the API whether it has a value or not. We can’t just skip it. It is nullable, so default value will be undefined, it will be converted to Url as /states?countryId=undefined. This is not good, because in Json, it will be string undefined, in ASP.NET it will be string undefined. We have to convert undefined to null.
- We also set countryId in form input textbox. If ASP.NET sends null, it is converted to null in Json, React receives as null or undefined, Formik will start to give warnings that control value is changing from undefined to defined or vice versa. We have to handle null number and use empty string.

To solve the issues of undefined, null or empty string, it would be just easier if we keep the IDs, primary keys, foreign keys as strings in React. Handle in API side, if we receive null integer from ASP.NET, in React we set it as empty string. If we receive a valid integer from ASP.NET, it will be converted to string e.g. 8 to “8”.

### Would it affect performance or bad practice to use string for Ids?

If we are using Forms, input textbox or dropdown, they will already convert our int to string.

If we are using Url params or Url query strings e.g. /states/8 or /states?countryId=2. In Url, these int will be converted to string. Whether we use number type in TypeScript, it will be converted to string. So why not we keep it in string format, so that we can handle forms and url params easy way.

For other data that will not be used in dropdown, url params, we will keep in number type. Non nullable data, which will always be a number, not null, not unspecified, but always a number, we will keep it number type, as we have no reason to keep as string.

Search dtos depend on Common.DEFAULT_PAGE_SIZE, so create a new file src\utility\Common.ts.

```react
export default class Common {
  static readonly DEFAULT_PAGE_SIZE = 5;
}
```

The source code of all DTOs is available on GitHub at https://github.com/saqibrazzaq/efcorebeginner/tree/main/AddressBook/AddressBook/Dtos.