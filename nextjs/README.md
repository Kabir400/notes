# Next.js App Router Notes

## 1. Installation, Running, and Building

### Install Next.js (Short Version)

To create and run a new Next.js 14 app:

```sh
npx create-next-app@latest my-app
cd my-app
npm run dev
```

To build for production:

```sh
npm run build
npm run start
```

---

## 2. Folder Structure and Purpose of Files

### Key Files and Their Uses:

- **`app/page.js`** → The main entry page of the application.
- **`app/layout.js`** → Defines the layout shared across all pages (e.g., headers, footers).
- **`app/template.js`** →Similar to layout.js, but it re-renders on every navigation, ensuring a fresh instance each time.
- **`app/not-found.js`** → Displays a custom 404 page for missing routes.
- **`app/loading.js`** → Shows a loading state while a page or component is fetching data.
- **`app/globals.css`** → If you add this in the layout.js the css will be applicable in the entire page.
- **`module.css`** → You can use module.css like react in next js.

#### `layout.js`

##### Overview

`layout.js` is **used to define a shared UI structure** for a route segment and its children.

- **It receives `page.js` as a children props**

##### Example

```jsx
export default function Layout({ children }) {
  return (
    <div>
      <nav>Navbar</nav>
      {children}
    </div>
  );
}
```

**Note:** Some browser extension might cause `hidration error` in the development mode, to solve this issue you can use `<body suppressHydrationWarning> </body>` in the root layout.

#### `error.js`

##### Overview

`error.js` handles errors **within a specific route segment** in Next.js App Router. It acts as a React error boundary and catches **client-side errors** only in its segment.

##### Key Points

- Placed inside a route folder.
- Cannot catch errors in `layout.js` or `template.js` (use `global-error.js` for that).
- Must be a **Client Component** (`'use client'`).
- Provides a `reset` function for error recovery.

##### Example

```jsx
"use client";
import { useRouter } from "next/navigation";
import { startTransition } from "react";

export default function Error({ error, reset }) {
  const router = useRouter();
  const reload = () => {
    startTransition(() => {
      router.refresh();
      reset();
    });
  };

  return (
    <div>
      <h1>Something went wrong!</h1>
      <p>{error.message}</p>
      <button onClick={reload}>Try Again</button>
    </div>
  );
}
```

##### Usage

- Handles errors **only within its folder**.

- Shows a friendly error UI instead of a crash.

- **If you use `reset()` instead of `reload()`, ensure `page.js` is a client component, or the error will persist.**

- In Client Components: `reset()` should work fine.

- In Server Components: `reset()` alone won't work properly. Use `router.refresh()` inside `startTransition()` for a full reset.

- `startTransition(() => { ... })` Marks the update as low-priority so React can keep the UI responsive.**it ensures router.refresh() and reset() don't block other UI updates.**

- `router.refresh()` **Forces Next.js to refetch and re-render the current route from the server.**

- `reset()` **Resets the error boundary state, allowing React to retry rendering the component.**

#### `global-error.js`

- You **can't catch errors for layout.js or template.js using error.js,** `so you need global-error.js`

- You have to use `<html>` and `<body>` tags in the return statement of global-error.js because **global-error.js is rendered before layout.js or template.js**, meaning it must define the full document structure.

##### Example

```jsx
export default function GlobalError({ error, reset }) {
  return (
    <html>
      <body>
        <h1>Something went wrong!</h1>
        <p>{error.message}</p>
        <button onClick={reset}>Try Again</button>
      </body>
    </html>
  );
}
```

### Route-Specific Files:

You can also add `loading.js`, `error.js`,`layout.js` , `template.js` and `not-found.js` inside any route folder. If present, they will only apply to that route.

```
app/dashboard/
│   ├── page.js       (Dashboard page)
│   ├── loading.js    (Loading state for dashboard only)
│   ├── error.js      (Error boundary for dashboard only)
│   ├── layout.js     (layout for dashboard only)
│   ├── not-found.js  (Custom 404 for dashboard only)
```

### Basic Folder Structure:

```
my-app/
├── app/
│   ├── layout.js  (Global Layout for pages)
│   ├── page.js    (Main home page)((/)->route)
│   ├── about/
│   │   ├── page.js  (About page)((/about)->route)
│   ├── error.js   (Global error handling)
│   ├── not-found.js (Global 404 page)
│   ├── loading.js  (Global loading UI)
│   ├── api/
│   │   ├── route.js (API routes)
├── public/        (Static assets like images)
├── styles/        (Global CSS files)
├── next.config.js (Next.js configuration)
├── package.json   (Dependencies & scripts)
```

---

## 3. Routing in Next.js

### 1. Normal Routing (File-based routing)

Each file inside the `app/` directory automatically becomes a route.

- `app/page.js` → Accessible at `/`
- `app/about/page.js` → Accessible at `/about`

### 2. Using `<Link href> ` for Client-side Navigation

```jsx
import Link from "next/link";
export default function Home() {
  return (
    <div>
      <Link href="/about">Go to About Page</Link>
    </div>
  );
}
```

### 3. Dynamic Routing: `[id]`

If you need to create dynamic pages based on an `id`:

```
app/product/[id]/page.js → Accessible at `/product/1`, `/product/2`, etc.
```

Example:
`server component`

```jsx
export default function ProductPage({ params, searchParams }) {
  const { id } = await params; //getting id form the params

  const query= await searchParams;
  const searchTerm=query.get("q");//getting the query (ex.?q=kabir)

  return (<div>
  <h1>Product ID: {id}</h1>
  <h1>Searched Term: {searchTerm}</h1>
  </div>);
}
```

`client component`

```jsx
"use client";

import { useParams, useSearchParams } from "next/navigation";

export default function Post() {
  const params = useParams();
  const { id } = params; //getting the id from the params

  const query = useSearchParams();
  const searchTerm = query.get("q"); //getting the query (ex.?q=kabir)

  return (
    <div>
      <h1>Product ID: {id}</h1>
      <h1>Searched Term: {searchTerm}</h1>
    </div>
  );
}
```

## Client and Server Components

- **Client Components** (`"use client"`): Use when you have to use client side features like events(click,change...etc) and hooks (useSate,useEffect...etc).
- **Server Components** (default): Rendered on the server, reducing client-side JavaScript and improving performance and seo.

## Data Fetching & Caching

### Data Fetching in Server Component

```jsx
async function UserList() {
  const users = await fetch("https://jsonplaceholder.typicode.com/users").then(
    (res) => res.json()
  );
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
export default UserList;
```

### Cache Control

- **`force-cache`** : Caches the response indefinitely until manually invalidated.
- **`no-store`**: Disables caching, ensuring fresh data on every request.
- **`revalidate: X`**: Fetches new data after X seconds while serving cached data in the meantime.

### Fetching Strategies

#### Static Site Generation (SSG)

- Pre-renders the page at build time.
- Uses `fetch()` with `force-cache` or `revalidate: false`.
- Example:
  ```js
  async function getData() {
    const res = await fetch("https://api.example.com/data", {
      cache: "force-cache",
    });
    return res.json();
  }
  ```

#### Server-Side Rendering (SSR)

- Fetches data on every request.
- Uses `fetch()` with `no-store`.
- Example:
  ```js
  async function getData() {
    const res = await fetch("https://api.example.com/data", {
      cache: "no-store",
    });
    return res.json();
  }
  ```

#### Incremental Static Regeneration (ISR)

- Allows static pages to update after a certain interval without rebuilding the whole site.
- Uses `fetch()` with `revalidate: X`.
- Example:
  ```js
  async function getData() {
    const res = await fetch("https://api.example.com/data", {
      next: { revalidate: 60 },
    });
    return res.json();
  }
  ```

#### Manual Revalidation

- Manually triggers a cache revalidation via API routes.
- Example API route:(route.js)
  ```js
  import { revalidatePath } from "next/cache";
  export async function POST(req) {
    revalidatePath("/page");
    return Response.json({ revalidated: true });
  }
  ```

#### Generate Static Params

- Used to generate static paths at build time for dynamic routes.
- Example:(page.js)
  ```js
  export async function generateStaticParams() {
    const data = await fetch("https://api.example.com/paths").then((res) =>
      res.json()
    );
    return data.map((item) => ({ id: item.id }));
    //Ex:[{id:101},{id:102}] , it will only cache for id 101 and 102
  }
  ```

#### Export `dynamicParams`

- Setting `dynamicParams to true` allows Next.js to **dynamically render pages for params not returned by generateStaticParams**.
- Conversely, `setting it to false` will **result in a 404 error** for any params not pre-generated.

```jsx
export const dynamicParams = true;
```

#### Export `revalidate`

- Used to set global revalidation time for a page.
- Example:(page.js)
  ```js
  export const revalidate = 60; // Revalidates every 60 seconds
  ```

**CSR**: Use `use client` for this (ex:React)

##### Additional Notes

`Caching Fetch Requests in the Same Lifecycle`

- When using `multiple fetch() calls` in a single function or component lifecycle, Next.js automatically `caches the requests to the same URL`, reducing redundant network calls. This ensures efficiency without requiring manual memoization.

# Parallel Data Fetching in Next.js

## Introduction

Parallel data fetching improves performance by retrieving multiple API requests simultaneously rather than sequentially. This reduces loading time, especially for pages requiring multiple data sources.

## Using `Promise.all`

You can use `Promise.all` to fetch multiple data sources in parallel:

```js
const [products, comments] = await Promise.all([
  fetch("/api/products").then((res) => res.json()),
  fetch("/api/comments").then((res) => res.json()),
]);
```

However, **with `Promise.all`, the response is only available when all requests are resolved**. For example, if `products` are fetched early but `comments` take longer, the user won’t see the products immediately—they’ll have to wait for both requests to complete.

## Optimized Approach: Separate Components

To avoid blocking UI rendering, fetch data separately in different components:

```jsx
const Products = async () => {
  const products = await fetch("/api/products").then((res) => res.json());
  return <ProductList data={products} />;
};

const Comments = async () => {
  const comments = await fetch("/api/comments").then((res) => res.json());
  return <CommentList data={comments} />;
};

export default function Page() {
  return (
    <div>
      <Suspense fallback={<LoadingProducts />}>
        <Products />
      </Suspense>
      <Suspense fallback={<LoadingComments />}>
        <Comments />
      </Suspense>
    </div>
  );
}
```

### Benefits:

- **Faster Rendering**: Products appear as soon as they are available, while comments continue loading.
- **Better User Experience**: Users don’t have to wait for all data sources to load before seeing content.
- **Scalability**: Independent components make the codebase more modular and maintainable.

## Server Component inside Client Component

In Next.js, you **cannot directly use a server component inside a client component**. Instead, you must pass the server component as a **prop** to the client component. If you attempt to use a server component inside a client component, **Next.js will convert the entire children to a client component**, losing server benefits.

## Context API Example

### Root Layout

```jsx
import ThemeProvider from "./_components/ThemeProvider";

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```

### Theme Provider

```jsx
"use client";

import { createContext, useState } from "react";

export const ThemeContext = createContext("light");

function ThemeProvider({ children }) {
  const Theme = useState("light");
  return (
    <>
      <ThemeContext.Provider value={Theme}>{children}</ThemeContext.Provider>
    </>
  );
}

export default ThemeProvider;
```

### Theme Consumer (About Page)

```jsx
"use client";
import { useContext } from "react";
import { ThemeContext } from "../_components/ThemeProvider";

function page() {
  const [theme, setTheme] = useContext(ThemeContext);

  return (
    <div>
      <h1>About</h1>
      <p>Current theme: {theme}</p>
    </div>
  );
}

export default page;
```

# Next.js Optimizations

## Default Import Alias

Next.js allows setting up import aliases using the `jsconfig.json` or `tsconfig.json` file to simplify imports and avoid long relative paths. Example:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@components/*": ["./components/*"],
      "@utils/*": ["./utils/*"],
      "@/*": ["./*"]
    }
  }
}
```

Usage in a file:

```jsx
import Button from "@components/Button";
import { formatDate } from "@utils/date";
import Header from "@/Header";
```

## Image Optimization

Next.js provides built-in image optimization using the `next/image` component. It automatically optimizes images for better performance.

Example usage:

```jsx
import Image from "next/image";

function Profile() {
  return (
    <Image
      src="/profile.jpg"
      alt="Profile"
      width={200}
      height={200}
      quality={80}
      placeholder="blur"
      blurDataURL="/placeholder.png"
    />
  );
}
```

### Key Features:

- **Automatic Resizing & Optimization**: Next.js automatically resizes images for different screen sizes and optimizes them.
- **Lazy Loading**: Images load only when they are visible in the viewport.
- **Modern Formats**: Supports WebP, AVIF, and other efficient formats.
- **Placeholders**: You can use `placeholder="blur"` with a `blurDataURL` to show a low-quality preview while loading.
- **Remote Images**: By default, Next.js restricts external image sources for security reasons. You need to whitelist the domains in `next.config.js`:

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "example.com",
      },
      {
        protocol: "https",
        hostname: "*.mycdn.com", // Supports wildcard subdomains
      },
    ],
  },
};
module.exports = nextConfig;
```

## Font Optimization

Next.js offers built-in font optimization using the `next/font` module.

### Google Fonts Example

```jsx
import { Inter } from "next/font/google";

const inter = Inter({ subsets: ["latin"] });

export default function Home() {
  return <h1 className={inter.className}>Optimized Font</h1>;
}
```

### Local Fonts Example

```jsx
import localFont from "next/font/local";

const myFont = localFont({
  src: "../fonts/MyCustomFont.woff2",
  display: "swap",
});

export default function Home() {
  return <h1 className={myFont.className}>Local Font</h1>;
}

//When you use display: "swap", it specifies that if the custom font has not loaded yet, the browser will first display the fallback font and then swap it with the custom font once it's available.
```

**Benefits:**

- Reduces layout shifts (FOUT/FOIT issues)
- Automatically subsets fonts
- Loads fonts efficiently
- Supports both Google and local fonts

## Static & Dynamic Metadata for SEO

### Static Metadata (Using `metadata` Object)

You can define static metadata inside `layout.js` or page components:

```jsx
export const metadata = {
  title: "My Awesome Site",
  description: "This is an example site with optimized metadata.",
};
```

### Dynamic Metadata (Using `generateMetadata` with Fetch)

For dynamic metadata, use `generateMetadata` function and fetch actual data:

```jsx
export async function generateMetadata({ params }) {
  const { id } = await params;
  const res = await fetch(`https://api.example.com/articles/${id}`);
  const article = await res.json();

  return {
    title: article.title,
    description: article.description,
  };
}
```

**Benefits:**

- Improves SEO ranking
- Enhances social media previews
- Dynamically updates metadata for different pages

# More about Routing

## 1. Private Folders (`_folder`)

- Any folder prefixed with an underscore (`_folder`) is ignored by the Next.js router.
- Useful for storing utility files, helper functions, or shared layouts without exposing them as routes.
- If you `don't use page.js` file it will still be ignored by the next.js router.

### Example:

```
my-app/
├── app/
│   ├── _utils/
│   │   ├── page.js  // Not exposed as a route
│   ├── page.js        // Exposed as "/"

```

---

## 2. Route Groups (`(group)`)

- Helps organize routes without affecting the URL structure.
- Wrap a set of routes inside `()` to group them logically.

### Example:

```
my-app/
├── app/
│   ├── (dashboard)/   // Route group (does not appear in the URL)
│   │   ├── settings/
│   │   │   ├── page.js  // URL: "/settings"
│   │   ├── profile/
│   │   │   ├── page.js  // URL: "/profile"

```

Even though routes are inside `(dashboard)`, the group name does not appear in the URL.

---

## 3. Redirecting to a Different Page

### Client-Side Redirection (Using `useRouter`)

Use `next/navigation`'s `useRouter` for client-side navigation.

```tsx
"use client";
import { useRouter } from "next/navigation";

export default function Home() {
  const router = useRouter();

  const redirectToDashboard = () => {
    router.push("/dashboard");
  };

  return <button onClick={redirectToDashboard}>Go to Dashboard</button>;
}
```

- `<Link replace href="path"> ` and `router.replace(path)` Works like push, but prevents adding a new entry to the history stack, meaning users can't go back to the previous page with the browser back button.
- `router.back()` Navigates back one page in the history stack.

### Server-Side Redirection (`redirect`)

Use `redirect()` from `next/navigation` inside a Server Component.

```tsx
import { redirect } from "next/navigation";

export default function Page() {
  redirect("/dashboard"); // Redirects immediately
}
```

---

## 4. Catch-All Routes (`[...id]`)

- Matches multiple dynamic segments and captures them as an array.

### Example:

```
my-app/
├── app/
│   ├── blog/
│   │   ├── [...id]/
│   │   │   ├── page.js  // URL: "/blog/*" (Catch-all route)

```

- `/blog/nextjs/routing` → `id = ["nextjs", "routing"]`
- `/blog/2025/02/guide` → `id = ["2025", "02", "guide"]`

---

## 5. Redirecting to the Not Found Page

### Client-Side (`useRouter`)

```tsx
"use client";
import { useRouter } from "next/navigation";

export default function Page() {
  const router = useRouter();

  return (
    <button onClick={() => router.push("/not-found")}>Go to Not Found</button>
  );
}
```

### Server-Side (`notFound()`)

```tsx
import { notFound } from "next/navigation";

export default function Page() {
  notFound(); // Immediately renders the 404 page
}
```

---

## 6. Parallel Routing

- Used for rendering multiple pages side by side in a layout.
- Defined using slot folders (`@slot`).

### Example:

```
my-app/
├── app/
│   ├── layout.js    // Global layout for all pages
│   ├── @dashboard/  // Parallel route
│   │   ├── page.js  // Accessible in a parallel slot
│   ├── @settings/   // Parallel route
│   │   ├── page.js  // Accessible in a parallel slot

```

#### `layout.js`

```tsx
export default function Layout({ dashboard, settings }) {
  return (
    <div>
      <div>{dashboard}</div>
      <div>{settings}</div>
    </div>
  );
}
```

- `dashboard` and `settings` render in parallel inside the layout.

---

#### 7. `default.js` For Parallel Routing:

When no slot is matched for the specific route, the default slot is used. Without default.js file it will give 404 error.

```
my-app/
├── app/
│   ├── layout.js
│   ├── @dashboard/
│   │   ├── page.js
│   │   ├── default.js   // This fallback slot when no slot is matched
│   ├── @settings/
│   │   ├── default.js   // fallback slot
    │── default.js    //fallback slot for children props in layout
```

## 8. Intercepting Routes

Intercepting routes in Next.js allows you to override the normal navigation flow by displaying a different page instead of the default one when navigating. This is useful for modal routes, inline content previews, and seamless user experiences.

## Prefix Syntax:

It works exactly like when you import something form other directories in your app.

- `(..)`: One level up. (like: `../`)
- `(.)`: Current Directory (like: `./`)
- `(...)`: Intercepts a route globally (from the app root).
- `(..)(..)`: Two level up. (like: `../../`)

## Example:

```plaintext
app/
 ├── blog/
 │   ├── page.js  /Normal blog list
 │   ├── layout.js
 │   ├── [id]/
 │   │   ├── page.js  // Default post page
 ├── (..)blog/
 │   ├── [id]/
 │   │   ├── page.js  // Intercepts blog/[id]
 │       ├── layout.js //layout for intercepting routes
```

- If a user navigates to `/blog/123`, the normal page renders.
- If they navigate from within `/blog`, the intercepted modal appears instead.

This technique enhances UX by reducing unnecessary full-page reloads while maintaining deep linking.

## Third-Party Packages

- Third-party packages are starting to add the `"use client"` directive to components that need client-side features, making it clear where they should run.
- Many npm packages haven't made this transiton yet.
- This means they work fine in the client components, but might break in the server components.

**To avoid this you can use "use client" directive where the library is used in your code.**

## `client-only` and `server-only` code:

Some codes are intended to run only on the client-side or server-side.To esure that you can use third party packages like `client-only` and `server-only`.

```jsx
import "client-only";

export const clientSideFunction = () => {
  console.log("use window object , local storage...etc.");
  return "client result";
};
```

Now if you use this code in the server side you will get build time error.
