Learn React
Installation
# Building a React Framework
If your app has constraints not well-served by existing frameworks, or you prefer to solve these problems yourself, you can build your own framework.
### Note
### Consider using an existing framework 
Building a framework is complex and requires extensive expertise across various domains. This complexity is not limited to React — it is a widespread challenge encountered by all UI libraries. Using an existing framework can save significant time and effort by allowing you to focus on building your application. Existing frameworks have tested, robust features and community support.
For a list of recommended frameworks, check out Creating a React App.
Building a framework is a large undertaking that often requires expertise in many different areas. Understanding your goals and requirements before starting to build your own framework can help guide your development process and save a considerable amount of time.
For example, if you need to build a framework that integrates with a specific system or infrastructure, it’s important to understand the features and limitations of those systems. Understanding your constraints can help guide your framework development process.
If you are building your own framework to learn, using popular tools like Vite and React Router can be a good starting point and let you focus on how to combine different tools to build a framework.
## Step 1: Install a build tool 
The first step is to install a build tool like `vite`, `parcel`, or `rsbuild`. These build tools provide features to package and run source code, provide a development server for local development and a build command to deploy your app to a production server.
### Vite 
Vite is a build tool that aims to provide a faster and leaner development experience for modern web projects.
Terminal
Copy
npm create vite@latest my-app — —template react
Vite is opinionated and comes with sensible defaults out of the box. Vite has a rich ecosystem of plugins to support fast refresh, JSX, Babel/SWC, and other common features. See Vite’s React plugin or React SWC plugin and React SSR example project to get started.
Vite is already being used as a build tool in one of our recommended frameworks: React Router.
### Parcel 
Parcel combines a great out-of-the-box development experience with a scalable architecture that can take your project from just getting started to massive production applications.
Terminal
Copy
npm install —save-dev parcel
Parcel supports fast refresh, JSX, TypeScript, Flow, and styling out of the box. See Parcel’s React recipe to get started.
### Rsbuild 
Rsbuild is an Rspack-powered build tool that provides a seamless development experience for React applications. It comes with carefully tuned defaults and performance optimizations ready to use.
Terminal
Copy
npx create-rsbuild —template react
Rsbuild includes built-in support for React features like fast refresh, JSX, TypeScript, and styling. See Rsbuild’s React guide to get started.
### Note
#### Metro for React Native 
If you’d like your framework to support React Native you’ll need to use Metro, the JavaScript bundler for React Native. Metro supports bundling for platforms like iOS and Android, but lacks many features when compared to Vite or Parcel. We recommend starting with Vite or Parcel unless your project requires React Native support.
## Step 2: Build your framework 
The build tool you select starts with a client-only, single-page app (SPA). While SPAs can be a good place to start, many SPAs will encounter problems as they grow. Frameworks can provide the scaffolding to solve these problems. Most frameworks will implement routing, code-splitting, different rendering strategies, and data-fetching. These features are interconnected. For example, if you use a router that only works on the client it could prevent you from implementing server-side rendering. The best frameworks provide a cohesive, consistent experience across these features for developers and users.
### Routing 
Routing determines what to display when a user visits a particular URL. You need to set up a router to map URLs to different parts of your app. You’ll also need to handle nested routes, route parameters, and query parameters. Most modern routers use file-based routing. Routing can be integrated with other features like:
  * **Rendering strategies** to enable different rendering strategies on different routes, so you can introduce new strategies without having to rewrite your whole app. This can decrease the time it takes for the first byte of content to be loaded (Time to First Byte), the first piece of content to be rendered (First Contentful Paint), and the largest visible content of the app to be rendered (Largest Contentful Paint).
  * **Data fetching** to enable data fetching before the page loads on a route. This can prevent layout shifts (Cumulative Layout Shift) and decrease the time it takes for the largest visible content of the app to render (Largest Contentful Paint)
  * **Code splitting** to reduce the JavaScript bundle size sent to the client and improve performance on underpowered devices. This can reduce the time it takes for the browser to respond to a user interaction (First Input Delay) and the largest visible content of the app to be rendered (Largest Contentful Paint).


If you’re not sure how to get started with routing, we recommend using React Router or Tanstack Router.
### Data-fetching 
Data-fetching is the process of fetching data from a server or other data source. You need to set up or create a data-fetching library to handle data retrieval from your server and manage the state of that data. You’ll also need to handle loading states, error states, and caching data. Data fetching can be integrated with features like:
  * **Routing** to enable data fetching to take place before page loads. This can improve how quickly a page loads and becomes visible to users (Largest Contentful Paint) and reduce time it takes for your app to be interactive (Time to Interactive).
  * **Rendering strategies** to prerender fetched data before it is sent to the client. This can reduce the time it takes for the largest visible content of the app to render (Largest Contentful Paint).


Integrating routing and data fetching is particularly important to prevent network waterfalls. In a SPA, if you fetch data during a component’s initial render, the first data fetch is delayed until all code has loaded and components have finished rendering. This is commonly known as a waterfall: instead of fetching data at the same time as your code is loading, you need to first wait for your code to load before fetching data. To address these waterfalls, your app needs to fetch the data for each route in parallel with sending code to the browser.
Popular data fetching libraries that you can use as a part of your framework include React Query, SWR, Apollo, and Relay.
### Rendering strategies 
Since the build tool you select only support single page apps (SPAs) you’ll need to implement other rendering patterns like server-side rendering (SSR), static site generation (SSG), and/or React Server Components (RSC). Even if you don’t need these features at first, in the future there may be some routes that would benefit SSR, SSG or RSC.
  * **Single-page apps (SPA)** load a single HTML page and dynamically updates the page as the user interacts with the app. SPAs are fast and responsive, but they can have slower initial load times. SPAs are the default architecture for most build tools.
  * **Streaming Server-side rendering (SSR)** renders a page on the server and sends the fully rendered page to the client. SSR can improve performance, but it can be more complex to set up and maintain than a single-page app. With the addition of streaming, SSR can be very complex to set up and maintain. See Vite’s SSR guide.
  * **Static site generation (SSG)** generates static HTML files for your app at build time. SSG can improve performance, but it can be more complex to set up and maintain than server-side rendering.
  * **React Server Components (RSC)** lets you mix build-time, server-only, and interactive components in a single React tree. RSC can improve performance, but it currently requires deep expertise to set up and maintain. See Parcel’s RSC examples.


Your rendering strategies need to integrate with your router so apps built with your framework can choose the rendering strategy on a per-route level. This will enable different rendering strategies without having to rewrite your whole app. For example, the landing page for your app might benefit from being statically generated (SSG), while a page with a content feed might perform best with server-side rendering. Using the right rendering strategy for the right routes can decrease the time it takes for the first byte of content to be loaded (Time to First Byte), the first piece of content to render (First Contentful Paint), and the largest visible content of the app to render (Largest Contentful Paint).
### Code-splitting 
Code-splitting is the process of breaking your app into smaller bundles that can be loaded on demand. An app’s code size increases with every new feature and additional dependency. Apps can become slow to load because all of the code for the entire app needs to be sent before it can be used. Caching, reducing features/dependencies, and moving some code to run on the server can help mitigate slow loading but are incomplete solutions that can sacrifice functionality if overused.
Similarly, if you rely on the apps using your framework to split the code, you might encounter situations where loading becomes slower than if no code splitting were happening at all. For example, lazily loading a chart delays sending the code needed to render the chart, splitting the chart code from the rest of the app. Parcel supports code splitting with React.lazy. However, if the chart loads its data _after_ it has been initially rendered you are now waiting twice. This is a waterfall: rather than fetching the data for the chart and sending the code to render it simultaneously, you must wait for each step to complete one after the other.
Splitting code by route, when integrated with bundling and data fetching, can reduce the initial load time of your app and the time it takes for the largest visible content of the app to render (Largest Contentful Paint).
### And more… 
These are just a few examples of the features a framework will need to consider.
There are many other problems that users need to solve like:
  * Accessibility
  * Asset loading
  * Authentication
  * Error handling
  * Mutating data
  * Navigations
  * Nested routes
  * Optimistic updates
  * Caching
  * Progressive enhancement
  * Static site generation
  * Server-side rendering


Many of these problems individually can be difficult as each problem is interconnected with the others and can require deep expertise in problem areas you may not be familiar with. If you don’t want to solve these problems on your own, you can get started with a framework that provides these features out of the box.
PreviousCreating a React App
NextAdd React to an Existing Project
