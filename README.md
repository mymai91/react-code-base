# react-code-base
### 1) Create react-app with typescript

```
npx create-react-app interview-code-base  --template typescript
```

```
cd interview-code-base
```

```
npm i @types/react-router-dom
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

open src/modules/Authentication/apis/login.ts

```
import Api from "../../../services/api";
import { loginParams } from "../models";

const loginApi = async ({ email, password }: loginParams) => {
  const res = await Api().post("/login", {
    user: {
      email,
      password,
    },
  });
  return res.data;
};

export default loginApi;
```
 
 ```
import authApi from './authApi';

const updateCollection = ({ id, name, description }) => authApi().patch(`collections/${id}`, { name, description });

export default updateCollection;
```

### 4) Installing React-query

https://tanstack.com/query/latest/docs/react/quick-start

https://github.com/uidotdev/react-query-course/tree/16-updating-assignment/src

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

Create hook example:

open `src/modules/Authentication/models/index.ts`

```
export interface loginParams {
  email: string;
  password: string;
}
```

open `src/modules/Authentication/hooks/useLogin.ts`

```
import React from "react";
import { useMutation } from "@tanstack/react-query";
import loginApi from "../apis/login";

export const useLogin = () => {
  return useMutation(loginApi, {
    onSuccess: (res) => {
      console.log("res in hook", res);
    },
    onError: () => {},
  });
};
```

open Login component `src/modules/Authentication/components/Login.tsx`

```
import React, { FC, memo, useCallback } from "react";
import { useForm } from "react-hook-form";
import { yupResolver } from "@hookform/resolvers/yup";
import * as yup from "yup";
import { loginParams } from "../models";
import { useLogin } from "../hooks/useLogin";

const LoginSchema = yup.object().shape({
  email: yup.string().required(),
  password: yup.string().required(),
});

const Login: FC = () => {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<loginParams>({
    resolver: yupResolver(LoginSchema),
  });

  const { mutate: loginMutate } = useLogin();
  const onSubmit = useCallback(
    (data: loginParams) => {
      // loginMutate(data, {
      //   onSuccess: (res: any) => {
      //     // console.log("res", res);
      //     // router.push(`/cards/${res.card.slug}`);
      //   },
      // });
      loginMutate(data);
    },
    [loginMutate]
  );

  return (
    <div>
      <p>Login</p>
      <form onSubmit={handleSubmit(onSubmit)}>
        <div>
          <label>Email</label>
          <input {...register("email")} />
          {errors.email && <p>{errors.email?.message}</p>}
        </div>
        <div style={{ marginBottom: 10 }}>
          <label>Password</label>
          <input {...register("password")} type="password" />
          {errors.password && <p>{errors.password?.message}</p>}
        </div>
        <input type="submit" />
      </form>
    </div>
  );
};

export default memo(Login);
```

### 5) Install React-Router-dom

https://www.npmjs.com/package/react-router-dom

https://reactrouter.com/en/main/route/lazy

```
npm i react-router-dom
```

create `routes/BaseRoutes.tsx`

```
import React, { memo, lazy } from "react";
import { Route, Routes } from "react-router-dom";
import Notfound from "../modules/Dashboard/components/Notfound";
import { ProtectedRoute } from "./ProtectRoutes";

const routes = {
  HOME_PAGE: "/",
  LOGIN_PAGE: "/login",
  CREATE_CARD_PAGE: "/createcard",
};

const publicRoutes = [
  {
    path: routes.HOME_PAGE,
    element: lazy(() => import("../modules/Dashboard/components/Home")),
  },

  {
    path: routes.LOGIN_PAGE,
    element: lazy(() => import("../modules/Authentication/components/Login")),
  },
];

const privateRoutes = [
  {
    path: routes.CREATE_CARD_PAGE,
    element: lazy(() => import("../modules/Card/components/CreateCard")),
  },
];

// TODO update react-code-base

const BaseRoutes = () => {
  return (
    <Routes>
      {publicRoutes.map((route) => {
        return (
          <Route
            key={route.path}
            path={route.path}
            element={<route.element />}
          />
        );
      })}

      {privateRoutes.map((route) => {
        return (
          <Route
            key={route.path}
            path={route.path}
            element={
              <ProtectedRoute>
                <route.element />
              </ProtectedRoute>
            }
          />
        );
      })}
      <Route path="*" element={<Notfound />} />
    </Routes>
  );
};

export default memo(BaseRoutes);
```

create `routes/ProtectRoutes.tsx`

```
import React from "react";
import { Navigate } from "react-router-dom";

// TODO update react-code-base

export const ProtectedRoute = ({ children }: any) => {
  const isUser = () => {
    return false;
  };

  if (!isUser()) {
    // user is not authenticated
    return <Navigate to="/" />;
  }
  return children;
};

```

open `App.tsx`

```
import React, { memo } from "react";
import logo from "./logo.svg";
import "./App.css";
import { BrowserRouter } from "react-router-dom";
import BaseRoutes from "./routes/BaseRoutes";

function App() {
  return (
    <BrowserRouter>
      <BaseRoutes />
    </BrowserRouter>
  );
}

export default memo(App);
```

### 6) Setting up Redux in the app

```
npm install @reduxjs/toolkit react-redux uuid @types/uuid
```

Create models folder to define the types src/models/Todo.ts

```
export interface Todo {
  id: string;
  description: string;
  completed: boolean;
}
```

Setting up Redux in the app

- create folder `src/redux`
- Add file `src/redux/todoSlice.ts` todoSlice will control state management for our app
    - set up our Redux store and what Redux Toolkit calls a “slice”
    
    We define what we need
    
```
import { createSlice, PayloadAction } from "@reduxjs/toolkit";
import { Todo } from "../models/Todo";

const initialState = [] as Todo[];
const todoSlice = createSlice({
  name: "todos",
  initialState,
  reducers: {
    addTodo: {
      reducer: (state, action: PayloadAction<Todo>) => {
        state.push(action.payload);
      },
      prepare: (description: string) => ({
        payload: {
          id: uuidv4(),
          description,
          completed: false,
        } as Todo,
      }),
    },
    removeTodo(state, action: PayloadAction<string>) {
      const index = state.findIndex((todo) => todo.id === action.payload);
      state.splice(index, 1);
    },
		setTodoStatus(
      state,
      action: PayloadAction<{ completed: boolean; id: string }>
    ) {
      const index = state.findIndex((todo) => todo.id === action.payload.id);
      state[index].completed = action.payload.completed;
    },
  }
})

export const {addTodo, removeTodo, setTodoStatus} = todoSlice.actions;
export default todoSlice;
```

Setup store for Redux create file src/redux/store/index.ts

```
import { configureStore } from "@reduxjs/toolkit";
import todoSlice from "../../modules/Todo/feature/todoSlice";

export const store = configureStore({
  reducer: {
    [todoSlice.name]: todoSlice.reducer,
  },
});

export type AppDispatch = typeof store.dispatch;
export type RootState = ReturnType<typeof store.getState>;
```

Adding Redux to React App `index.tsx`

```
import React from "react";
import ReactDOM from "react-dom";
import { Provider } from "react-redux";
import App from "./App";
import { store } from "./redux/store";

ReactDOM.render(
  <React.StrictMode>
    <Provider store={store}>
    <QueryClientProvider client={queryClient}>
      <ThemeProvider>
        <App />
      </ThemeProvider>
      QueryClientProvider
    </Provider>
  </React.StrictMode>,
  document.getElementById("root")
);
```

Create modules folder and Todo is child module

```
modules/Todo/components
modules/Todo/feature
modules/Todo/hooks
modules/Todo/models
modules/Todo/services
```

 Create template src/modules/components/TodoForm.tsx
 
 ```
 import React, { FC, useState } from "react";
import { useDispatch, useSelector } from "react-redux";
import { AppDispatch, RootState } from "../../../redux/store";

const TodoForm: FC = () => {
  const [todoDescription, setTodoDescription] = useState<string>("");

  const todoList = useSelector((state: RootState) => state);
  const dispatch = useDispatch<AppDispatch>();

	const addTask = () => {
		dispatch(addTodo(todoDescription));
	}

	const removeTask = () => {
		dispatch(removeTodo(todo.id));
	}
	
	const setTask = () => {
		dispatch(
      setTodoStatus({ completed: !todo.completed, id: todo.id })
    );
  };
  return (
    <div>
      <p>TodoForm</p>
    </div>
  );
};

export default TodoForm;
```

### 7) React-hook-form

https://react-hook-form.com/get-started/

```
npm install react-hook-form
```
SchemaValidation https://react-hook-form.com/get-started/#SchemaValidation

```
npm install @hookform/resolvers yup
```
example `src/modules/Authentication/components/Login.tsx`

```
import React, { FC, memo, useCallback } from "react";
import { useForm } from "react-hook-form";
import { yupResolver } from "@hookform/resolvers/yup";
import * as yup from "yup";

const SignupSchema = yup.object().shape({
  firstName: yup.string().required(),
  age: yup.number().required().positive().integer(),
});

type SignupData = {
  firstName: string;
  lastName: string;
  age: number;
};

const Login: FC = () => {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<SignupData>({
    resolver: yupResolver(SignupSchema),
  });

  const onSubmit = useCallback((data: SignupData) => {
    console.log(JSON.stringify(data));
  }, []);

  return (
    <div>
      <p>Login</p>
      <form onSubmit={handleSubmit(onSubmit)}>
        <div>
          <label>First Name</label>
          <input {...register("firstName")} />
          {errors.firstName && <p>{errors.firstName?.message}</p>}
        </div>
        <div style={{ marginBottom: 10 }}>
          <label>Last Name</label>
          <input {...register("lastName")} />
          {errors.lastName && <p>{errors.lastName?.message}</p>}
        </div>
        <input type="submit" />
      </form>
    </div>
  );
};

export default memo(Login);
```

### 8) React-testing-library

https://testing-library.com/docs/react-testing-library/example-intro

```
npm install --save-dev @testing-library/react
```

https://testing-library.com/docs/react-testing-library/example-intro

https://mswjs.io/docs/api/rest

https://github.com/mswjs/examples/tree/master/examples/rest-react/src/mocks

```
npm install msw --save-dev
```

### 9) React-toasy

https://www.npmjs.com/package/react-toastify

```
npm i react-toastify
```

res.headers.get("authorization")
