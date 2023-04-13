---
external: false
title: "Taming React.lazy(): Implementing a Custom Retry Mechanism for Dynamically Loaded Modules"
description: Discover how implementing a custom retry mechanism for React.lazy() can help tackle challenges related to dynamically loaded modules.

date: 2023-04-12
---

As developers, our goal is to create efficient and performant web applications. One way to achieve this is by loading modules on demand, which helps reduce the initial loading time and improve the user experience. React.lazy() helps in this regard, allowing us to load components dynamically as they are needed. However, it can introduce some challenges. In this post, we'll share our experience with handling issues related to dynamically loaded modules and how implementing a custom retry mechanism helped reduce app crashes by 92%.

## The Power of React.lazy()

React.lazy() is a feature of React that allows us to load components only when they are needed. By splitting our app into smaller chunks and loading them on demand, we can reduce the initial bundle size, speed up the loading process, and provide a better overall user experience.

However, as with any powerful tool, there are some potential pitfalls when working with React.lazy(). One such challenge is handling errors related to fetching the dynamically imported modules.

## Understanding the Challenges

Sometimes, dynamically imported modules fail to load due to network issues, temporary server downtime, or a new deployment taking place. When this happens, users may experience app crashes, rendering the application unusable.

To address these issues, we need a robust solution that can handle errors gracefully and retry the module import process when necessary. This is where a custom retry mechanism comes into play.

## Implementing a Custom Retry Mechanism

The idea behind a custom retry mechanism is to create a wrapper function that handles retries in case of errors when importing modules with React.lazy(). This function should:

1. Attempt to import the module using the provided import function.
2. Catch any errors that occur during the import process.
3. Retry the import process a specified number of times, with a delay between each attempt.
4. Perform a full-page reload if all retries fail, and store a flag in sessionStorage to prevent infinite reloading.

Here's the code for our custom retry mechanism:

```jsx
// utils/retryLazy.js
import React from "react";

const retryImport = (importFn, moduleName, retriesLeft = 3, delay = 1000) => {
  return new Promise((resolve, reject) =>
    importFn()
      .then(resolve)
      .catch((error) => {
        setTimeout(() => {
          console.log(`Retrying... Retries left: ${retriesLeft}`);
          if (retriesLeft === 1) {
            const reloadKey = `reload_${moduleName}`;
            const hasReloaded = sessionStorage.getItem(reloadKey);

            if (!hasReloaded) {
              sessionStorage.setItem(reloadKey, "true");
              window.location.reload();
            } else {
              sessionStorage.removeItem(reloadKey);
              reject(error);
            }
            return;
          }
          retryImport(importFn, moduleName, retriesLeft - 1, delay).then(
            resolve,
            reject
          );
        }, delay);
      })
  );
};

const retryLazy = (importFn, moduleName, retriesLeft = 3, delay = 1000) => {
  return React.lazy(() =>
    retryImport(importFn, moduleName, retriesLeft, delay)
  );
};

export default retryLazy;
```

To use this custom retry mechanism, simply wrap your dynamic imports with **`retryLazy()`**:

```jsx
// App.js
import React, { Suspense } from "react";
import retryLazy from "./utils/retryLazy";

const MyComponent = retryLazy(() => import("./MyComponent"), "MyComponent");

function App() {
  return (
    <div className="App">
      <Suspense fallback={<div>Loading...</div>}>
        <MyComponent />
      </Suspense>
    </div>
  );
}

export default App;
```

With this implementation, the custom retry mechanism will handle errors during the import process and retry the import with a specified delay between each attempt. If all retries fail, it will perform a full-page reload, but only once per module, thanks to the **`sessionStorage`** flag.

## Impressive Results: A 92% Reduction in App Crashes

After implementing the custom retry mechanism, we observed a significant improvement in our app's stability. The number of app crashes dropped by an astounding 92%, greatly enhancing the user experience and reducing the negative impact on users.

This custom retry mechanism demonstrates how a simple yet effective solution can help tackle the challenges of working with dynamically loaded modules in React. By handling errors gracefully and retrying the import process when necessary, we can ensure a more resilient and robust application.

In summary, using React.lazy() to load modules dynamically can provide significant performance benefits, but it's essential to handle errors effectively to avoid app crashes. By implementing a custom retry mechanism, we can enhance our app's stability and provide a better user experience. Give it a try in your project!

## Bonus: npm package

If you want to try the function described in this article, you can use the npm package I published, [react-retry-lazy](https://www.npmjs.com/package/react-retry-lazy). Simply install it and start using it! Any feedback is much appreciated.
