# React Router Dom

This is a summary of the [React Router tutorial](https://reactrouter.com/en/main/start/tutorial).

**Demo**: [sximenez.github.io/2023-03-mar-react-router-dom](https://sximenez.github.io/2023-03-mar-react-router-dom/)

## In a nutshell

An application needs a method allowing to navigate between its differents screens.

A `router` can serve this purpose, and in the case of SPAs (single page apps), it allows to change screens without reloading the entire app.

This gives the user the impression of instantaneousness and performance.

React Router is a library providing this sort of client-side routing for React apps.

## 1. Add a router

`createBrowerRouter`: as of the time of this writing, this is the "recommended router for all React Router web projects".

At its simplest, it takes a **path** (usually a _slug_ or the last segment of the url) and an **element** (usually a component we can name `<Root />`):

```Javascript
const router = createBrowserRouter([
  {
    path: "/",
    element: <div>Hello world!</div>,
  },
]);
```

This first path is our _root route_: all other roots will render inside of it as its **children**.

The root route contains the UI layout of the app, usually with the navigation links of the app.

## 2. Add a component to render the router

`RouterProvider`: this is the component that React will use to render the router.

While the router loads, you can use `fallbackElement` to show a spinner or something to let the user know a page is loading.

```Javascript
ReactDOM.createRoot(document.getElementById("root")).render(
  <RouterProvider
    router={router}
    fallbackElement={<BigSpinner />}
  />
);
```

## 3. Add an error page component

For a better user experience, an error page can be created and linked to the root route using `errorElement`.

```Javascript
const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    errorElement: <ErrorPage />
  },
]);
```

## 4. Add children routes

Children routes allow to render components within the layout declared in the Root component.

```Javascript
const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    errorElement: <ErrorPage />,
    children: [
      {
        path: "contacts/:contactId",
        element: <Contact />,
      }
    ]
  },
]);
```

To render the child within the parent, we need to declare the `<Outlet />` component in the `<Root />`.

## 4. Linking with Links

The `<Link></Link>` component is like an `a` element, only that it will not reload pages when linking (see the inspect network tab).

## 5. Load data

React Router has a convention when loading data into a view.

It provides two APIs:

`loader`

You can use a loader to provide data to a route **before it renders**.

It can be an element, a component or a function.

`useLoaderdata`

This is a hook, a React function, that allows to use the return value of a loader in a rendered screen.

## 6. Use parameters from the URL

React Router uses the color `:` to declare dynamic slugs in a URL:

```Javascript
  {
    path: "contacts/:contactId",
    element: <Contact />,
  }
```

We can pass parameters or "params" to a loader that match with a dynamic slug.

:contactID will get the value of params.contactID for example.

## 7. Create data

When submitting a classic `<form>`, the browser really acts like an `a` element that sends a post request.

The React Router `<Form>` component lets you use this simple method, but instead of sending a post request, it sends an async request to the loader.

The post request is translated into an `action` that can be applied to the loader, and defined in the router:

```Javascript
const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    errorElement: <ErrorPage />,
    loader: rootLoader,
    action: rootAction,
    children: [
      {
        path: "contacts/:contactId",
        element: <Contact />,
        loader: conctactLoader,
      }
    ],
  },
]);

export async function loader() {
  const contacts = await getContacts();
  return { contacts };
}

export async function action() {
  const contact = await createContact();
  return { contact };
}
```

## 8. Update data

We can use the `<Form>` component to update data too by adding a new `<Form>` on a new route: `contacts/:contactId/edit`.

```Markdown
Important: each route should have its own loader.
```

We can then set up a specific action when the form is submitted:

```Javascript
{
  path: "contacts/:contactId/edit",
  element: <EditContact />,
  loader: conctactLoader,
  action: editAction,
}

export async function action({ request, params }) {
  const formData = await request.formData();
  const updates = Object.fromEntries(formData);
  await updateContact(params.contactId, updates);
  return redirect(`/contacts/${params.contactId}`);
}
```

The magic here is that when we submit, a classic form would normally create and post a `FormData`, a sort of object containing the inputs and the values of the form.

We can send this `FormData` to the action instead, convert it to something we can access, and then redirect back to the desired page using `redirect`.

React Router revalidates all data after the submit, updating the sidebar too!

## 9. Delete data

We can do the same to delete a record:

- create a new route with the action function that will call the delete function within our model:

```Javascript
export async function action({ params }) {
  await deleteContact(params.contactId);
  return redirect("/");
}
```

Set the action within the router.

We can add a custom `errorElement` attribute if we want.

## 10. Render active links

We can use the `NavLink` component to render active links on a page, with a class function to help with the rendering:

```Javascript
<NavLink
  to={`contacts/${contact.id}`}
  className={({ isActive, isPending }) =>
    isActive
      ? "active"
      : isPending
        ? "pending"
        : ""
  }
>
{code}
</NavLink>
```

## 11. Improving responsiveness

We can use the `useNavigation` hook's state to apply a CSS class to a component.

This allows us to provide visual feedback to the user when a page is loading for example, like a fade:

```Javascript
<div
  id="detail"
  className={
    navigation.state === "loading" ? "loading" : ""
  }>

  <Outlet />

</div>
```

`Add a form, add an action, React Router does the rest.`

## 12. Add a default child route

You can set a default route for the `<Outlet />` by setting and index route and a conditional before every child:

```Javascript
const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    errorElement: <ErrorPage />,
    loader: rootLoader,
    action: rootAction,
    children: [
      { index: true, element: <Index /> },
      {
        path: "contacts/:contactId",
        element: <Contact />,
        loader: conctactLoader,
      },
```

## 13. Add a search function

Searching and filtering add essential layers of UX to apps.

We can use a React Router `<Form></Form>` for the search input, which will post to the loader.

Then, we can update the loader like this:

```Javascript
export async function loader({ request }) {
  const url = new URL(request.url);
  const q = url.searchParams.get("q");
  const contacts = await getContacts(q);
  return { contacts, q };
}
```

Some final tweaks for better UX:

```Javascript
export default function Root() {

  const { contacts, q } = useLoaderData();
  const navigation = useNavigation();

  useEffect(() => {
    document.getElementById("q").value = q;
  }, [q]);
```

We can use `useSubmit` to post data automatically (for every key stroke).

However, we have to create a variable to store the whole search string, so that we don't have a huge history:

```Javascript
onChange={
  (event) => {
    const isFirstSearch = q == null;
    submit(event.currentTarget.form, {
      replace: !isFirstSearch,
    });
  }}
```

We can also add a spinner to let the user know the search is ongoing:

```Javascript
const searching =
  navigation.location &&
  new URLSearchParams(navigation.location.search).has(
    "q"
  );
```

## 14. Modify data without navigating

The `useFetcher` hook allows to change data without adding a new entry to the history stack.

This is useful for applying tags for example.

## 15. Pathless routes

To tell the user there's a problem with a path, we could use the `errorElement` attribute as a redirect.

However, we can also use a pathless route with an element wrapping the children within.

In this case, every time there's an error, the error component we declare will show up, and in the Outlet too!

## Conclusion

This exercise has been fun and straightforward.

It gives you a better understanding of React, at least of the layers it uses to organize screens within an app.

I look forward to using the library in a next project.
