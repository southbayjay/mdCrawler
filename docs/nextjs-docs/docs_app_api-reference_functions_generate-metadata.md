Menu
Using App Router
Features available in /app
Using Latest Version
15.2.0
Using App Router
Features available in /app
Using Latest Version
15.2.0
API ReferenceFunctionsgenerateMetadata
# generateMetadata
This page covers all **Config-based Metadata** options with `generateMetadata` and the static metadata object.
layout.tsx | page.tsx
TypeScript
JavaScriptTypeScript
```
importtype { Metadata } from'next'
// either Static metadata
exportconstmetadata:Metadata= {
 title:'...',
}
// or Dynamic metadata
exportasyncfunctiongenerateMetadata({ params }) {
return {
  title:'...',
 }
}
```

> **Good to know** :
>   * The `metadata` object and `generateMetadata` function exports are **only supported in Server Components**.
>   * You cannot export both the `metadata` object and `generateMetadata` function from the same route segment.
> 

## The `metadata` object
To define static metadata, export a `Metadata` object from a `layout.js` or `page.js` file.
layout.tsx | page.tsx
TypeScript
JavaScriptTypeScript
```
importtype { Metadata } from'next'
exportconstmetadata:Metadata= {
 title:'...',
 description:'...',
}
exportdefaultfunctionPage() {}
```

See the Metadata Fields for a complete list of supported options.
## `generateMetadata` function
Dynamic metadata depends on **dynamic information** , such as the current route parameters, external data, or `metadata` in parent segments, can be set by exporting a `generateMetadata` function that returns a `Metadata` object.
app/products/[id]/page.tsx
TypeScript
JavaScriptTypeScript
```
importtype { Metadata, ResolvingMetadata } from'next'
typeProps= {
 params:Promise<{ id:string }>
 searchParams:Promise<{ [key:string]:string|string[] |undefined }>
}
exportasyncfunctiongenerateMetadata(
 { params, searchParams }:Props,
 parent:ResolvingMetadata
):Promise<Metadata> {
// read route params
const { id } =await params
// fetch data
constproduct=awaitfetch(`https://.../${id}`).then((res) =>res.json())
// optionally access and extend (rather than replace) parent metadata
constpreviousImages= (await parent).openGraph?.images || []
return {
  title:product.title,
  openGraph: {
   images: ['/some-specific-page-image.jpg',...previousImages],
  },
 }
}
exportdefaultfunctionPage({ params, searchParams }:Props) {}
```

### Parameters
`generateMetadata` function accepts the following parameters:
  * `props` - An object containing the parameters of the current route:
    * `params` - An object containing the dynamic route parameters object from the root segment down to the segment `generateMetadata` is called from. Examples:
Route| URL| `params`  
---|---|---  
`app/shop/[slug]/page.js`| `/shop/1`| `{ slug: '1' }`  
`app/shop/[tag]/[item]/page.js`| `/shop/1/2`| `{ tag: '1', item: '2' }`  
`app/shop/[...slug]/page.js`| `/shop/1/2`| `{ slug: ['1', '2'] }`  
    * `searchParams` - An object containing the current URL's search params. Examples:
URL| `searchParams`  
---|---  
`/shop?a=1`| `{ a: '1' }`  
`/shop?a=1&b=2`| `{ a: '1', b: '2' }`  
`/shop?a=1&a=2`| `{ a: ['1', '2'] }`  
  * `parent` - A promise of the resolved metadata from parent route segments.


### Returns
`generateMetadata` should return a `Metadata` object containing one or more metadata fields.
> **Good to know** :
>   * If metadata doesn't depend on runtime information, it should be defined using the static `metadata` object rather than `generateMetadata`.
>   * `fetch` requests are automatically memoized for the same data across `generateMetadata`, `generateStaticParams`, Layouts, Pages, and Server Components. React `cache` can be used if `fetch` is unavailable.
>   * `searchParams` are only available in `page.js` segments.
>   * The `redirect()` and `notFound()` Next.js methods can also be used inside `generateMetadata`.
> 

## Metadata Fields
### `title`
The `title` attribute is used to set the title of the document. It can be defined as a simple string or an optional template object.
#### String
layout.js | page.js
```
exportconstmetadata= {
 title:'Next.js',
}
```

<head> output
```
<title>Next.js</title>
```

#### Template object
app/layout.tsx
TypeScript
JavaScriptTypeScript
```
importtype { Metadata } from'next'
exportconstmetadata:Metadata= {
 title: {
  template:'...',
  default:'...',
  absolute:'...',
 },
}
```

##### Default
`title.default` can be used to provide a **fallback title** to child route segments that don't define a `title`.
app/layout.tsx
```
importtype { Metadata } from'next'
exportconstmetadata:Metadata= {
 title: {
  default:'Acme',
 },
}
```

app/about/page.tsx
```
importtype { Metadata } from'next'
exportconstmetadata:Metadata= {}
// Output: <title>Acme</title>
```

##### Template
`title.template` can be used to add a prefix or a suffix to `titles` defined in **child** route segments.
app/layout.tsx
TypeScript
JavaScriptTypeScript
```
importtype { Metadata } from'next'
exportconstmetadata:Metadata= {
 title: {
  template:'%s | Acme',
  default:'Acme',// a default is required when creating a template
 },
}
```

app/about/page.tsx
TypeScript
JavaScriptTypeScript
```
importtype { Metadata } from'next'
exportconstmetadata:Metadata= {
 title:'About',
}
// Output: <title>About | Acme</title>
```

> **Good to know** :
>   * `title.template` applies to **child** route segments and **not** the segment it's defined in. This means:
>     * `title.default` is **required** when you add a `title.template`.
>     * `title.template` defined in `layout.js` will not apply to a `title` defined in a `page.js` of the same route segment.
>     * `title.template` defined in `page.js` has no effect because a page is always the terminating segment (it doesn't have any children route segments).
>   * `title.template` has **no effect** if a route has not defined a `title` or `title.default`.
> 

##### Absolute
`title.absolute` can be used to provide a title that **ignores** `title.template` set in parent segments.
app/layout.tsx
TypeScript
JavaScriptTypeScript
```
importtype { Metadata } from'next'
exportconstmetadata:Metadata= {
 title: {
  template:'%s | Acme',
 },
}
```

app/about/page.tsx
TypeScript
JavaScriptTypeScript
```
importtype { Metadata } from'next'
exportconstmetadata:Metadata= {
 title: {
  absolute:'About',
 },
}
// Output: <title>About</title>
```

> **Good to know** :
>   * `layout.js`
>     * `title` (string) and `title.default` define the default title for child segments (that do not define their own `title`). It will augment `title.template` from the closest parent segment if it exists.
>     * `title.absolute` defines the default title for child segments. It ignores `title.template` from parent segments.
>     * `title.template` defines a new title template for child segments.
>   * `page.js`
>     * If a page does not define its own title the closest parents resolved title will be used.
>     * `title` (string) defines the routes title. It will augment `title.template` from the closest parent segment if it exists.
>     * `title.absolute` defines the route title. It ignores `title.template` from parent segments.
>     * `title.template` has no effect in `page.js` because a page is always the terminating segment of a route.
> 

### `description`
layout.js | page.js
```
exportconstmetadata= {
 description:'The React Framework for the Web',
}
```

<head> output
```
<metaname="description"content="The React Framework for the Web" />
```

### Basic Fields
layout.js | page.js
```
exportconstmetadata= {
 generator:'Next.js',
 applicationName:'Next.js',
 referrer:'origin-when-cross-origin',
 keywords: ['Next.js','React','JavaScript'],
 authors: [{ name:'Seb' }, { name:'Josh', url:'https://nextjs.org' }],
 creator:'Jiachi Liu',
 publisher:'Sebastian Markbåge',
 formatDetection: {
  email:false,
  address:false,
  telephone:false,
 },
}
```

<head> output
```
<metaname="application-name"content="Next.js" />
<metaname="author"content="Seb" />
<linkrel="author"href="https://nextjs.org" />
<metaname="author"content="Josh" />
<metaname="generator"content="Next.js" />
<metaname="keywords"content="Next.js,React,JavaScript" />
<metaname="referrer"content="origin-when-cross-origin" />
<metaname="color-scheme"content="dark" />
<metaname="creator"content="Jiachi Liu" />
<metaname="publisher"content="Sebastian Markbåge" />
<metaname="format-detection"content="telephone=no, address=no, email=no" />
```

### `metadataBase`
`metadataBase` is a convenience option to set a base URL prefix for `metadata` fields that require a fully qualified URL.
  * `metadataBase` allows URL-based `metadata` fields defined in the **current route segment and below** to use a **relative path** instead of an otherwise required absolute URL.
  * The field's relative path will be composed with `metadataBase` to form a fully qualified URL.
  * If not configured, `metadataBase` is **automatically populated** with a default value.


layout.js | page.js
```
exportconstmetadata= {
 metadataBase:newURL('https://acme.com'),
 alternates: {
  canonical:'/',
  languages: {
'en-US':'/en-US',
'de-DE':'/de-DE',
  },
 },
 openGraph: {
  images:'/og-image.png',
 },
}
```

<head> output
```
<linkrel="canonical"href="https://acme.com" />
<linkrel="alternate"hreflang="en-US"href="https://acme.com/en-US" />
<linkrel="alternate"hreflang="de-DE"href="https://acme.com/de-DE" />
<metaproperty="og:image"content="https://acme.com/og-image.png" />
```

> **Good to know** :
>   * `metadataBase` is typically set in root `app/layout.js` to apply to URL-based `metadata` fields across all routes.
>   * All URL-based `metadata` fields that require absolute URLs can be configured with a `metadataBase` option.
>   * `metadataBase` can contain a subdomain e.g. `https://app.acme.com` or base path e.g. `https://acme.com/start/from/here`
>   * If a `metadata` field provides an absolute URL, `metadataBase` will be ignored.
>   * Using a relative path in a URL-based `metadata` field without configuring a `metadataBase` will cause a build error.
>   * Next.js will normalize duplicate slashes between `metadataBase` (e.g. `https://acme.com/`) and a relative field (e.g. `/path`) to a single slash (e.g. `https://acme.com/path`)
> 

#### Default value
If not configured, `metadataBase` has a **default value**.
> On Vercel:
>   * For production deployments, `VERCEL_PROJECT_PRODUCTION_URL` will be used.
>   * For preview deployments, `VERCEL_BRANCH_URL` will take priority, and fallback to `VERCEL_URL` if it's not present.
> 

> If these values are present they will be used as the **default value** of `metadataBase`, otherwise it falls back to `http://localhost:${process.env.PORT || 3000}`. This allows Open Graph images to work on both local build and Vercel preview and production deployments. When overriding the default, we recommend using environment variables to compute the URL. This allows configuring a URL for local development, staging, and production environments.
> See more details about these environment variables in the System Environment Variables docs.
#### URL Composition
URL composition favors developer intent over default directory traversal semantics.
  * Trailing slashes between `metadataBase` and `metadata` fields are normalized.
  * An "absolute" path in a `metadata` field (that typically would replace the whole URL path) is treated as a "relative" path (starting from the end of `metadataBase`).


For example, given the following `metadataBase`:
app/layout.tsx
TypeScript
JavaScriptTypeScript
```
importtype { Metadata } from'next'
exportconstmetadata:Metadata= {
 metadataBase:newURL('https://acme.com'),
}
```

Any `metadata` fields that inherit the above `metadataBase` and set their own value will be resolved as follows:
`metadata` field| Resolved URL  
---|---  
`/`| `https://acme.com`  
`./`| `https://acme.com`  
`payments`| `https://acme.com/payments`  
`/payments`| `https://acme.com/payments`  
`./payments`| `https://acme.com/payments`  
`../payments`| `https://acme.com/payments`  
`https://beta.acme.com/payments`| `https://beta.acme.com/payments`  
### `openGraph`
layout.js | page.js
```
exportconstmetadata= {
 openGraph: {
  title:'Next.js',
  description:'The React Framework for the Web',
  url:'https://nextjs.org',
  siteName:'Next.js',
  images: [
   {
    url:'https://nextjs.org/og.png',// Must be an absolute URL
    width:800,
    height:600,
   },
   {
    url:'https://nextjs.org/og-alt.png',// Must be an absolute URL
    width:1800,
    height:1600,
    alt:'My custom alt',
   },
  ],
  videos: [
   {
    url:'https://nextjs.org/video.mp4',// Must be an absolute URL
    width:800,
    height:600,
   },
  ],
  audio: [
   {
    url:'https://nextjs.org/audio.mp3',// Must be an absolute URL
   },
  ],
  locale:'en_US',
  type:'website',
 },
}
```

<head> output
```
<metaproperty="og:title"content="Next.js" />
<metaproperty="og:description"content="The React Framework for the Web" />
<metaproperty="og:url"content="https://nextjs.org/" />
<metaproperty="og:site_name"content="Next.js" />
<metaproperty="og:locale"content="en_US" />
<metaproperty="og:image"content="https://nextjs.org/og.png" />
<metaproperty="og:image:width"content="800" />
<metaproperty="og:image:height"content="600" />
<metaproperty="og:image"content="https://nextjs.org/og-alt.png" />
<metaproperty="og:image:width"content="1800" />
<metaproperty="og:image:height"content="1600" />
<metaproperty="og:image:alt"content="My custom alt" />
<metaproperty="og:video"content="https://nextjs.org/video.mp4" />
<metaproperty="og:video:width"content="800" />
<metaproperty="og:video:height"content="600" />
<metaproperty="og:audio"content="https://nextjs.org/audio.mp3" />
<metaproperty="og:type"content="website" />
```

layout.js | page.js
```
exportconstmetadata= {
 openGraph: {
  title:'Next.js',
  description:'The React Framework for the Web',
  type:'article',
  publishedTime:'2023-01-01T00:00:00.000Z',
  authors: ['Seb','Josh'],
 },
}
```

<head> output
```
<metaproperty="og:title"content="Next.js" />
<metaproperty="og:description"content="The React Framework for the Web" />
<metaproperty="og:type"content="article" />
<metaproperty="article:published_time"content="2023-01-01T00:00:00.000Z" />
<metaproperty="article:author"content="Seb" />
<metaproperty="article:author"content="Josh" />
```

> **Good to know** :
>   * It may be more convenient to use the file-based Metadata API for Open Graph images. Rather than having to sync the config export with actual files, the file-based API will automatically generate the correct metadata for you.
> 

### `robots`
layout.tsx | page.tsx
```
importtype { Metadata } from'next'
exportconstmetadata:Metadata= {
 robots: {
  index:true,
  follow:true,
  nocache:false,
  googleBot: {
   index:true,
   follow:true,
   noimageindex:false,
'max-video-preview':-1,
'max-image-preview':'large',
'max-snippet':-1,
  },
 },
}
```

<head> output
```
<metaname="robots"content="index, follow" />
<meta
name="googlebot"
content="index, follow, max-video-preview:-1, max-image-preview:large, max-snippet:-1"
/>
```

### `icons`
> **Good to know** : We recommend using the file-based Metadata API for icons where possible. Rather than having to sync the config export with actual files, the file-based API will automatically generate the correct metadata for you.
layout.js | page.js
```
exportconstmetadata= {
 icons: {
  icon:'/icon.png',
  shortcut:'/shortcut-icon.png',
  apple:'/apple-icon.png',
  other: {
   rel:'apple-touch-icon-precomposed',
   url:'/apple-touch-icon-precomposed.png',
  },
 },
}
```

<head> output
```
<linkrel="shortcut icon"href="/shortcut-icon.png" />
<linkrel="icon"href="/icon.png" />
<linkrel="apple-touch-icon"href="/apple-icon.png" />
<link
rel="apple-touch-icon-precomposed"
href="/apple-touch-icon-precomposed.png"
/>
```

layout.js | page.js
```
exportconstmetadata= {
 icons: {
  icon: [
   { url:'/icon.png' },
newURL('/icon.png','https://example.com'),
   { url:'/icon-dark.png', media:'(prefers-color-scheme: dark)' },
  ],
  shortcut: ['/shortcut-icon.png'],
  apple: [
   { url:'/apple-icon.png' },
   { url:'/apple-icon-x3.png', sizes:'180x180', type:'image/png' },
  ],
  other: [
   {
    rel:'apple-touch-icon-precomposed',
    url:'/apple-touch-icon-precomposed.png',
   },
  ],
 },
}
```

<head> output
```
<linkrel="shortcut icon"href="/shortcut-icon.png" />
<linkrel="icon"href="/icon.png" />
<linkrel="icon"href="https://example.com/icon.png" />
<linkrel="icon"href="/icon-dark.png"media="(prefers-color-scheme: dark)" />
<linkrel="apple-touch-icon"href="/apple-icon.png" />
<link
rel="apple-touch-icon-precomposed"
href="/apple-touch-icon-precomposed.png"
/>
<link
rel="apple-touch-icon"
href="/apple-icon-x3.png"
sizes="180x180"
type="image/png"
/>
```

> **Good to know** : The `msapplication-*` meta tags are no longer supported in Chromium builds of Microsoft Edge, and thus no longer needed.
### `themeColor`
> **Deprecated** : The `themeColor` option in `metadata` is deprecated as of Next.js 14. Please use the `viewport` configuration instead.
### `colorScheme`
> **Deprecated** : The `colorScheme` option in `metadata` is deprecated as of Next.js 14. Please use the `viewport` configuration instead.
### `manifest`
A web application manifest, as defined in the Web Application Manifest specification.
layout.js | page.js
```
exportconstmetadata= {
 manifest:'https://nextjs.org/manifest.json',
}
```

<head> output
```
<linkrel="manifest"href="https://nextjs.org/manifest.json" />
```

### `twitter`
The Twitter specification is (surprisingly) used for more than just X (formerly known as Twitter).
Learn more about the Twitter Card markup reference.
layout.js | page.js
```
exportconstmetadata= {
 twitter: {
  card:'summary_large_image',
  title:'Next.js',
  description:'The React Framework for the Web',
  siteId:'1467726470533754880',
  creator:'@nextjs',
  creatorId:'1467726470533754880',
  images: ['https://nextjs.org/og.png'],// Must be an absolute URL
 },
}
```

<head> output
```
<metaname="twitter:card"content="summary_large_image" />
<metaname="twitter:site:id"content="1467726470533754880" />
<metaname="twitter:creator"content="@nextjs" />
<metaname="twitter:creator:id"content="1467726470533754880" />
<metaname="twitter:title"content="Next.js" />
<metaname="twitter:description"content="The React Framework for the Web" />
<metaname="twitter:image"content="https://nextjs.org/og.png" />
```

layout.js | page.js
```
exportconstmetadata= {
 twitter: {
  card:'app',
  title:'Next.js',
  description:'The React Framework for the Web',
  siteId:'1467726470533754880',
  creator:'@nextjs',
  creatorId:'1467726470533754880',
  images: {
   url:'https://nextjs.org/og.png',
   alt:'Next.js Logo',
  },
  app: {
   name:'twitter_app',
   id: {
    iphone:'twitter_app://iphone',
    ipad:'twitter_app://ipad',
    googleplay:'twitter_app://googleplay',
   },
   url: {
    iphone:'https://iphone_url',
    ipad:'https://ipad_url',
   },
  },
 },
}
```

<head> output
```
<metaname="twitter:site:id"content="1467726470533754880" />
<metaname="twitter:creator"content="@nextjs" />
<metaname="twitter:creator:id"content="1467726470533754880" />
<metaname="twitter:title"content="Next.js" />
<metaname="twitter:description"content="The React Framework for the Web" />
<metaname="twitter:card"content="app" />
<metaname="twitter:image"content="https://nextjs.org/og.png" />
<metaname="twitter:image:alt"content="Next.js Logo" />
<metaname="twitter:app:name:iphone"content="twitter_app" />
<metaname="twitter:app:id:iphone"content="twitter_app://iphone" />
<metaname="twitter:app:id:ipad"content="twitter_app://ipad" />
<metaname="twitter:app:id:googleplay"content="twitter_app://googleplay" />
<metaname="twitter:app:url:iphone"content="https://iphone_url" />
<metaname="twitter:app:url:ipad"content="https://ipad_url" />
<metaname="twitter:app:name:ipad"content="twitter_app" />
<metaname="twitter:app:name:googleplay"content="twitter_app" />
```

### `viewport`
> **Deprecated** : The `viewport` option in `metadata` is deprecated as of Next.js 14. Please use the `viewport` configuration instead.
### `verification`
layout.js | page.js
```
exportconstmetadata= {
 verification: {
  google:'google',
  yandex:'yandex',
  yahoo:'yahoo',
  other: {
   me: ['my-email','my-link'],
  },
 },
}
```

<head> output
```
<metaname="google-site-verification"content="google" />
<metaname="y_key"content="yahoo" />
<metaname="yandex-verification"content="yandex" />
<metaname="me"content="my-email" />
<metaname="me"content="my-link" />
```

### `appleWebApp`
layout.js | page.js
```
exportconstmetadata= {
 itunes: {
  appId:'myAppStoreID',
  appArgument:'myAppArgument',
 },
 appleWebApp: {
  title:'Apple Web App',
  statusBarStyle:'black-translucent',
  startupImage: [
'/assets/startup/apple-touch-startup-image-768x1004.png',
   {
    url:'/assets/startup/apple-touch-startup-image-1536x2008.png',
    media:'(device-width: 768px) and (device-height: 1024px)',
   },
  ],
 },
}
```

<head> output
```
<meta
name="apple-itunes-app"
content="app-id=myAppStoreID, app-argument=myAppArgument"
/>
<metaname="mobile-web-app-capable"content="yes" />
<metaname="apple-mobile-web-app-title"content="Apple Web App" />
<link
href="/assets/startup/apple-touch-startup-image-768x1004.png"
rel="apple-touch-startup-image"
/>
<link
href="/assets/startup/apple-touch-startup-image-1536x2008.png"
media="(device-width: 768px) and (device-height: 1024px)"
rel="apple-touch-startup-image"
/>
<meta
name="apple-mobile-web-app-status-bar-style"
content="black-translucent"
/>
```

### `alternates`
layout.js | page.js
```
exportconstmetadata= {
 alternates: {
  canonical:'https://nextjs.org',
  languages: {
'en-US':'https://nextjs.org/en-US',
'de-DE':'https://nextjs.org/de-DE',
  },
  media: {
'only screen and (max-width: 600px)':'https://nextjs.org/mobile',
  },
  types: {
'application/rss+xml':'https://nextjs.org/rss',
  },
 },
}
```

<head> output
```
<linkrel="canonical"href="https://nextjs.org" />
<linkrel="alternate"hreflang="en-US"href="https://nextjs.org/en-US" />
<linkrel="alternate"hreflang="de-DE"href="https://nextjs.org/de-DE" />
<link
rel="alternate"
media="only screen and (max-width: 600px)"
href="https://nextjs.org/mobile"
/>
<link
rel="alternate"
type="application/rss+xml"
href="https://nextjs.org/rss"
/>
```

### `appLinks`
layout.js | page.js
```
exportconstmetadata= {
 appLinks: {
  ios: {
   url:'https://nextjs.org/ios',
   app_store_id:'app_store_id',
  },
  android: {
   package:'com.example.android/package',
   app_name:'app_name_android',
  },
  web: {
   url:'https://nextjs.org/web',
   should_fallback:true,
  },
 },
}
```

<head> output
```
<metaproperty="al:ios:url"content="https://nextjs.org/ios" />
<metaproperty="al:ios:app_store_id"content="app_store_id" />
<metaproperty="al:android:package"content="com.example.android/package" />
<metaproperty="al:android:app_name"content="app_name_android" />
<metaproperty="al:web:url"content="https://nextjs.org/web" />
<metaproperty="al:web:should_fallback"content="true" />
```

### `archives`
Describes a collection of records, documents, or other materials of historical interest (source).
layout.js | page.js
```
exportconstmetadata= {
 archives: ['https://nextjs.org/13'],
}
```

<head> output
```
<linkrel="archives"href="https://nextjs.org/13" />
```

### `assets`
layout.js | page.js
```
exportconstmetadata= {
 assets: ['https://nextjs.org/assets'],
}
```

<head> output
```
<linkrel="assets"href="https://nextjs.org/assets" />
```

### `bookmarks`
layout.js | page.js
```
exportconstmetadata= {
 bookmarks: ['https://nextjs.org/13'],
}
```

<head> output
```
<linkrel="bookmarks"href="https://nextjs.org/13" />
```

### `category`
layout.js | page.js
```
exportconstmetadata= {
 category:'technology',
}
```

<head> output
```
<metaname="category"content="technology" />
```

### `facebook`
You can connect a Facebook app or Facebook account to you webpage for certain Facebook Social Plugins Facebook Documentation
> **Good to know** : You can specify either appId or admins, but not both.
layout.js | page.js
```
exportconstmetadata= {
 facebook: {
  appId:'12345678',
 },
}
```

<head> output
```
<metaproperty="fb:app_id"content="12345678" />
```

layout.js | page.js
```
exportconstmetadata= {
 facebook: {
  admins:'12345678',
 },
}
```

<head> output
```
<metaproperty="fb:admins"content="12345678" />
```

If you want to generate multiple fb:admins meta tags you can use array value.
layout.js | page.js
```
exportconstmetadata= {
 facebook: {
  admins: ['12345678','87654321'],
 },
}
```

<head> output
```
<metaproperty="fb:admins"content="12345678" />
<metaproperty="fb:admins"content="87654321" />
```

### `other`
All metadata options should be covered using the built-in support. However, there may be custom metadata tags specific to your site, or brand new metadata tags just released. You can use the `other` option to render any custom metadata tag.
layout.js | page.js
```
exportconstmetadata= {
 other: {
  custom:'meta',
 },
}
```

<head> output
```
<metaname="custom"content="meta" />
```

If you want to generate multiple same key meta tags you can use array value.
layout.js | page.js
```
exportconstmetadata= {
 other: {
  custom: ['meta1','meta2'],
 },
}
```

<head> output
```
<metaname="custom"content="meta1" /> <metaname="custom"content="meta2" />
```

## Unsupported Metadata
The following metadata types do not currently have built-in support. However, they can still be rendered in the layout or page itself.
Metadata| Recommendation  
---|---  
`<meta http-equiv="...">`| Use appropriate HTTP Headers via `redirect()`, Middleware, Security Headers  
`<base>`| Render the tag in the layout or page itself.  
`<noscript>`| Render the tag in the layout or page itself.  
`<style>`| Learn more about styling in Next.js.  
`<script>`| Learn more about using scripts.  
`<link rel="stylesheet" />`| `import` stylesheets directly in the layout or page itself.  
`<link rel="preload />`| Use ReactDOM preload method  
`<link rel="preconnect" />`| Use ReactDOM preconnect method  
`<link rel="dns-prefetch" />`| Use ReactDOM prefetchDNS method  
### Resource hints
The `<link>` element has a number of `rel` keywords that can be used to hint to the browser that an external resource is likely to be needed. The browser uses this information to apply preloading optimizations depending on the keyword.
While the Metadata API doesn't directly support these hints, you can use new `ReactDOM` methods to safely insert them into the `<head>` of the document.
app/preload-resources.tsx
TypeScript
JavaScriptTypeScript
```
'use client'
import ReactDOM from'react-dom'
exportfunctionPreloadResources() {
ReactDOM.preload('...', { as:'...' })
ReactDOM.preconnect('...', { crossOrigin:'...' })
ReactDOM.prefetchDNS('...')
return'...'
}
```

##### `<link rel="preload">`
Start loading a resource early in the page rendering (browser) lifecycle. MDN Docs.
```
ReactDOM.preload(href: string, options: { as: string })
```

<head> output
```
<linkrel="preload"href="..."as="..." />
```

##### `<link rel="preconnect">`
Preemptively initiate a connection to an origin. MDN Docs.
```
ReactDOM.preconnect(href: string, options?: { crossOrigin?: string })
```

<head> output
```
<linkrel="preconnect"href="..."crossorigin />
```

##### `<link rel="dns-prefetch">`
Attempt to resolve a domain name before resources get requested. MDN Docs.
```
ReactDOM.prefetchDNS(href: string)
```

<head> output
```
<linkrel="dns-prefetch"href="..." />
```

> **Good to know** :
>   * These methods are currently only supported in Client Components, which are still Server Side Rendered on initial page load.
>   * Next.js in-built features such as `next/font`, `next/image` and `next/script` automatically handle relevant resource hints.
> 

## Types
You can add type safety to your metadata by using the `Metadata` type. If you are using the built-in TypeScript plugin in your IDE, you do not need to manually add the type, but you can still explicitly add it if you want.
### `metadata` object
layout.tsx | page.tsx
```
importtype { Metadata } from'next'
exportconstmetadata:Metadata= {
 title:'Next.js',
}
```

### `generateMetadata` function
#### Regular function
layout.tsx | page.tsx
```
importtype { Metadata } from'next'
exportfunctiongenerateMetadata():Metadata {
return {
  title:'Next.js',
 }
}
```

#### Async function
layout.tsx | page.tsx
```
importtype { Metadata } from'next'
exportasyncfunctiongenerateMetadata():Promise<Metadata> {
return {
  title:'Next.js',
 }
}
```

#### With segment props
layout.tsx | page.tsx
```
importtype { Metadata } from'next'
typeProps= {
 params:Promise<{ id:string }>
 searchParams:Promise<{ [key:string]:string|string[] |undefined }>
}
exportfunctiongenerateMetadata({ params, searchParams }:Props):Metadata {
return {
  title:'Next.js',
 }
}
exportdefaultfunctionPage({ params, searchParams }:Props) {}
```

#### With parent metadata
layout.tsx | page.tsx
```
importtype { Metadata, ResolvingMetadata } from'next'
exportasyncfunctiongenerateMetadata(
 { params, searchParams }:Props,
 parent:ResolvingMetadata
):Promise<Metadata> {
return {
  title:'Next.js',
 }
}
```

#### JavaScript Projects
For JavaScript projects, you can use JSDoc to add type safety.
layout.js | page.js
```
/** @type{import("next").Metadata} */
exportconstmetadata= {
 title:'Next.js',
}
```

## Streaming metadata
Metadata returned by `generateMetadata` is streamed to the client. This allows Next.js to inject metadata into the HTML as soon as it's resolved.
Since page metadata primarily targets bots and crawlers, Next.js will stream metadata for bots that can execute JavaScript and inspect the full page DOM (e.g. `Googlebot`). However, metadata will continue blocking the render of the page for **HTML-limited** bots (e.g. `Twitterbot`) as these cannot execute JavaScript while crawling.
Next.js automatically detects the user agent of incoming requests to determine whether to serve streaming metadata or fallback to blocking metadata.
If you need to customize this list, you can define them manually using the `htmlLimitedBots` option in `next.config.js`. Next.js will ensure user agents matching this regex receive blocking metadata when requesting your web page.
next.config.ts
TypeScript
JavaScriptTypeScript
```
importtype { NextConfig } from'next'
constconfig:NextConfig= {
 htmlLimitedBots:'MySpecialBot|MyAnotherSpecialBot|SimpleCrawler',
}
exportdefault config
```

Specifying a `htmlLimitedBots` config will override the Next.js' default list, allowing you full control over what user agents should opt into this behavior. This is advanced behavior, and the default should be sufficient for most cases.
## Version History
Version| Changes  
---|---  
`v15.2.0`| Introduced streaming support to `generateMetadata`.  
`v13.2.0`| `viewport`, `themeColor`, and `colorScheme` deprecated in favor of the `viewport` configuration.  
`v13.2.0`| `metadata` and `generateMetadata` introduced.  
## Next Steps
View all the Metadata API options.
### Metadata Files
API documentation for the metadata file conventions.
### generateViewport
API Reference for the generateViewport function.
### Metadata
Use the Metadata API to define metadata in any layout or page.
Was this helpful?
supported.
Send
