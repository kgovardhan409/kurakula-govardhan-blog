---
title: "Frontend Interview Questions - Angular, React & JavaScript"
date: 2026-08-01
lastmod: 2026-08-01
weight: 3
draft: false
tags: ["Frontend", "React", "Angular", "JavaScript", "Interview"]
categories: ["Frontend"]
---

Quick reference for frontend interview questions (Angular, React, JavaScript) organized by company and topic. Click on any question to expand details.

<!--more-->

---

<details>
<summary><strong>What is React Query?</strong> - <code>React</code></summary>

### Problem Statement

What is React Query and what problem does it solve?

### Definition

React Query (now **TanStack Query**) is a data-fetching / **server-state** management library for React. It handles fetching, caching, and syncing data from an API — so you stop hand-rolling `useEffect` + `useState` fetch logic.

### Why Not Just Redux/Context?

Redux/Context manage **client state** (UI state you own). React Query manages **server state** — data that lives on a server, can go stale, and needs refetching. Mixing the two leads to a lot of boilerplate (loading/error flags, manual cache invalidation).

### Key Features

- **Caching**: dedupes identical requests, serves cached data instantly while refetching in the background (stale-while-revalidate)
- **Automatic refetching**: on window focus, network reconnect, or interval
- **Retries**: failed requests retry automatically with backoff
- **Pagination / Infinite queries**: built-in helpers (`useInfiniteQuery`)
- **Mutations**: `useMutation` for POST/PUT/DELETE with optimistic updates

### Basic Usage

```javascript
function Todos() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['todos'],
    queryFn: () => fetch('/api/todos').then(res => res.json()),
  });

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;
  return <ul>{data.map(todo => <li key={todo.id}>{todo.title}</li>)}</ul>;
}
```

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Fetch API Data in React with useEffect + useState</strong> - <code>React</code> | <code>TypeScript</code></summary>

### Problem Statement

Fetch a user's data from an API and render it, using React hooks and TypeScript. A common warm-up coding question — it's really testing async handling, hook usage, and typing discipline.

**API:** `GET https://jsonplaceholder.typicode.com/users/1`
```json
{
  "id": 1,
  "name": "Leanne Graham",
  "username": "Bret",
  "email": "Sincere@april.biz",
  "phone": "1-770-736-8031 x56442",
  "website": "hildegard.org"
}
```

### Common Mistakes Under Interview Pressure

- Calling `fetch()` without `await` or `.json()` — you end up storing the unresolved `Promise`/`Response` object, not the data
- Declaring an `interface` inside the component body instead of at module level
- Typing `useState` as `any`, losing all type safety
- Malformed `useEffect` (missing arrow function, stray parens)

### Corrected Solution

```tsx
interface User {
  id: number;
  name: string;
  username: string;
  email: string;
  phone: string;
  website: string;
}

function App() {
  const [user, setUser] = useState<User | null>(null);

  const getUserData = async () => {
    const response = await fetch("https://jsonplaceholder.typicode.com/users/1");
    const data: User = await response.json();
    setUser(data);
  };

  useEffect(() => {
    getUserData();
  }, []);

  return (
    <div className="App">
      <h2>User Data</h2>
      {user ? (
        <div>
          <p><strong>Name: {user.name}</strong></p>
          <p><strong>Website: {user.website}</strong></p>
          <p><strong>Email: {user.email}</strong></p>
          <p><strong>Phone: {user.phone}</strong></p>
        </div>
      ) : (
        <div>No data</div>
      )}
    </div>
  );
}
```

### Key Points

1. **`interface` at module level** — not redeclared on every render inside the component
2. **`useState<User | null>`** instead of `any` — get compile-time safety on every field you access
3. **`await` both the fetch and `.json()`** — `fetch()` only resolves the response headers, the body still needs a second async step
4. **Empty dependency array `[]`** — runs the fetch once, on mount, not on every render
5. **Bonus (expected in a complete answer):** add an `isLoading` boolean so "still fetching" and "fetched but empty" aren't both shown as "No data"

**Companies:** <code>Nike</code>

</details>
