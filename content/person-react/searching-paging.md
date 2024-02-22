---
title: 8. Searching, paging and sorting in React
weight: 290
ShowToc: true
TocOpen: true
---

In the previous articles, you learned how to write a simple React app that implement UI for CRUD operations, using web API. At this stage, the List persons page call getAll web API, which returns all the records from the database. What if we have 100 or more records in our database. We have to implement searching and paging, this is also basic and required feature in any app.

We added searching, paging and sorting in our web API. Lets update the List persons page with search API instead of getAll API.

Before you start implementing paging, I would recommend to add at least 20 persons using the React app. So that we have some data to test next, forward links.

## PagedReq – Common DTO for Search request

Run the web API project and open the default Swagger UI. Get All method does not require any parameter, we do not require any DTO for get all

![web api get all no parameters](/images/blog/web-api-get-all-no-parameters-1024x181.jpg "web api get all no parameters")

Now check the search method that we implemented. It takes parameters so that client can pass search string, page number to fetch and other search criteria.

![web api search with parameters](/images/blog/web-api-search-with-parameters-1024x525.jpg "web api search with parameters")

It has total 5 parameters. 4 out of 5 parameters are common. We created a common PagedReq DTO for this in our web API. Then created Person search request dto which inherits the common class.

We will implement the same in React. Lets create the **PagedReq.ts** file first in **dtos** folder.

```react
export class PagedReq {
  pageNumber?: number = 1;
  pageSize?: number = Common.DEFAULT_PAGE_SIZE;
  orderBy?: string = "";
  searchText?: string = "";

  constructor({
    pageNumber = 1,
    pageSize = Common.DEFAULT_PAGE_SIZE,
    orderBy,
    searchText = "",
  }: PagedReq) {
    this.pageNumber = pageNumber;
    this.pageSize = pageSize;
    this.orderBy = orderBy;
    this.searchText = searchText;
  }
}
```

The constructor initializes page number with 1, by default we will get the first page from the API. Page size is set to 5, which is defined in Common file. Create **Common.ts** file in **src\utility** folder.

```react
export default class Common {
  static readonly DEFAULT_PAGE_SIZE = 5;
}
```

## Person search request DTO

Now that we have defined the base class for the Common DTO parameters. Lets create search dto for the Person now. Open src\dtos\Person.ts file and add new search dto as follows.

```react
export class PersonReqSearch extends PagedReq {
  gender?: string;
  constructor(
    {
      pageNumber = 1,
      pageSize = Common.DEFAULT_PAGE_SIZE,
      orderBy,
      searchText = "",
    }: PagedReq,
    { gender = "" }
  ) {
    super({
      pageNumber: pageNumber,
      pageSize: pageSize,
      orderBy: orderBy,
      searchText: searchText,
    });
    this.gender = gender;
  }
}
```

**PersonReqSearch** class extends from the base class **PagedReq**. Note the constructors of PersonReqSearch class. It consists of two sets of curly braces.

1. In the first set, it has parameters for the base class PagedReq.
2. In the second set, it has initialization parameters for itself.

In this case, only gender belongs to Person, rest of the parameters are for the base class.

## Generic Search Response Dto

We have created Dto for search request. Now we will create Dto for search response.

Search response will be different from Get all response. GetAll persons web API returned all persons as array PersonRes[]. Search response will contain a paged list and metadata about the list. The search query might return thousands of records, but we will access these records by page. So metadata contains information like which page number it returned, total pages, page size, count etc. Add a new file named **PagedRes.ts** in **src\dtos** folder.

```react
interface MetaData {
  currentPage?: number;
  totalPages?: number;
  pageSize?: number;
  totalCount?: number;
  hasNext?: boolean;
  hasPrevious?: boolean;
}

export default interface PagedRes<T> {
  pagedList?: T[];
  metaData?: MetaData;
}
```

The metada has information to display the total records and generate next and previous links, so that it can navigate to the correct page number.

PagedRes<T> is generic class which can be set to any type. pageList: T[] is generic array and can be initialize with any type e.g. PagedRes<PersonRes> and pageList: PersonRes[]. We can use this generic class for any dto.

## Search API helper method

Open src\api\personApi.js and add the new search method as follows.

```react
search: async function (searchParams) {
  const response = await api.request({
    url: "/persons/search",
    method: "GET",
    params: searchParams,
  })

  return response.data
},
```

It receives person request search as parameter and calls http GET method of our web API. The search request dto is sent to the API as GET request parameters in URL.

## Update List Persons page, add search and paging support

Lets work on the UI now. Open **src\persons\Persons.tsx** file. We will update this page and replace get all persons with search persons.

### State variable for Search Request parameters

Start by deleting persons state variable. Add a new state variable named pagedRes as follows.

```react
const [pagedRes, setPagedRes] = useState<PagedRes<PersonRes>>();
```

We are creating a state variable of type PagedRes<PersonRes>. The web API returns search response of this type in Json. We know in advance so we assign API response result to state variable. The strong typing of TypeScript is really helpful here. It is very easy to do mistakes in accessing complex objects when received from a web API.

We will add two more state variables for search text and search request params.

```react
const [searchText, setSearchText] = useState<string>("");
const [searchReq, setSearchReq] = useState<PersonReqSearch>(
  new PersonReqSearch({}, {})
);
```

searchText will be bound to the search textbox.

searchReq state variable will keep Person request search dto. Why we need to keep it in state variable? This dto contains parameters for page number, page size, sort and search criteria. We send this dto to the web api everytime we do

- type in search textbox and press enter
- go to previous page
- go to next page
- update in sort column
- update in any other search criteria etc.

Whenever any of the above is changed, we will just update the search request.

### useEffect hook for calling search web API

Delete the existing **useEffect** hook and **loadAllPersons** method. Add the following useEffect hook and new search method.

```react
useEffect(() => {
  searchPersons();
}, [searchReq]);

const searchPersons = () => {
  PersonApi.search(searchReq).then((res) => {
    setPagedRes(res);
  });
};
```

Previously we called get all web API on page load using [] in useEffect hook. Now we have called search persons API in useEffect with [searchReq]. This way the search web API will be called every time the search request is updated. If any member is updated like page number, search text, sort column, the web search API will be called. And if search is successful, we update the paged response with the new search results.

### Search Textbox

Now we will add a textbox for search, after the heading. Add a new method displaySearchBar as follows.

```react
const displaySearchBar = () => (
  <Flex>
    <Center></Center>
    <Box flex={1} ml={4}></Box>

    <Box ml={4}>
      <Input
        placeholder="Search..."
        value={searchText}
        onChange={(e) => setSearchText(e.currentTarget.value)}
        onKeyDown={(e) => {
          if (e.key === "Enter") {
            setSearchReq({
              ...searchReq,
              ...{ searchText: searchText },
            });
          }
        }}
      />
    </Box>
    <Box ml={0}>
      <Button
        colorScheme={"blue"}
        onClick={() => {
          setSearchReq({ ...searchReq, ...{ searchText: searchText } });
        }}
      >
        Search
      </Button>
    </Box>
  </Flex>
);
```

We have added Chakra UI’s Input component for input textbox and a Search button. The textbox onChange event updates the searchText state variable. Whenever we type something in the search textbox, it will update the state variable. On Enter key, we update the search request parameters with the new search text value. We did not call search web API here. We only updated the search request state variable, which will automatically call web search API due to useEffect hook.

The search request state variable is also updated on Search button click, which will result into calling the search API again, due to the useEffect hook.

Update the main function’s return statement and call this displaySearchBar method as follows.

```react
return (
  <Box width={"100%"} p={4}>
    <Stack spacing={4} as={Container} maxW={"3xl"}>
      {showHeading()}
      {displaySearchBar()}
      {showPersons()}
    </Stack>
  </Box>
);
```

### Replace person array with pagedRes<PersonRes> in the table

At this stage, you should have one error in the showPersons method. In table body, we mapped persons array to display a row for each person. We will now use pagedRes instead of the person array as follows.

```react
pagedRes?.pagedList?.map
```

Save the file and see the updates in the Person List page at http://localhost:3000/persons. It should display just 5 records now, because we have set default page size to 5. It should also display the search textbox after the heading as shown in the screenshot below.

![replace get all with search](/images/blog/replace-get-all-with-search.jpg "replace get all with search")

Type something in search textbox, press Enter or search button, the search should work.

![search textbox](/images/blog/search-textbox.jpg "search textbox")

### Add Previous and Next page navigation links

Lets add page navigation links in the table. Before adding the next, previous links, we will define methods for next and previous. Add the two methods as follows.

```react
const previousPage = () => {
  if (pagedRes?.metaData) {
    let previousPageNumber = (pagedRes?.metaData?.currentPage || 2) - 1;
    setSearchReq({
      ...searchReq,
      ...{ pageNumber: previousPageNumber },
    });
  }
};

const nextPage = () => {
  if (pagedRes?.metaData) {
    let nextPageNumber = (pagedRes?.metaData?.currentPage || 0) + 1;
    setSearchReq({ ...searchReq, ...{ pageNumber: nextPageNumber } });
  }
};
```

previousPage method checks the current page from metadata in search response dto. If current page is valid, it will decrement the current page by 1 and just update the search request state variable. The useEffect hook will again call the web API automatically.

Similarly the nextPage method checks the current page from metadata. If current page is valid, it will increment it by 1 and update the search request state variable. Whenever the search request state variable is update, the search web API will be called again due to the useEffect hook.

After defining the methods for next and previous, lets add the links in the table. The Table currently consists of Thead and Tbody. Thead displays the headings and Tbody displays the persons, each person per row. In this case it is suitable to add page navigation links in the table footer, Tfoot. In showPersons, add Tfoot just after the Tbody as follows.

```react
<Tfoot>
  <Tr>
    <Th colSpan={2} textAlign="center">
      <Button
        isDisabled={!pagedRes?.metaData?.hasPrevious}
        variant="link"
        mr={5}
        onClick={previousPage}
      >
        Previous
      </Button>
      Page {pagedRes?.metaData?.currentPage} of{" "}
      {pagedRes?.metaData?.totalPages}
      <Button
        isDisabled={!pagedRes?.metaData?.hasNext}
        variant="link"
        ml={5}
        onClick={nextPage}
      >
        Next
      </Button>
    </Th>
  </Tr>
</Tfoot>
```

Button component is used to display both the Next and Previous links. Previous button’s onClick event calls the previousPage method, similarly we used nextPage in Next button’s onClick event. The metadata has hasNext and hasPrevious boolean properties, which is used to disable previous and next links. Current page and total page is also displayed in Page x of y format, these values are available in metadata.

Save the file and check updates in the person list page. We have now implemented searching and paging in this React component. Try next and previous links and use search, it should work now.

![searching and paging with persons](/images/blog/searching-and-paging-in-persons.jpg "searching and paging with persons")

If it does not work and shows error, please check the source file at https://github.com/saqibrazzaq/efcorebeginner/blob/main/Person/react-client/src/persons/Persons.tsx.