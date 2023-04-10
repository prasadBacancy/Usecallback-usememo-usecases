# Usecallback-usememo-usecases



<h3 align="center">
 UseMemo
</h3> 

 1) Example-1

```const Example = () => {
  const [filter, setFilter] = useState('')
  // `filteredPlayers` is a derived array that is
  // recomputed on every re-render of `Example`
  const filteredPlayers = ALL_PLAYERS.filter((player) =>
    player.name.includes(filter),
  )

  useEffect(() => {
    window.fetch('https://api.benmvp.com/players/update', {
      method: 'POST',
      // ⚠️ not including `filteredPlayers` in the
      // dependencies is a lurking bug and will trigger
      // the `react-hooks/exhaustive-deps` ESLint rule,
      // but including it in the deps will also cause
      // the effect to run every re-render 😭
      body: JSON.stringify(filteredPlayers),
    })
  }, [])

  return (
    <div>
      <input
        type="text"
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
      />
    </div>
  )
}
```

By not including filteredPlayers in the useEffect() dependencies, when it does change we won’t make another POST API call, which is a bug. However, if we do add it to the dependencies, every time the component re-renders, we’ll make a POST API call, even if the filteredPlayers data actually hasn’t changed. This is because filteredPlayers is a derived value so it’s recalculated on every re-render.
By using the useMemo() Hook, we can “cache” the value of filteredPlayers for every filter value.

```const Example = () => {
  const [filter, setFilter] = useState('')
  // "cache" `filteredPlayers` on every value of `filter`
  const filteredPlayers = useMemo(
    () => ALL_PLAYERS.filter((player) => player.name.includes(filter)),
    [filter],
  )

  useEffect(() => {
    window.fetch('https://api.benmvp.com/players/update', {
      method: 'POST',
      body: JSON.stringify(filteredPlayers),
    })
    // now the effect will only be called when `filter` changes because
    // that's the only time the `filteredPlayers` reference will change
  }, [filteredPlayers])

  return (
    <div>
      <input
        type="text"
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
      />
    </div>
  )
}
```

So now, if filter remains the same, we’ll get back the same array reference for filteredPlayers. As a result the effect will not be re-run because filteredPlayers is the exact same reference as the previous render. But when filter does change, then filteredPlayers is recalculated and we have a new reference. As such, the effect will also be re-run.



  2 ) Example-2


Without memoization :

```function App() {
  const [data, setData] = useState([.....])

  function format() {
    console.log('formatting ...') // this will print at every render
    const formattedData = []
    data.forEach(item => {
      const newItem = // ... do somthing here, formatting, sorting, filtering (by date, by text,..) etc
      if (newItem) {
        formattedData.push(newItem)
      }
    })
    return formattedData
  }

  const formattedData = format()

  return <>
    {formattedData.map(item => <div key={item.id}>
      {item.title}
    </div>)}
  </>
}
```
With Memoization

  ```function App() {
  const [data, setData] = useState([.....])

  function format() {
    console.log('formatting ...')
    const formattedData = []
    data.forEach(item => {
      const newItem = // ... do somthing here, formatting, sorting, filtering (by date, by text,..) etc
      if (newItem) {
        formattedData.push(newItem)
      }
    })
    return formattedData
  }

  const formattedData = useMemo(format, [data])

  return <>
    {formattedData.map(item => <div key={item.id}>
      {item.title}
    </div>)}
  <>
}
```

  3 ) Example-3

  ```import { useMemo } from 'react';
import ChildComponent from './ChildComponent';

function MyComponent() {
  const memoizedChild = useMemo(() => <ChildComponent prop1={prop1} prop2={prop2} />, [prop1,prop2]);

  return (
    <div>
      <p>Parent Component</p>
      {memoizedChild}
    </div>
  );
}
```

In this example, we are creating a new instance of ChildComponent and memoizing it using useMemo. The second argument to useMemo is an  array [prop1,prop2], which means that the memoized value will not be recalculated unless one of its dependencies changes.

By memoizing the child component, we can prevent unnecessary re-renders when the parent component is updated. This is particularly useful when the child component is expensive to render and its props are not changing frequently.


  4 ) Example-4
  

  ```import { useState, useCallback } from "react";
import NameInput from "./components/NameInput";
import User from "./components/User";
import users from "./data/users";
// We use “crypto-js/sha512” to simulate expensive computation
import sha512 from "crypto-js/sha512";

function App() {
  const [name, setName] = useState("");
  const handleNameChange = useCallback((name) => setName(name), []);

  const newUsers = users.map((user) => ({
    ...user,
    // An expensive computation
    address: sha512(user.address).toString()
  }));

  return (
    <div className="App">
      <NameInput name={name} handleNameChange={handleNameChange} />
      {newUsers.map((user) => (
        <User
          handleNameChange={handleNameChange}
          name={user.name}
          address={user.address}
          key={user.id}
        />
      ))}
    </div>
  );
}

export default App;
```

Memoized version of newUser.

```import { useMemo } from "react";

function App() {
  const newUsers = useMemo(
    () =>
      users.map((user) => ({
        ...user,
        address: sha512(user.address).toString()
      })),
    []
  );
  
  // Rest of the component logic here
}
```


  5 ) Example-5

  let's create an custom hook for Pagination and see where we can use useMemo().

  ```
  interface UsePaginationProps {
  /** Total number of rows */
  count: number;
  /** The current page */
  page: number;
  /** How many rows per page should be visible */
  rowsPerPage: number;
  /** What are the provided options for rowsPerPage */
  rowsPerPageOptions: number[];
}
```

```
const state: UsePaginationProps = {
  count: 27,
  page: 2,
  rowsPerPage: 10,
}
```


```
function usePagination({
  count,
  page,
  rowsPerPage,
  rowsPerPageOptions
}: UsePaginationProps) {
  const pageCount = useMemo(() => {
    return Math.ceil(count / rowsPerPage);
  }, [count, rowsPerPage]);

  const pages = useMemo(() => {
    const value = Array.from(new Array(pageCount), (_, k) => k + 1);

    if (page < 3) {
      return value.slice(0, 5);
    }

    if (pageCount - page < 3) {
      return value.slice(-5);
    }

    return value.slice(page - 3, page + 2);
  }, [page, pageCount]);

  const showFirst = useMemo(() => {
    return page > 3;
  }, [page]);

  const showNext = useMemo(() => {
    return pageCount - page > 0;
  }, [page, pageCount]);

  const showLast = useMemo(() => {
    return pageCount - page > 2;
  }, [page, pageCount]);

  const showPages = useMemo(() => {
    return pages.length !== 1;
  }, [pages.length]);

  const showPagination = useMemo(() => {
    return count >= Math.min(...rowsPerPageOptions);
  }, [count, rowsPerPageOptions]);

  const showPrevious = useMemo(() => {
    return page > 1;
  }, [page]);


  return {
    pages,
    showFirst,
    showNext,
    showLast,
    showPages,
    showPagination,
    showPrevious
  };
}

export default usePagination;
```
