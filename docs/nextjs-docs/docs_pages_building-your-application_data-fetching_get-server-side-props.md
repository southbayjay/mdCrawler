Menu
Using Pages Router
Features available in /pages
Using Latest Version
15.2.0
Using Pages Router
Features available in /pages
Using Latest Version
15.2.0
Building Your ApplicationData FetchinggetServerSideProps
# getServerSideProps
`getServerSideProps` is a Next.js function that can be used to fetch data and render the contents of a page at request time.
## Example
You can use `getServerSideProps` by exporting it from a Page Component. The example below shows how you can fetch data from a 3rd party API in `getServerSideProps`, and pass the data to the page as props:
pages/index.tsx
TypeScript
JavaScriptTypeScript
```
importtype { InferGetServerSidePropsType, GetServerSideProps } from'next'
typeRepo= {
 name:string
 stargazers_count:number
}
exportconstgetServerSideProps= (async () => {
// Fetch data from external API
constres=awaitfetch('https://api.github.com/repos/vercel/next.js')
constrepo:Repo=awaitres.json()
// Pass data to the page via props
return { props: { repo } }
}) satisfiesGetServerSideProps<{ repo:Repo }>
exportdefaultfunctionPage({
 repo,
}:InferGetServerSidePropsType<typeof getServerSideProps>) {
return (
  <main>
   <p>{repo.stargazers_count}</p>
  </main>
 )
}
```

## When should I use `getServerSideProps`?
You should use `getServerSideProps` if you need to render a page that relies on personalized user data, or information that can only be known at request time. For example, `authorization` headers or a geolocation.
If you do not need to fetch the data at request time, or would prefer to cache the data and pre-rendered HTML, we recommend using `getStaticProps`.
## Behavior
  * `getServerSideProps` runs on the server.
  * `getServerSideProps` can only be exported from a **page**.
  * `getServerSideProps` returns JSON.
  * When a user visits a page, `getServerSideProps` will be used to fetch data at request time, and the data is used to render the initial HTML of the page.
  * `props` passed to the page component can be viewed on the client as part of the initial HTML. This is to allow the page to be hydrated correctly. Make sure that you don't pass any sensitive information that shouldn't be available on the client in `props`.
  * When a user visits the page through `next/link` or `next/router`, Next.js sends an API request to the server, which runs `getServerSideProps`.
  * You do not have to call a Next.js API Route to fetch data when using `getServerSideProps` since the function runs on the server. Instead, you can call a CMS, database, or other third-party APIs directly from inside `getServerSideProps`.


> **Good to know:**
>   * See `getServerSideProps` API reference for parameters and props that can be used with `getServerSideProps`.
>   * You can use the next-code-elimination tool to verify what Next.js eliminates from the client-side bundle.
> 

## Error Handling
If an error is thrown inside `getServerSideProps`, it will show the `pages/500.js` file. Check out the documentation for 500 page to learn more on how to create it. During development, this file will not be used and the development error overlay will be shown instead.
## Edge Cases
### Caching with Server-Side Rendering (SSR)
You can use caching headers (`Cache-Control`) inside `getServerSideProps` to cache dynamic responses. For example, using `stale-while-revalidate`.
```
// This value is considered fresh for ten seconds (s-maxage=10).
// If a request is repeated within the next 10 seconds, the previously
// cached value will still be fresh. If the request is repeated before 59 seconds,
// the cached value will be stale but still render (stale-while-revalidate=59).
//
// In the background, a revalidation request will be made to populate the cache
// with a fresh value. If you refresh the page, you will see the new value.
exportasyncfunctiongetServerSideProps({ req, res }) {
res.setHeader(
'Cache-Control',
'public, s-maxage=10, stale-while-revalidate=59'
 )
return {
  props: {},
 }
}
```

However, before reaching for `cache-control`, we recommend seeing if `getStaticProps` with ISR is a better fit for your use case.
Was this helpful?
supported.
Send
