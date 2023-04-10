# Usecallback-usememo-usecases



<h3 align="center">
 UseCallback
</h3> 

 1) Example-1
 Use in Debounce technique

 ```
 import { useState, useCallback } from 'react';

const DebounceExample = () => {
  const [searchTerm, setSearchTerm] = useState('');

  const debounce = useCallback((func, delay) => {
    let timeoutId;
    return function (...args) {
      if (timeoutId) {
        clearTimeout(timeoutId);
      }
      timeoutId = setTimeout(() => {
        func.apply(this, args);
      }, delay);
    };
  }, []);

  const handleSearch = useCallback(
    debounce((value) => {
      console.log('Searching for:', value);
      // Perform search logic here
    }, 500),
    [debounce]
  );

  const handleInputChange = useCallback((event) => {
    const value = event.target.value;
    setSearchTerm(value);
    handleSearch(value);
  }, [handleSearch]);

  return (
    <div>
      <input type="text" value={searchTerm} onChange={handleInputChange} />
    </div>
  );
};

export default DebounceExample;
```
In this example, we define the debounce function inside the component and memoize it using useCallback. We pass [] as the dependency array for debounce to ensure that it is only created once during the lifetime of the component.

We then pass debounce as a dependency to the memoized handleSearch function, which uses it to debounce the search logic. This ensures that the debounce function is not recreated every time handleSearch is called.

Finally, we define the handleInputChange function, which is memoized using useCallback and calls handleSearch with the new value after the debounce time has passed.

By memoizing the debounce and handleSearch functions using useCallback, we can ensure that they are only recreated when necessary and avoid unnecessary re-renders of the component, which can improve the performance of the application.

 2 ) Example-2
 
 Wrapping API calls in useCallback can help optimize React components by memoizing the function that makes the API call. This can prevent unnecessary re-renders of the component and improve the overall performance of the application.

Here's an example of how to wrap an API call in useCallback:

```
import { useState, useEffect, useCallback } from 'react';

const APIExample = () => {
  const [data, setData] = useState([]);

  const fetchData = useCallback(async () => {
    try {
      const response = await fetch('https://api.example.com/data');
      const data = await response.json();
      setData(data);
    } catch (error) {
      console.error('Error fetching data:', error);
    }
  }, []);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return (
    <div>
      {data.map((item) => (
        <div key={item.id}>
          <h2>{item.title}</h2>
          <p>{item.description}</p>
        </div>
      ))}
    </div>
  );
};

export default APIExample;
```

3 ) Example-3
In React, when a parent component re-renders, all of its child components also re-render by default. However, sometimes we may want to optimize the rendering of child components if we know that their props have not changed.

One way to optimize child component rendering is by using the useCallback hook to memoize the child component function. This can be particularly useful when passing callback functions down to child components, as it ensures that the callback function reference remains the same between renders, as long as its dependencies have not changed.

Here's an example:

```
import React, { useCallback } from "react";
import ChildComponent from "./ChildComponent";

function ParentComponent() {
  const handleButtonClick = useCallback(() => {
    console.log("Button clicked");
  }, []);

  return (
    <div>
      <h1>Parent Component</h1>
      <ChildComponent onButtonClick={handleButtonClick} />
    </div>
  );
}

```

In this example, the handleButtonClick function is memoized using useCallback. This means that its reference will only change if its dependency array ([] in this case) changes. When ParentComponent re-renders, ChildComponent will only re-render if its props have actually changed, which in this case is onButtonClick. If handleButtonClick had not been memoized using useCallback, ChildComponent would re-render on every ParentComponent re-render, even if onButtonClick had not changed.

By memoizing callback functions using useCallback, we can optimize the rendering of child components and improve the overall performance of our React application.

4 ) Example-4 : useCallback for memoizing reducer in redux

In Redux, a reducer is a pure function that takes the current state and an action, and returns a new state. When using the useReducer hook in a React component to manage state, we can use the useCallback hook to memoize the reducer function and improve performance.

Here's an example:

``` 
import React, { useReducer, useCallback } from "react";
import reducer from "./reducer";

function MyComponent() {
  const [state, dispatch] = useReducer(useCallback(reducer, []), {});

  const handleClick = useCallback(() => {
    dispatch({ type: "INCREMENT" });
  }, [dispatch]);

  return (
    <div>
      <h1>Count: {state.count}</h1>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}
```

In this example, the reducer function is memoized using useCallback. This means that the reference to the reducer function will only change if its dependencies change. In this case, the dependency array is empty ([]), which means that the reducer function will only be memoized once during the lifecycle of the component.

By memoizing the reducer function, we can prevent unnecessary re-renders caused by changes to the reducer function reference. This can be particularly useful when using complex reducer logic or when working with large state trees.

It's worth noting that memoizing the reducer function is optional and may not always be necessary. If your reducer logic is simple and does not depend on any external variables, memoizing may not provide any significant performance benefits. However, if you are experiencing performance issues with your Redux store, memoizing the reducer function using useCallback can be a useful optimization technique to consider.


5 ) Example-5 : Memoizing state update functions

Memoizing state update functions in React using useCallback can improve performance and prevent unnecessary re-renders. When using the useState hook in a React component, the state update function returned by useState is a new function reference on every render. This can cause child components to re-render unnecessarily, even if the state value itself hasn't changed.

To memoize a state update function using useCallback, you can pass the state update function as the first argument to useCallback, and an empty array [] as the second argument. This tells React to memoize the function and only re-create it if one of its dependencies (in this case, none) changes.

Here's an example of how to memoize a state update function using useCallback:

```
import React, { useState, useCallback } from "react";

function MyComponent() {
  const [count, setCount] = useState(0);

  const handleIncrement = useCallback(() => {
    setCount((prevCount) => prevCount + 1);
  }, []);

  const handleDecrement = useCallback(() => {
    setCount((prevCount) => prevCount - 1);
  }, []);

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={handleIncrement}>Increment</button>
      <button onClick={handleDecrement}>Decrement</button>
    </div>
  );
}
```

In this example, the handleIncrement and handleDecrement functions are memoized using useCallback. This means that they will only be re-created if the count state value changes, which prevents unnecessary re-renders caused by changes to the function reference.



