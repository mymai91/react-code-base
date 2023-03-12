# react-code-base
### 1) Create react-app with typescript

```
npx create-react-app interview-code-base  --template typescript
```

```
cd interview-code-base
```

### 2) Install tailwind css
https://www.npmjs.com/package/@material-tailwind/react

```
npm i @material-tailwind/react

npm install -D tailwindcss

npx tailwindcss init
```

Add the paths to all of your template files in your `tailwind.config.js` file.

```
/** @type {import('tailwindcss').Config} */
const withMT = require("@material-tailwind/react/utils/withMT");
 
module.exports = withMT({
  content: ["./src/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
});
```

Add Tailwind’s layers to your **`./src/index.css`** file

```
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Wrap your entire application with the ThemeProvider coming from `@material-tailwind/react` at `src/index.tsx`

```
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

// @material-tailwind/react
import { ThemeProvider } from "@material-tailwind/react";

const root = ReactDOM.createRoot(document.getElementById("root"));

root.render(
  <React.StrictMode>
    <ThemeProvider>
      <App />
    </ThemeProvider>
  </React.StrictMode>
);
```

From `App.tsx` add wrap App with memo `

```
import React, { memo } from "react";
...

function App(): React.ReactElement {
}

export default memo(App);
```

### 3) Installing React-query

https://tanstack.com/query/latest/docs/react/quick-start

```
npm i @tanstack/react-query
```

open `src/index.tsx` add `QueryClient` & `QueryClientProvider`

```
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

// Create a client
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60,
    },
  },
});

root.render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <ThemeProvider>
        <App />
      </ThemeProvider>
    </QueryClientProvider>
  </React.StrictMode>
);
```

