# How We Use Next.js for Frontend Development
We use React and Next.js for our frontend development. Styling is done through TailwindCSS and network requests are handled by Axios. We use Redux to maintain a caching layer, and Vercel for hosting and continous deployment.

## Function Components vs Class Components
We always write function components, never class components.

## When to use redux (and when not to)
We are in general opposed to overusing redux, and prefer only to use it as a simple caching layer for backend-data and to manage global data (like user information and session storage).
See [MitPensum-frontend](https://github.com/Kvalifik/mitpensum-frontend/tree/staging/store) as an example of how to use Redux as a caching layer.

## Communicating with backends
We use Axios as a network request library, and pack all of our request handling into neat React Hooks that can be imported into any component. See [MitPensum-frontend](https://github.com/Kvalifik/mitpensum-frontend/tree/staging/hooks).

## Styling
We use TailwindCSS for styling. See [kiddo-frontend](https://github.com/Kvalifik/kiddo-frontend) for good example.

## How to setup a project using Next.js
To set up a Next.js project, it is best to duplicate an existing project. The [kiddo-frontend](https://github.com/Kvalifik/kiddo-frontend) is a great starting point.

## Vercel
We use Vercel for hosting of the site and handling of CD as well as branch previews.
