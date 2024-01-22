---
title: 4. Person Response DTO, call API from React component and fix CORS
weight: 250
ShowToc: true
TocOpen: true
---

This is the 4th article in the React client app series for consuming the Person Web API. In the previous articles we created a new React app and setup the layout, top menu and routes for the home page. We also setup Axios http client for calling our web API. In this article we will create a new page with UI to get all persons from our API.

## Create Person Response Dto

In the web api, we created simple CRUD app using ASP.NET Core and Entity Framework. Now we are building the UI for that API. We are using TypeScript and will use strongly typed objects, so we will define the Dtos for listing, creating and updating the Person. These Dtos only contain the data that is required by the client. These Dtos are not entity classes. We have not exposed entity classes in the API. The entity classes are private to the API and represent the database schema. The Dtos on the other hand only contain data that is required by the client.

Create a new folder **dtos** in **src**. In dtos folder, add a new file **Person.ts**. Open the web API project as well in Visual Studio 2022, that we created previously. We need it to refer to the Person Dtos we created there. Below was the PersonRes Dto that the API returned as response.

```javascript
public class PersonRes
{
  public int PersonId { get; set; }
  public string? FirstName { get; set; }
  public string? LastName { get; set; }
  public string PhoneNumber { get; set; } = "";
  public string Gender { get; set; } = "";
}
```

Run the web API project and call GET method /api/Persons/{personId} e.g. url https://localhost:7277/api/Persons/1 It returns the following Json.

```javascript
{
  "personId": 1,
  "firstName": "Iron",
  "lastName": "Man",
  "phoneNumber": "1112223333",
  "gender": "m"
}
```

Note the variable names in PersonRes class and the Json response. Json uses camel case, first letter is always small. The variable names in our React app must be same as the variable names in the Json response, only then we will be able to get Json converted into TypeScript objects. Once we get our Dtos right, we can refer to them using Visual Studio intellisense e.g. person.firstName, person.phoneNumber etc. If we do a mistake, the compiler will tell us. Without TypeScript, we will have to depend on correctly using the variable names every time we get the response from the API.

In src\dtos\Person.ts file, add a new interface for **PersonRes** as follows.

```javascript
export interface PersonRes {
  personId?: number;
  firstName?: string;
  lastName?: string;
  phoneNumber?: string;
  gender?: string;
}
```

## Create the List Persons page

Now it is time to create the page to list all Persons. So far we created the following folders in the **src**.

- api
- dtos
- layout
- home

Create a new folder **persons** in **src**, which will be our 5th folder. We will keep person list, create, update and delete person files here. In src\persons folder, create a new file **Persons.tsx**. Type rafce and insert the template code.

Open App.tsx, where we have defined routes. We will add a new route for Person list. The new routing should be like below.

```xml
<BrowserRouter>
  <Routes>
    <Route path="/" element={<Layout />}>
      <Route index element={<Home />} />
      {/* Persons */}
      <Route path="persons">
        <Route index element={<Persons />} />
      </Route>
    </Route>
  </Routes>
</BrowserRouter>
```

According to this route, / renders Layout and then Home page.

And /persons should render Layout. In the Outlet part of Layout, /persons will also render Persons.tsx component. Because we have defined Persons route as child of Layout.

Run the project with npm start command. We already created top navigation menu and added /persons URL in the menu item. Click on it and you should see Persons component is rendered now.

![persons component](/images/persons-component.jpg "person component")

## Call Web API with Axios

The routing is set, the Persons list page (Persons.tsx) displays the default content. Now open Persons.tsx again, we will update it to call our API to get all Persons.

Just for testing, we will first call the API directly in Persons page. Update the Persons.tsx as follows

```javascript
import { PersonApi } from "../api/personApi"

const Persons = () => {
  
  console.log("base url: " + process.env.REACT_APP_API_BASE_URL)
  PersonApi.getAll().then(res => {
    console.log(res);
  })

  return (
    <div>Persons</div>
  )
}

export default Persons
```

We have imported api/personApi, then called PersonApi.getAll method. **getAll** is async method, we chain it with **then** method like getAll().then(res). res contains the response data from the Api. In this test, we are just displaying whatever we receive from the web Api.

We also need to test procecss.env.REACT_APP_API_BASE_URL, whether it is correctly being set from our .env file or not.

Run the project using npm start, it will start in development mode and will load .env.development file, which contains the following line.

```javascript
REACT_APP_API_BASE_URL=https://localhost:7277/api
```

After running the app with npm start, open localhost:3000 in browser. Open Developer tools, Console in the browser, so that we can view console.log and error messages. The console should print the correct base URL.

If it prints base url: undefined, that means the environment variable is not loaded from .env.development. Make sure that .env.development file exists in **project root**, where App.tsx and package.json is present. It **must not be in the src folder**.

Move the .env file to project root, stop the application and re-run the application again using npm start. Now it should display correct base url in browser console window.

Next it should also print the response from the web Api. We are calling https://localhost:7277/api/persons/ which should display all persons in Json format.

If the Json response is not displayed, check the error message. In our case, we got the CORS error.

![cors error](/images/cors-error-1024x273.jpg "cors error")

base url is highlighted in yellow, which is correct.

CORS error is displayed in red.

## Fix CORS issue when we call web API

### Why CORS error?

This error comes when the client and API are on different hosts or port. In our case, both API and React app are on localhost. But port is different.

Web API: https://localhost:7277
React Client: http://localhost:3000

This needs to be fixed in our API. ASP.NET Core web API by default allows clients from the same host and port. The client is making requests from port 3000, by default ASP.NET will reject this request. It rejects due to security reason, no external client can call the API. It is like creating a whitelist of the clients, which can call the API.

### How to fix CORS?

We need to configure ASP.NET API and tell it that localhost:3000 is safe to connect. Open the Web API project in Visual Studio. Open Extensions\ServiceExtensions.cs file. Add a new extension method to configure CORS.

```cs
public static void ConfigureCors(this IServiceCollection services)
{
  services.AddCors(options =>
  {
    options.AddPolicy("CorsPolicy", builder =>
    {
      builder
      .WithOrigins(
        "http://localhost:3000",
        "https://person-react.efcorebeginner.com")
      .AllowCredentials()
      .AllowAnyMethod()
      .AllowAnyHeader();
    });
  });
}
```

We added a new policy for CORS named CorsPolicy. We have allowed the following hosts to make http requests to this API.

- http://localhost:3000
- https://person-react.efcorebeginner.com

We have also allowed any method and any header, which means the API will allow our React API to call any http method with any type of header.

Now open Program.cs and call this extension method, like we did for other extension methods. This is now the 6th extension method that we added in the API.

```javascript
// Add services to the container.
builder.Services.ConfigureEnvironmentVariables();
builder.Services.ConfigureSqlContext();
builder.Services.ConfigureAutoMapper();
builder.Services.ConfigureRepositoryManager();
builder.Services.ConfigureServices();
builder.Services.ConfigureCors();
```

After configuring CORS in program.cs, we also have to call app.useCors before the app.Run method. Scroll to the end of Program.cs file and add this method, it can be second last line.

```javascript
app.UseCors("CorsPolicy");

app.Run();
```

Now stop and run the Web API project again. Stop and run the React client app again using npm start. Open localhost:3000 in browser and view Console log. It should now display the person list as follows.

![cors fixed](/images/cors-fixed-1024x306.jpg "cors fixed")

At this point, we can successfully call our Web API methods from React app. In the next tutorial, we will build UI for showing the person list.