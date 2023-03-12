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

### 3) Installing React-query

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

```
import { useMutation } from "@tanstack/react-query";
import { CardParams } from "../models";
import CardService from "../services";

export const useCreateCard = () => {
  return useMutation(cardService.createCard, {
    onSuccess: (res) => {
      console.log("onSuccess res");
    },

    onMutate: async (data: CardParams) => {
      /**Optimistic mutation */
    },
    onError: (_err, data, context) => {
      console.log("Please try again");
    },
  });
};

```

```
import { useCreateCard } from "../hooks/useCreateCard";


const CardForm: FC = () => {
  const [avatar, setAvatar] = useState<AvatarState>(defaultAvatar);
  const { mutate: createCardMutate } = useCreateCard();
  
  const onSubmit: SubmitHandler<CardParams> = useCallback(
    (data: CardParams) => {
      createCardMutate(
        { data },
        {
          onSuccess: (res: any) => {
            router.push(`/cards/${res.card.slug}`);
          },
        }
      );
    },
    [createCardMutate, router]
  );
  
  <form>
  	<section className="my-5">
          <Button key="submit" type="primary" onClick={handleSubmit(onSubmit)}>
            Submit
          </Button>
        </section>
 </form>

}
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
### 5) Install React-Router-dom

https://www.npmjs.com/package/react-router-dom

https://reactrouter.com/en/main/route/lazy

```
npm i react-router-dom
```

create `routes/BaseRoutes.tsx`

```
import React, { Suspense, memo, useEffect, lazy } from "react";
import {
  Routes,
  Route,
  useNavigate,
  useLocation,
  matchPath,
} from "react-router-dom";
import { ProtectedRoute } from "./ProtectedRoute";

export const routes = {
  HOMEPAGE: "/",
  LOGIN_PAGE: "/login",
  EDIT_PROFILE_PAGE: "/profile/:urlHash",
};

export const publicRouteData: any = [
  {
    path: routes.HOMEPAGE,
    Element: lazy(() => import("../../Auth/LoginPage")),
  },
  {
    path: routes.LOGIN_PAGE,
    Element: lazy(() => import("../../Auth/LoginPage")),
    exact: true,
  },
];

export const privateRouteData: any = [
  {
    path: routes.EDIT_PROFILE_PAGE,
    Element: lazy(() => import("../../ProfilePage/EditProfilePage")),
    exact: true,
    needAuthentication: true,
  },
];

const BaseRoutes = () => {
  const navigate = useNavigate();
  const location = useLocation();

  return (
    <Routes>
      {publicRouteData.map((route) => (
        <Route key={route.path} path={route.path} element={<route.Element />} />
      ))}
      {privateRouteData.map((route) => (
        <Route
          key={route.path}
          path={route.path}
          element={
            <ProtectedRoute>
              <route.Element />
            </ProtectedRoute>
          }
        />
      ))}
      <Route component={NotFound} />
    </Routes>
  );
};

export default memo(BaseRoutes);

```

create `routes/ProtectRoutes.tsx`

```
import { Navigate } from "react-router-dom";
import { useAuth } from "../hooks/useAuth";

export const ProtectedRoute = ({ children }) => {
  const { user } = useAuth();
  if (!user) {
    // user is not authenticated
    return <Navigate to="/" />;
  }
  return children;
};

```

open `App.tsx`

```
import React, { memo } from "react";
import "./App.css";
import { BrowserRouter } from "react-router-dom";
import BaseRoutes from "./routers/BaseRoutes";

function App(): React.ReactElement {
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

### 8) React-testing-library

https://testing-library.com/docs/react-testing-library/example-intro

```
npm install --save-dev @testing-library/react
```
### 9) React-toasy

https://www.npmjs.com/package/react-toastify

```
npm i react-toastify
```
