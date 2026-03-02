# Next JS, React, JS SPEEDRUN





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
