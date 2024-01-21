---
title: 3. Setup Axios to call Persons web API
weight: 40
ShowToc: true
TocOpen: true
---

The web API is independent and can be hosted on a different machine and domain than the React client app. The communication between React client and web API will take place using the HTTP protocol. We can use JavaScript Fetch API, which is readily available. But we will use Axios library, to better manage our API calls.

Create a new folder **api** in **src**. In src\api folder create a new file axiosconfig.js as follows.

```javascript
import axios from "axios";

export const api = axios.create({
  baseURL: process.env.REACT_APP_API_BASE_URL,
});

api.interceptors.response.use(
  (res) => {
    return res;
  },
  async (err) => {
    console.log("Log inside axiosconfig.js")
    console.log(err);
    
    return Promise.reject(err);
  }
);

api.interceptors.request.use(
  (config) => {
    
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);
```

We will create an instance of axios and use it for all API calls. It sets the baseURL from environment variable. React loads the environment variables from .env files. We will create two new .env files, one for development and second for production.

We have also used Axios api interceptors for both request and response. These methods intercept the http request and can modify the request/response as we like. Currently we have used the interceptors to handle errors in sending or receiving response.

Create new file **.env.development** in project root. For now it will contain a single line setting the environment variable as below

```bat
REACT_APP_API_BASE_URL=https://localhost:7277/api
```

In my system, the Visual Studio sets the port to 7277 for the api. It will be different in your case, so edit the port number.

Create another file .env.production in project root. It will set the environment variable to the production website.

```bat
REACT_APP_API_BASE_URL=https://person-api.efcorebeginner.com/api
```

When you run the React client app with npm start command, it will load the .env.development file.

Now we will create service classes which will be used to call our web API. Create a new file **personApi.js** in **src\api** folder. Add a new method getAll in it. The content of the file will be as follows.

```javascript
import { api } from "./axiosconfig"

export const PersonApi = {
  getAll: async function () {
    const response = await api.request({
      url: `/persons/`,
      method: "GET",
    })

    return response.data
  },
}
```

**personApi** class uses Axios for making http requests. The **getAll** method sends GET http request to our API and receives response. The response data is sent back to the caller. The API classes just focus on sending http requests and receive response from the API. The base URL, api error handling and other http related things are configured in axios setup (axiosconfig.js).

In the next article, we will create the page to list all Persons from the API.