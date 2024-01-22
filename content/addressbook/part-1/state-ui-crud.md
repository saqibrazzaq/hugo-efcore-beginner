---
title: 9. UI for States CRUD with search dropdown and Url search params
weight: 600
ShowToc: true
TocOpen: true
---

We have created CRUD pages for Country. They were very simple, just do CRUD on single table, no relationship, all read, write, update, delete on a single table. In this tutorial, we will create UI for managing states. We will learn some new things today.

## Search dropdown for Country

In Country search page, Countries.tsx, we had text box for searching countries, which was enough for countries page. What else we need to search a country? In state search page, we will definitely need search text box. But we also need a dropdown for country. For example, we may want to filter the states for a specific country.

Create a new folder **dropdowns** in src. Add a new file **CountryDropdown.tsx** and type rafce to insert the template code. The function will have props of type CountryDropdownParams with the following members

- handleChange – method to call when dropdown value is updated
- selectedCountry – Object of type CountryRes, if we want a country to be selected in the dropdown

```react
interface CountryDropdownParams {
  handleChange?: any;
  selectedCountry?: CountryRes;
}

const CountryDropdown = ({handleChange, selectedCountry}: CountryDropdownParams) => {
}
```

We will use [chakra-react-select](https://www.npmjs.com/package/chakra-react-select) package for dropdown, which is based on the [react-select](https://www.npmjs.com/package/react-select) package. Usage of both packages is exactly same. Internally chakra-react-select packages updates the styles of dropdown according to the Chakra UI theme.

Now declare three state variables

- inputValue – text you type for searching in the dropdown list
- items – option items for the dropdown
- isLoading – true when loading, false when loading is complete

```react
const [inputValue, setInputValue] = useState("");
const [items, setItems] = useState<CountryRes[]>([]);
const [isLoading, setIsLoading] = useState(false);
```

Before we add the Select dropdown from chakra-react-select, lets add code for Select dropdown’s onInputChange event. This event if fired whenever user types in the textbox. Whenever user types, we store it in the state variable inputValue. And we will create a useEffect hook, which will fire when inputValue changes. This useEffect hook will search the countries from web API and set the items for the dropdown. The useEffect uses timer, it waits for 1 second after the input value is changed. After one second, it calls Country search web API and clears the timer.

```react
const loadCountries = () => {
  setIsLoading(true);
  CountryApi.search(new CountryReqSearch({ searchText: inputValue }, {}))
    .then((res) => {
      setItems(res.pagedList);
    })
    .finally(() => setIsLoading(false));
};

useEffect(() => {
  const timer = setTimeout(() => {
    loadCountries();
  }, 1000);

  return () => clearTimeout(timer);
}, [inputValue]);

const handleInputChange = (newValue: string) => {
  setInputValue(newValue);
};
```

Seems complex? It might seem a bit complex at start, but once you learn how to create select dropdown for one entity, it is very easy to create search select dropdown for any other entity or dto.

The final piece of the code in the country dropdown is the actual Select component of Chakra react select library. It is really simple to use. Below are the settings we use, with explanation.

- getOptionLabel – label, the value that user sees for an option item, we set country name as label
- getOptionValue – backend value of the option item, we set countryId as value
- options – array of items which are populated in the select dropdown
- onChange – event when the select dropdown value is changed, means when we search and choose an option
- value – value of the Select dropdown. It is different than getOptionValue. getOptionValue is the value of an item inside the Select dropdown.

```react
<Select
  getOptionLabel={(c) => c.name || ""}
  getOptionValue={(c) => c.countryId || ""}
  options={items}
  onChange={handleChange}
  onInputChange={handleInputChange}
  isClearable={true}
  placeholder="Select country..."
  isLoading={isLoading}
  value={selectedCountry}
></Select>
```

The complete code for the CountryDropdown.tsx is at [GitHub](https://github.com/saqibrazzaq/efcorebeginner/blob/main/AddressBook/react-client/src/dropdowns/CountryDropdown.tsx).

## Searching, paging, sorting from URL search params

First lets discuss why we should implement searching, paging, sorting from url search params?

Run the app and open the Country page from menu. Add around 10 countries, so that you have 2 pages, our default page size is 5. After adding around 10 countries, navigate to next and previous page. Search something. Click on Edit, it will open the Edit page. Now click on Back button in app or browser (same thing). It will open the default countries at http://localhost:3000/countries again!! Did you see the problem?

Suppose you have a full list of 300 countries, you navigate on 7th page, you want to edit a country, then go back. Ideally you should go back to the 7th page. But it will open the default 1st page. What if there are a lot of filters like dropdowns and searchboxes. You add filter and reach on your desired records. You want to edit or view detail of one, then go back. It will again open the default first page. You have to apply filters again.

For good user experience, the filters should be part of the URL e.g. http://localhost:3000/countries?pageNumber=7. When you navigate to page 3, the url should be http://localhost:3000/countries?pageNumber=3 and app should go to page 3. When you move to another page e.g. http://localhost:3000/countries/edit/7 and then go back, you will come again to http://localhost:3000/countries?pageNumber=3, the same position where you left off!!

It will be very useful in case of state search, where we can filter using search text, country id and page number.

Open src\state\States.tsx in Visual Studio Code. Add the following code for initializing search params. React router dom has useLocation Hook to get the url. We get search params with useSearchParams hook and store in searchParams variable. Then we set pageSize in search param to default page size. The below code will set the url to localhost:3000/states?pageSize=5.

```react
const location = useLocation();
const [searchParams, setSearchParams] = useSearchParams(location.search);
searchParams.set("pageSize", Common.DEFAULT_PAGE_SIZE.toString());
```

After that we will initialize state variables for paged list and search text, just like we did in country search page. Here we will add one more state variable to store the selected country. searchText is initialized with searchText param variable from Url. For example if Url is localhost:3000/states?searchText=ca, the searchText state variable will be initialized with the value ca. selectedCountry is initialized with empty object.

```react
const [pagedRes, setPagedRes] = useState<PagedRes<StateRes>>();
const [searchText, setSearchText] = useState(searchParams.get("searchText") || "");
const [selectedCountry, setSelectedCountry] = useState<CountryRes>({});
```

### Initialize Url search params on page load

After that, add a useEffect hook which will execute whenever there is update in searchParams. That means this useEffect hook will execute when there will be change in Url params. It will be executed when page is loaded with localhost:3000/states url. If url is changed to localhost:3000/states?pageNumber=2, this hook will execute. If url changes to localhost:3000/states?countryId=4, again this hook will execute. So our search method will take parameters from Url search params and call the web API.

```react
useEffect(() => {
  loadCountry();
  searchStates();
}, [searchParams]);

const loadCountry = () => {
  let countryId = searchParams.get("countryId") || undefined;
  CountryApi.get(countryId).then(res => setSelectedCountry(res))
}

const searchStates = () => {
  if (!searchParams) return;
  console.log(Object.fromEntries(searchParams))
  StateApi.search(Object.fromEntries(searchParams)).then(res => {
    setPagedRes(res);
    console.log(res)
  }).catch(error => {
    console.log(error)
  })
}
```

### Update search params on button click

In the above code snippet, we added useEffect, which takes the latest search params from Url, then calls the web API. When page is loaded first time, searchParams will automatically be initialized with the url search params. What if we type something in search and call web API. How do we handle dropdown change event, when country is changed from dropdown, how to call web API again with new Id? How to call web API again when we click on next, previous page buttons?

We will add a generic method, which will update Url search params. This method will be called when next, previous page button is clicked. The same method will be called when we select countryId or type something in the search box. This method accepts key and value. It has just two lines of code. First we call searchParams.set method with key/value. Then we call setSearchParams from react-router-dom. This way our useEffect hook will be executed again, with latest values from url search params.

```react
const updateSearchParams = (key: string, value: string) => {
  searchParams.set(key, value);
  setSearchParams(searchParams)
}
```

Lets see how our next and previous page methods will look like. As you can see below, these are very simple, we are just calling updateSearchParams with key/value. They key is the variable name in the ASP.NET search DTO. Value is the variable value. previousPage method will decrement page number and set Url with new page number e.g. localhost:3000/states?pageNumber=1. nextPage will increment and update url with new value e.g. localhost:3000/states?pageNumber=2. Url will be updated. useEffect hook will also be executed, which will call ASP.NET web API for search.

```react
const previousPage = () => {
  if (pagedRes?.metaData) {
    let previousPageNumber = (pagedRes?.metaData?.currentPage || 2) - 1;
    updateSearchParams("pageNumber", previousPageNumber.toString());
  }
};

const nextPage = () => {
  if (pagedRes?.metaData) {
    let nextPageNumber = (pagedRes?.metaData?.currentPage || 0) + 1;
    updateSearchParams("pageNumber", nextPageNumber.toString())
  }
};
```

Lets write the code for displaying search bar, as we did in Countries.tsx. In Country search page, we only had a search text box and search button. In State search, we will have an additional dropdown for Countries. We already created the Country dropdown, here we will use it. The dropdown requires two props, selectedCountry and handleChange. In handleChange event, we get the countryId of the selected option and update URL params. The search web API will be called again due to update in Url params and page will refresh with updated results.

Then we have search textbox and search button, the implementation is same as Country search page. Just one update here, when we click on search button on press enter in search textbox, we update Url params. The update in Url params will automatically execute the search web API because of useEffect hook we used earlier.

```react
const displaySearchBar = () => (
  <Flex>
    <Center>
      <Text>Select country:</Text>
    </Center>
    <Box flex={1} ml={4}>
      <CountryDropdown
        selectedCountry={selectedCountry}
        handleChange={(newValue?: CountryRes) => {
          updateSearchParams(
            "countryId",
            newValue ? newValue?.countryId + "" : ""
          );
        }}
      />
    </Box>

    <Box ml={4}>
      <Input
        placeholder="Search..."
        value={searchText}
        onChange={(e) => setSearchText(e.currentTarget.value || "")}
        onKeyDown={(e) => {
          if (e.key === "Enter") {
            updateSearchParams("searchText", searchText);
          }
        }}
      />
    </Box>
    <Box ml={0}>
      <Button
        colorScheme={"blue"}
        onClick={() => {
          updateSearchParams("searchText", searchText);
        }}
      >
        Search
      </Button>
    </Box>
  </Flex>
);
```

The rest of the code needs no explanation, as there is nothing new. You can get the complete code at GitHub. Complete the code for States.tsx and run the app with npm start. Go to localhost:3000 and search, filter states, you will note that Url search parameters are changing, the results are also refreshed.

See the gif below, we created some countries and states. We are filtering states with country id and page navigation, the Url is updating, which is causing the search results to refresh. After filtering, when we click on Edit or Delete page, then press the browser back button, we go back to the exact Url where we left off. Our search filter is kept. We don’t have to search all over again.

![states navigation url params](/images/states-navigation-url-params-1024x754.gif "states navigation url params")

## Workflow summary

When we select country from dropdown, it updates the Url search params. When we press enter on search textbox, it updates Url params. When we click on search button, it updates the Url params. When we click on previous or next page, it will again update the Url params.

![states workflow summary](/images/states-workflow-summary-1024x466.jpg "states workflow summary")

The state search web API is called in useEffect hook. The useEffect hook is executed when searchParams value is updated. This is the only way we call the search web API, no other event calls the search API.

![states workflow search params](/images/states-workflow-search-param-724x1024.jpg "states workflow search params")

## Create and Edit State page

The create/edit page for state will be similar to the edit country page, with one difference. In country, we only had simple text fields in edit page. Now, we will have Country dropdown in the edit state page. Our database has relations, the state table has CountryId as foreign key. If we want to create a new state, it must have a country. So we need to input country id in edit state page. Of course we cannot input country id using a text input field. We will reuse the Country dropdown in edit state page.

Open src\state\StateEdit.tsx. Start by adding the variables for countryId and stateId. In country edit, we had the following Url formats

- localhost:3000/countries/edit – create a new country
- localhost:3000/countries/edit/{countryId} – edit an existing country

In state edit, we will use stateId and countryId in URL. Why both? We answer in description of url formats below

- localhost:3000/states/edit – create a new state, country dropdown shows default countries, no country is selected
- localhost:3000/states/edit/{countryId} – create a new state. But this time country dropdown will search and select the country
- localhost:3000/states/edit/{countryId}/{stateId} – edit an existing state. Search and select the country in dropdown

```react
const params = useParams();
const countryId = params.countryId;
const stateId = params.stateId;
const updateText = stateId ? "Update State" : "Add State";
```

To support the above URL scheme of create/edit state page, we also have to update State search page and update code for edit/create buttons. Open States.tsx and update the code for Add State button. We just added countryId search param from Url. If countryId is in Url, it will also be passed to state edit page.

```react
<Link ml={2} as={RouteLink} to={"/states/edit/" + (searchParams.get("countryId") ?? "")}>
  <Button colorScheme={"blue"}>Add State</Button>
</Link>
```

We also need to update the edit link in States.tsx, showStates() method as follows. This is very important, the edit state must include countryId, the Url format is http://localhost:3000/states/{countryId}/{stateId}.

```react
<Link mr={2} as={RouteLink} to={"/states/edit/" + item.countryId + "/" + item.stateId}>
  Edit
</Link>
```

Now back to edit state page, open StateEdit.tsx. Continue adding two state variables, one for state and other for selectedCountry. Initialize selectedCountry with empty. Initialize state with the countryId. Then we have useState hooks. When stateId is provided, it will load state. When countryId is provided, it will load the selected country. loadCountry and loadState methods will call API helper method to get country and state.

```react
const [selectedCountry, setSelectedCountry] = useState<CountryRes>();
const [state, setState] = useState<StateReqEdit>(new StateReqEdit(countryId));

useEffect(() => {
  loadState();
}, [stateId]);

useEffect(() => {
  loadCountry(countryId);
}, [countryId]);

useEffect(() => {
  loadCountry(state.countryId);
}, [state.countryId]);
```

Lets work on form validation now. The state validation is similar to the country validation, which we did before, except it has countryId, which is a foreign key. We set countryId as required field and set minimum value to 1. If country Id is selected from dropdown, it will have valid id, otherwise our form will show validation errors.

```react
// Formik validation schema
const validationSchema = Yup.object({
  name: Yup.string().required("State Name is required"),
  code: Yup.string(),
  latitude: Yup.number(),
  longitude: Yup.number(),
  countryId: Yup.number().required().min(1, "Please select country"),
});
```

We will not write about submitForm method here, as there is nothing new to learn in it. They just call API helper methods, which uses axios to send http POST and PUT requests with data to the web API.

Lets create the form now with Formik. In the form, we will have hidden input field for countryId. The value of this hidden field will be set by the Country dropdown. In dropdown’s change event, we set the country Id in the hidden field. setFieldValue(key, value) is the formik method to set the form field value. If country id is not selected, we set countryId to empty string, which will cause validation error. The validation is also set on the hidden field countryId.

```react
<form onSubmit={handleSubmit}>
  <Stack spacing={4} as={Container} maxW={"3xl"}>
    <FormControl isInvalid={!!errors.countryId && touched.countryId}>
      <FormLabel htmlFor="countryId">Country Id</FormLabel>
      <Field
        as={Input}
        id="countryId"
        name="countryId"
        type="hidden"
      />
      <CountryDropdown
        selectedCountry={selectedCountry}
        handleChange={(newValue?: CountryRes) => {
          setSelectedCountry(newValue);
          setFieldValue("countryId", newValue?.countryId || "");
        }}
      />
      <FormErrorMessage>{errors.countryId}</FormErrorMessage>
    </FormControl>
    // Other form controls using text input, same as before
  </Stack>
</form>
```

The complete code of StateEdit.tsx is at [GitHub](https://github.com/saqibrazzaq/efcorebeginner/blob/main/AddressBook/react-client/src/pages/state/StateEdit.tsx). You can either get the complete code from GitHub or complete the edit page yourself. Save the file and run the project with npm start. Create new state in a country or update an existing state, it should work.

![edit state](/images/edit-state.jpg "edit state")

## Delete State

Lets now work on the delete state page. Open src\state\StateDelete.tsx. We will not cover much detail, as it is almost similar to the Delete country page. You can easily refer to [delete country](https://github.com/saqibrazzaq/efcorebeginner/blob/main/AddressBook/react-client/src/pages/country/CountryDelete.tsx) and write code for [delete state](https://github.com/saqibrazzaq/efcorebeginner/blob/main/AddressBook/react-client/src/pages/state/StateDelete.tsx) yourself. We will only write here about what is new in the delete state. It looks like there are few things.

In the delete page, we first load the state, so that we can show the user what he is going to delete. In delete country, the load country method just loads the data from the same table. But in state, we have countryId as foreign key. The country data exists in the Country table. So we need to fetch data from two tables.

Lets have a look at our DTOs again. In ASP.NET web API project, open dtos\State.cs and check the StateRes dto. CountryId is foreign key, when we get this record in repository, the countryId will be loaded by default, as it exists in the same State table. But Country property will not be loaded by default. Similarly ICollection Cities will also be empty.

```react
public class StateRes
{
  public int StateId { get; set; }
  public string? Name { get; set; }
  public string? Code { get; set; }
  public double Latitude { get; set; }
  public double Longitude { get; set; }

  // Foreign keys
  public int? CountryId { get; set; }
  public Country? Country { get; set; }

  // Child tables
  public ICollection<City>? Cities { get; set; }
}
```

### Load Country data in State using EF Core Include?

Open Services\StateService.cs and view the FindStateIfExists() method. The repository has generic method FindByCondition, it matches state id and returns the State entity. To load the State -> Country, we need to include Country in EF Core, like below.

```react
private State FindStateIfExists(int stateId, bool trackChanges)
{
  var entity = _repositoryManager.StateRepository.FindByCondition(
    x => x.StateId == stateId,
    trackChanges,
    include: i => i.Include(x => x.Country))
    .FirstOrDefault();
  if (entity == null) { throw new Exception("No state found with id " + stateId); }

  return entity;
}
```

Still our program will not run correctly. Try running it, you will get **cyclic redundancy error** in json response, because State contains Country, Country again contains States, State contains Country and so on…. To solve this, open Program.cs and update the AddControllers() line as follows. We ask the controllers to ignore Json reference cycles.

```react
builder.Services.AddControllers(config =>
{
    config.RespectBrowserAcceptHeader = true;
    config.ReturnHttpNotAcceptable = true;
}).AddJsonOptions(x =>
{
    x.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter());
    x.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;
    x.JsonSerializerOptions.WriteIndented = true;
});
```

Now stop the web API project and run it again. Open the default Swagger UI, go to the States section and click on /api/States/{stateId}. Enter a valid state id and check the result, it should now contain country data as well.

![state contains country](/images/state-contains-country-1024x544.jpg "state contains country")

Lets come to the React project now. Open the src\dtos\State.ts and view StateRes dto. It contains CountryRes. Now that our web API includes Country data in state, it should also be loaded in React dto.

```react
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
```

Open src\states\StateDelete.tsx. Complete the code yourself, same as CountryDelete.tsx or refer to the complete code on [GitHub](https://github.com/saqibrazzaq/efcorebeginner/blob/main/AddressBook/react-client/src/pages/state/StateDelete.tsx). There is only one new thing in delete state code, we are displaying state data, as well as the data from related table Country. state.country.name displays the country name of the loaded state.

```react
<Tr>
  <Th>Country</Th>
  <Td>{state?.country?.name}</Td>
</Tr>
```

We will also include city count here and show it in the table. If there are any cities in a state, we will display an error message and **disable** the **DELETE** button. State has one to many relationship with City table. Deleting a State can delete all its cities, so we will prevent that, because it will result in data loss. We did this in detail in the [previous article](/addressbook/part-1/country-ui-crud/).

Run the project with npm start, delete a state, it will now show Country information, because we have used Include in Entity Framework repository, which gets data from related tables.

![delete state which has cities](/images/delete-state-which-has-cities.jpg "delete state which has cities")