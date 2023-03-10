# React Router Tutorial

## react-router 공식 문서 Tutorial 학습

[공식 문서: https://reactrouter.com/en/main/start/tutorial](https://reactrouter.com/en/main/start/tutorial)

## 01. Adding a Router

- Create a Browser Router. This will enable client side routing for our web app.

- Create and render a browser router in main.jsx

```jsx
// main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import './index.css';

const router = createBrowserRouter([
  {
    path: '/',
    element: <div>Hello world!</div>,
  },
]);

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

This first route is what we often call the "root route" since the rest of our routes will render inside of it. It will serve as the root layout of the UI, we'll have nested layouts as we get farther along.

## 02. The Root Route

Add the global layout for this app.

- Create `src/routes/root.jsx`

- Create the root layout component

```jsx
// root.jsx
export default function Root() {
  return (
    <>
      <div id='sidebar'>
        <h1>React Router Contacts</h1>
        <div>
          <form id='search-form' role='search'>
            <input
              id='q'
              aria-label='Search contacts'
              placeholder='Search'
              type='search'
              name='q'
            />
            <div id='search-spinner' aria-hidden hidden={true} />
            <div className='sr-only' aria-live='polite'></div>
          </form>
          <form method='post'>
            <button type='submit'>New</button>
          </form>
        </div>
        <nav>
          <ul>
            <li>
              <a href={`/contacts/1`}>Your Name</a>
            </li>
            <li>
              <a href={`/contacts/2`}>Your Friend</a>
            </li>
          </ul>
        </nav>
      </div>
      <div id='detail'></div>
    </>
  );
}
```

- Set `<Rout>` as the root route's `element`

```jsx
// main.jsx
/* existing imports */
import Root from './routes/root';

const router = createBrowserRouter([
  {
    path: '/',
    element: <Root />,
  },
]);

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

## 03. Handling Not Found Errors

It's always a good idea to know how your app responds to errors early in the project because we all write far more bugs than features when building a new app! Not only will your users get a good experience when this happens, but it helps you during development as well.

우리가 만든 어플리케이션은 아직 Root 페이지 밖에 없기 때문에 다른 경로로 이동하라는 링크는 에러를 발생시킨다.

Anytime your app throws an error while rendering, loading data, or performing data mutations, React Router will catch it and render an error screen. Let's make our own error page.

- Create an error page component, `src/error-page.jsx`.

```jsx
// error-page.jsx
import { useRouteError } from 'react-router-dom';

export default function ErrorPage() {
  const error = useRouteError();
  console.error(error);

  return (
    <div id='error-page'>
      <h1>Oops!</h1>
      <p>Sorry, an unexpected error has occurred.</p>
      <p>
        <i>{error.statusText || error.message}</i>
      </p>
    </div>
  );
}
```

- Set the `<ErrorPage>` as the `errorElement` on the root route

```jsx
// main.jsx
/* previous imports */
import ErrorPage from './error-page';

const router = createBrowserRouter([
  {
    path: '/',
    element: <Root />,
    errorElement: <ErrorPage />,
  },
]);

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

Note that `useRouteError` provides the error that was thrown. When the user navigates to routes that don't exist you'll get an error response with a "Not Found" `statusText`. We'll see some other errors later in the tutorial and discuss them more.

For now, it's enough to know that pretty mush all of your errors will now be handled by this page instead of infinite spinners, unresponsive pages, or blank screens.

## 04. The Contact Route UI

Instead of a 404 "Not Found" page, we want to actually render something at the URLs we've linked to. For that, we need to make a new route.

- Create the contact route module, `src/routes/contact.jsx`

- Add the contact component UI

```jsx
// contact.jsx
import { Form } from 'react-router-dom';

export default function Contact() {
  const contact = {
    first: 'Your',
    last: 'Name',
    avatar: 'https://placekitten.com/g/200/200',
    twitter: 'your_handle',
    notes: 'Some notes',
    favorite: true,
  };

  return (
    <div id='contact'>
      <div>
        <img key={contact.avatar} src={contact.avatar || null} />
      </div>

      <div>
        <h1>
          {contact.first || contact.last ? (
            <>
              {contact.first} {contact.last}
            </>
          ) : (
            <i>No Name</i>
          )}{' '}
          <Favorite contact={contact} />
        </h1>

        {contact.twitter && (
          <p>
            <a target='_blank' href={`https://twitter.com/${contact.twitter}`}>
              {contact.twitter}
            </a>
          </p>
        )}

        {contact.notes && <p>{contact.notes}</p>}

        <div>
          <Form action='edit'>
            <button type='submit'>Edit</button>
          </Form>
          <Form
            method='post'
            action='destroy'
            onSubmit={(event) => {
              if (!confirm('Please confirm you want to delete this record.')) {
                event.preventDefault();
              }
            }}
          >
            <button type='submit'>Delete</button>
          </Form>
        </div>
      </div>
    </div>
  );
}

function Favorite({ contact }) {
  // yes, this is a `let` for later
  let favorite = contact.favorite;
  return (
    <Form method='post'>
      <button
        name='favorite'
        value={favorite ? 'false' : 'true'}
        aria-label={favorite ? 'Remove from favorites' : 'Add to favorites'}
      >
        {favorite ? '★' : '☆'}
      </button>
    </Form>
  );
}
```

- Import the contact component and create a new route

```jsx
// main.jsx
/* existing imports */
import Contact from './routes/contact';

const router = createBrowserRouter([
  {
    path: '/',
    element: <Root />,
    errorElement: <ErrorPage />,
  },
  {
    path: 'contacts/:contactId',
    element: <Contact />,
  },
]);

/* existing code */
```

Click one of the links or visit `/contacts/1` we get our new component!

However, it's not inside of our root layout.

## 05. Nested Routes

We want the contact component to render inside of the `<Root>` layout.

We do it by making the contact route a child of the root route.

- Move the contacts route to be a child of the root route

```jsx
// main.jsx
const router = createBrowserRouter([
  {
    path: '/',
    element: <Root />,
    errorElement: <ErrorPage />,
    children: [
      {
        path: 'contacts/:contactId',
        element: <Contact />,
      },
    ],
  },
]);
```

You'll now see the root layout again but a blank page on the right.

We need to tell the root route where we want it to render its child routes. We do that with `<Oullet>`.

Find the `<div id="detail">` and put an outlet inside

- Render an [`<Outlet>`](https://reactrouter.com/en/main/components/outlet)

```jsx
// root.jsx
import { Outlet } from 'react-router-dom';

export default function Root() {
  return (
    <>
      {/* all the other elements */}
      <div id='detail'>
        <Outlet />
      </div>
    </>
  );
}
```

## 06. Client Side Routing

When we click the links in the sidebar, the browser is doing a full document request for the next URL instead of using React Router.

Client side rendering allows our app to update the URL without requesting another document from the server.

Instead, the app can immediately render new UI.

Let's make it happen with `<Link>`.

- Change the sidebar `<a href>` to `<Link to>`

```jsx
// root.jsx
import { Outlet, Link } from 'react-router-dom';

export default function Root() {
  return (
    <>
      <div id='sidebar'>
        {/* other elements */}

        <nav>
          <ul>
            <li>
              <Link to={`contacts/1`}>Your Name</Link>
            </li>
            <li>
              <Link to={`contacts/2`}>Your Friend</Link>
            </li>
          </ul>
        </nav>

        {/* other elements */}
      </div>
    </>
  );
}
```

You can check that it's not requesting documents anymore when URL is changed in the devtools.
