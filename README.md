# react-code-base
### 1) Create react-app with typescript

```
npx create-react-app interview-code-base  --template typescript
```

### 2) Install tailwind css
https://www.npmjs.com/package/@material-tailwind/react

```
npm i @material-tailwind/react
```

Config `tailwind.config.js`

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

Wrap your entire application with the ThemeProvider coming from `@material-tailwind/react`

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

