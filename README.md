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

### 4) Installing Axios

https://github.com/axios/axios

```
npm install axios
```

Create src/services/api.ts

```
import axios from "axios";

const BASE_API_URL =
  process.env.REACT_APP_BASE_API_URL || "http://localhost:3001/api/v1";

const Api = () => {
  const instance = axios.create({
    baseURL: BASE_API_URL,
    timeout: 1000 * 60,
    headers: {
      "Content-Type": "application/json",
    },
  });

  instance.interceptors.response.use(
    (response) => response,
    (error) => error.response
  );
  return instance;
};

export default Api;

```

create src/services/authApi.ts

```
import axios from "axios";

const BASE_API_URL =
  process.env.REACT_APP_BASE_API_URL || "http://localhost:3001/api/v1";

const authApi = () => {
  const instance = axios.create({
    baseURL: BASE_API_URL,
    timeout: 50000,
    headers: {
      "Content-Type": "application/json",
    },
  });

  // Add a request interceptor to instance
  instance.interceptors.request.use(
    (config) => {
      const token = sessionStorage.getItem("authToken");
      // eslint-disable-next-line no-param-reassign
      config.headers.Authorization = `Bearer ${token}`;

      return config;
    },
    (error) => Promise.reject(error)
  );

  instance.interceptors.response.use(
    (response) => response,
    (error) => {
      if (!error.response || error.response.status === 401) {
        sessionStorage.clear();
        window.location.href = "/sign_in";
      } else {
        Promise.reject(error);
      }
    }
  );

  return instance;
};

export default authApi;

```

example: want to call api

```
import api from './api';

export const getLikeMenu = (page: number) =>
	api().get('menu', {
		params: {
			like: true,
			page,
		},
	});
 ```
 
 ```
import authApi from './authApi';

const approveInvitation = (id) => authApi().patch(`notifications/${id}/approve`);

export default approveInvitation;
 ```
 
 ```
import authApi from './authApi';

const updateCollection = ({ id, name, description }) => authApi().patch(`collections/${id}`, { name, description });

export default updateCollection;
```

