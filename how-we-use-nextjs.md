# How We Use Next.js for Frontend Development
We use React and Next.js for our frontend development. Styling is done using Tailwind CSS and network requests are handled by Axios. We prefer to use component state and only use Redux to maintain a global caching layer (which is not always needed) or to persist session information. We use Vercel for hosting and continuous deployment.

## Functional Components vs. Class Components
We always write functional components, never class components.

## State Management
We prefer to use local state management within each component instead of having a global state using Redux. We only use Redux (and redux-persist) for state persistence (e.g. of session information), or if we need to maintain a global API caching layer (which is typically overkill).
See [MitPensum-frontend](https://github.com/Kvalifik/mitpensum-frontend/tree/staging/store) for an example of how to use Redux as a caching layer.

## Communicating with Backends
We use Axios as a network request library, and encapsulate our request handling into neat React Hooks that can be imported into any component. See [MitPensum-frontend](https://github.com/Kvalifik/mitpensum-frontend/tree/staging/hooks).

## Styling
We use Tailwind CSS for styling. See [kiddo-frontend](https://github.com/Kvalifik/kiddo-frontend) for a good example.

## How to Setup a Project Using Next.js
To set up a Next.js project, it is best to duplicate an existing project. The [kiddo-frontend](https://github.com/Kvalifik/kiddo-frontend) is a great starting point.

## Continuous Deployment Using Vercel
We use Vercel for hosting the site using its continuous deployment feature, including its branch previews.