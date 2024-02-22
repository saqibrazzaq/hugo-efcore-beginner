---
title: How to display error message in React
ShowToc: true
TocOpen: true
cover:
  image: "/images/blog/error-message-react.jpg"
  caption: "Photo by Goh Rhy Yan on Unsplash"
---

We can display the error message by setting the state variable. If there is an error doing something, catch the exception and set the state variable as below.

```react
const [errorMessage, setErrorMessage] = useState("");
const doSomething = () => {
  // call some api or do something, if there is any error set message
  setErrorMessage("Some error has occurred.");
};
```

And just display the error message wherever you want with the && operator as below.

```react
{errorMessage && <p>{errorMessage}</p>}
```

Complete code snippet of React app is given below where you can display error message on click of button. It also contains code to clear the error message.

```react
function App() {
  const [errorMessage, setErrorMessage] = useState("");
  const doSomething = () => {
    // call some api or do something, if there is any error set message
    setErrorMessage("Error: Some error has occurred.");
  };
  const clearError = () => {
    setErrorMessage("");
  };
  return (
    <div>
      <h1>Heading</h1>
      {errorMessage && <p>{errorMessage}</p>}
      <p>Do some operation below</p>
      <button onClick={doSomething}>Do Something</button>
      <button onClick={clearError}>Clear Error</button>
    </div>
  );
}
```

When page is loaded, errorMessage value will be empty, the error message will not be displayed.

![react error message page load](/images/blog/react-error-message-page-load.jpg "react error message page load")

Click on **Do Something** button. You can call an API or do some other operation in this method. Handle error in catch block. If error occurs, set the error message state variable. When errorMessage value is updated, React will render the <p> again with the updated value and the error message will be displayed on screen.

![react error message on button click](/images/blog/react-error-message-on-button-click.jpg "react error message on button click")

When you clear the error, set errorMessage state variable to empty string, React will render the component again and error message will not be displayed because of && operator between errorMessage and <p>.