# Next.js Speedrun — Concepts You'll Need Along the Way


## Destructuring: 

Notice that the main function produced an output cardData. Usually in different languages, you try to get the property of an object then store it inside an instance
```tsx
export default async function Page() {
  const cardData = await fetchCardData(); // <--- main function
  const totalPaidInvoices = cardData.totalPaidInvoices;
  const totalPendingInvoices = cardData.totalPendingInvoices;
  const numberOfInvoices = cardData.numberOfInvoices;
  const numberOfCustomers = cardData.numberOfCustomers;

```
Instead, you can just write it like this:
```tsx
const {
    numberOfInvoices,
    numberOfCustomers,
    totalPaidInvoices,
    totalPendingInvoices,
  } = await fetchCardData();
```
You can only do this when your function has 
```
fetchCardData() {
  //...
  return {
        numberOfCustomers,
        numberOfInvoices,
        totalPaidInvoices,
        totalPendingInvoices,
      };
}
```
The reason why this works is because destructing comes from the concept of getting specific elements in an array: 
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring



## Fetching Data
Two options:

Option 1: Traditional / Client Fetching
```
Browser (React) → Next.js API Route → Database
```
You use fetch("/api/users") inside useEffect.
``` tsx
useEffect(() => {
  fetch("/api/users")
    .then(res => res.json())
    .then(setUsers);
}, []);
```
This exists because client-side React runs in the user’s browser.
```
const users = await sql`SELECT * FROM users`;
```
Option 2: No API Layer (React Server Components)
Flow:
```
Server Component → Database
(never touches browser)
```

```tsx
export default async function Page() {
  const users = await sql`SELECT * FROM users`;
  return <div>{users.length}</div>;
}
```

When you DO NOT need an API

Reading data during page render.
Examples:

- dashboard statistics
- invoices list
- blog posts
- product catalog
- admin overview

When you still need an API layer:
| Feature                   | Needs API? | Why                       |
| ------------------------- | ---------- | ------------------------- |
| Login form                | YES        | browser sends credentials |
| Button click mutation     | YES        | user action               |
| Polling                   | YES        | client repeatedly asks    |
| Mobile app / external app | YES        | not a server component    |

This is because when Server Component only runs when the page loads on the server. User interaction = API required.
See more on: https://nextjs.org/learn/dashboard-app/fetching-data#choosing-how-to-fetch-data

## Route Groups
![Overview](https://nextjs.org/_next/image?url=https%3A%2F%2Fh8DxKfmAPhn8O0p3.public.blob.vercel-storage.com%2Flearn%2Fdark%2Froute-group.png&w=1920&q=75)

Route groups allow you to organize files into logical groups without affecting the URL path structure. When you create a new folder using parentheses (), the name won't be included in the URL path. So /dashboard/(overview)/page.tsx becomes /dashboard.

- This is useful for loading skeletons. 
