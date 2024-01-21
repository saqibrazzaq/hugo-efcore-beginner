---
title: 9. Displaying error messages from web API
weight: 100
ShowToc: true
TocOpen: true
---

This is the 9th article in Beginning React with web API series. In previous articles, we created user interface for list, search, create, update and delete persons. We used Axios in API helper methods and consumed our web API, which is written in ASP.NET Core. So far we only read response from the API and used the response data to render in our components. We have not handled errors yet.

Currently what will happen if the API throws exception? We only handled in Axios configuration, it catches the error and displays error details in console.log. We have not created any UI component to display error message for the app user.

In this article, you will learn how to handle errors and display on your list, create, update, delete or get pages.

## How to get the error message from web API?

Lets take the edit person page as an example. Run both web API and the React client apps and open http://localhost:3000/persons/edit/1. If the person id 1 exists in the database, the form will be loaded with person’s data. If the person id 1 is not found, the form simply does not show anything. The page is silent, we don’t know what is wrong. As a developer, we can check console window to see the Axios exception as below.

![axios error edit page](/images/axios-error-edit-page.jpg "axios error edit page")

The Axios library displays complete details about the error including status code, configuration, request and response details etc. It also contains the exception message we sent from the web API. We handled invalid person id in API project’s PersonService class. The error message “No person found with id: 1” is thrown in the method below.

```react
private Entities.Person FindPersonIfExists(int personId, bool trackChanges)
{
  var entity = _repositoryManager.PersonRepository.FindByCondition(
    x => x.PersonId == personId,
    trackChanges)
    .FirstOrDefault();
  if (entity == null) throw new Exception("No person found with id " + personId);

  return entity;
}
```

The error starts propagating from web API’s service class, it passes on to the controller class, which is passed on the ASP.NET Core which returns 500 service error with details to the client. In React, this error is received in Axios, which is displayed currently in console window.

Now back to the browser console and find this error in Axios error log.

![axios error response data](/images/axios-error-response-data.jpg "axios error response data")

In Axios, the error message from the web API is found in response.data.error string. We will use this property whenever we receive error from the web API. Lets first display this error in console from the code. Open React project’s src\persons\PersonEdit.tsx and update the loadPerson method as follows.

```react
const loadPerson = () => {
    if (personId) {
      PersonApi.get(personId)
        .then((res) => {
          setPerson(res);
        })
        .catch((error) => {
          console.log("API error: " + error.response.data.error);
        });
    }
  };
```

Previously we were only using then(res) to get the response from API. It will only be executed if the API returns with success. If there is an error, we need to write catch(error) block. And we can get the exact error we received from the API, in error.response.data.error string. Save the file and try to edit person with invalid id like http://localhost:3000/persons/edit/133. Now check the console window.

![axios display error console](/images/axios-display-error-console.jpg "axios display error console")

The yellow highlighted error message is displayed using the catch(error) code.

## Where to display this error message?

### Display error in Toast notification

We can use toast notifications to show the error message. We used toast notifications for delete confirmation message previously. When user deletes a person, he gets message in notification, which disappears after 2 seconds. Lets update our catch block to do that. The updated code for loadPerson method should look like below.

```react
const loadPerson = () => {
  if (personId) {
    PersonApi.get(personId)
      .then((res) => {
        setPerson(res);
      })
      .catch((error) => {
        console.log("API error: " + error.response.data.error);
        toast({
          title: "Failed to get Person",
          description: error.response.data.error,
          status: "error",
          position: "bottom-right",
        });
      });
  }
};
```

Title is “Failed to get person”. In description we displayed the exact error message from the web API. Refresh the url with invalid id like http://localhost:3000/persons/edit/133, the result should look like as shown in the screenshot below.

![get person toast error notification](/images/get-person-toast-error-notificattion-1024x482.jpg "get person toast error notification")

## Display error somewhere on the Page

The toast notification disappears after 2 seconds. Such notifications are good for confirmation type messages like person is added, person id deleted. We just need to confirm, then continue our work. But error messages should not disappear. We just don’t need these for 2 seconds. Like form validation, error messages should keep displayed on screen, until the issue is fixed.

### Alert component

Chakra UI’s Alert component is suitable for displaying alert error or success messages. We will add the Alert component in a new React component, with two props, title and description. This way whenever we want to show alert box, we will simply render our React component with props.

Create a new file **Alerts.tsx** in **src\utility** folder. Update this file as follows.

```react
import {
  Alert,
  AlertDescription,
  AlertIcon,
  AlertTitle,
} from "@chakra-ui/react";

interface AlertMessageProps {
  title?: string;
  description?: string;
  status?: string;
}

const AlertBox: React.FC<AlertMessageProps> = (props) => {
  return (
    <Alert status={props.status == "success" ? "success" : "error"}>
      <AlertIcon />
      <AlertTitle>{props.title ?? "Title"}</AlertTitle>
      <AlertDescription>
        {props.description ?? "Description"}
      </AlertDescription>
    </Alert>
  );
};

export { AlertBox as SuccessAlert };
```

We have added a functional React component in this file. It displays title and description strings from props. If status in props is success, it will display success alert, otherwise it will display error alert. The difference between success and error is only color. Success alert will display green box, error alert will display red box.

### Display alert when web API returns error

We have defined the alert box. Lets display the error message in this alert now. Open src\PersonEdit.tsx and declare a state variable named error as follows.

```react
const [error, setError] = useState("");
```

In return statement of PersonEdit function, display the alert box as follows.

```react
return (
  <Box width={"100%"} p={4}>
    <Stack spacing={4} as={Container} maxW={"3xl"}>
      {displayHeading()}
      {error && <AlertBox description={error} />}
      {showUpdateForm()}
    </Stack>
  </Box>
);
```

If the state variable error has some value, the alert box will be displayed.

Now we need to set the error state variable in catch block of loadPerson. Update the catch block of loadPerson as follows.

```react
const loadPerson = () => {
  setError("");
  if (personId) {
    PersonApi.get(personId)
      .then((res) => {
        setPerson(res);
      })
      .catch((error) => {
        setError(error.response.data.error);
        toast({
          title: "Failed to get Person",
          description: error.response.data.error,
          status: "error",
          position: "bottom-right",
        });
      });
  }
};
```

In start of loadPerson, we reset the error. And in the catch(error) block, we set the error message from the web API in the state variable.

Save the file and run the React client app with invalid person id. It should display both error alert box and toast notification.

![error with toast notification load person](/images/error-with-toast-notification-load-person.jpg "error with toast notification load person")

We can set the error wherever we call web API. We can add catch block in createPerson and updatePerson methods as follows.

```react
const updatePerson = (values: PersonReqEdit) => {
  setError("");
  PersonApi.update(personId, values).then((res) => {
    toast({
      title: "Success",
      description: "Person updated successfully.",
      status: "success",
      position: "bottom-right",
    });
    navigate("/persons");
  }).catch(error => {
    setError(error.response.data.error);
  });
};

const createPerson = (values: PersonReqEdit) => {
  setError("")
  PersonApi.create(values).then((res) => {
    toast({
      title: "Success",
      description: "Person created successfully.",
      status: "success",
      position: "bottom-right",
    });
    navigate("/persons");
  }).catch(error => {
    setError(error.response.data.error);
  });
};
```

This way if there is an error in create or update person API calls, the error alert box will be displayed to the user.