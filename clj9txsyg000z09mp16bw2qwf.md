---
title: "Using HttpOnly Cookie for Session Management in Next.js"
datePublished: Sat Jun 24 2023 09:59:52 GMT+0000 (Coordinated Universal Time)
cuid: clj9txsyg000z09mp16bw2qwf
slug: using-httponly-cookie-for-session-management-in-nextjs
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/nvYIrRZAFgg/upload/c36555d80915f0a1b2d9fc62278354f0.jpeg
tags: cookies, authentication, nextjs, session-management, cookie-based-authentication

---

### 👋 Hello there!

Are you building a Next.js project, and then wondering **how to keep your user logged in after getting an access token from your backend server** 🤔 ?

And... are you thinking that storing your user's session in browser storage (including `localStorage`, `sessionStorage` or client `cookie`) is not secure enough for your app?!

There is a more secure way we can do this with Next.js by storing the user's session in `HttpOnly` cookie.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">The <code>HttpOnly</code> cookie is not accessible by the client script. <em>Learn more at</em> <a target="_blank" rel="noopener noreferrer nofollow" href="https://owasp.org/www-community/HttpOnly" style="pointer-events: none">https://owasp.org/www-community/HttpOnly</a></div>
</div>

Okay now, we're convinced that it is the way to go for. Let's define what we need:

1. Upon the user logging in, we store the user's session in `HttpOnly` cookie
    
2. Getting user's information from the session
    
3. Upon the user logging out, we clear the session
    

Great 🎉, we have tackled the `why doing it` and `what to do` parts.

Let's get into the fun part which is `how to do` 🚀

# Project Setup

```bash
> npx create-next-app@latest
```

Let's create a Next.js project, and below are my chosen options.

```bash
What is your project named? >> nextjs-explore
Would you like to use TypeScript with this project? >> Yes
Would you like to use ESLint with this project? >> Yes
Would you like to use Tailwind CSS with this project? >> Yes
Would you like to use `src/` directory with this project? >> Yes
Use App Router (recommended)? >> No
Would you like to customize the default import alias? >> No
```

# Library Installation

To accommodate our needs, we will be using this NPM package [`iron-session`](https://www.npmjs.com/package/iron-session)

```bash
> yarn add iron-session
```

# Implementations

### (1). Config session options

Create a config file `session.ts` in the directory `src/config`

```typescript
/** File: src/config/session.ts */

import type { IronSessionOptions } from "iron-session";

export const sessionIronOptions: IronSessionOptions = {
  cookieName: "nextjs_explore_auth",
  password: process.env.SESSION_IRON_PASSWORD || "SESSION_PWD",
  cookieOptions: {
    secure: process.env.NODE_ENV === "production",
  },
};
```

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">In the above config file, we have defined the cookie's name, and password to secure the cookie and enable the <code>secure</code> option in the production build.</div>
</div><div data-node-type="callout">
<div data-node-type="callout-emoji">❗</div>
<div data-node-type="callout-text"><code>password</code> for the cookie here must be at least 32 characters long, otherwise, it will throw an error.</div>
</div>

Then in the above same file, we can define an interface for our session data *(because we love TypeScript ❤️‍🔥)*

Create an `interface` for users in `src/types/auth.ts`

```typescript
/** File: src/types/auth.ts */

export interface IUser {
  id: number;
  username: string;
  fullName: string;
  avatarUrl?: string;
}
```

```typescript
/** File: src/config/session.ts */
import { IUser } from "@/types/auth";

/** ... */

declare module "iron-session" {
  interface IronSessionData {
    user?: IUser;
  }
}
```

### (2). Create an API-Route to store the user's session

In the directory `src/pages/api`, create a new directory named `session`. Then create a new file for `store.ts`

```typescript
/** File: src/pages/api/session/store.ts */

import type { NextApiRequest, NextApiResponse } from "next";
import { withIronSessionApiRoute } from "iron-session/next";

import { sessionIronOptions } from "@/configs/session";

const storeSession = async (req: NextApiRequest, res: NextApiResponse) => {
  if (req.method === "POST") {
    req.session.user = { ...req.body };
    await req.session.save();
  }

  res.send({});
};

export default withIronSessionApiRoute(storeSession, sessionIronOptions);
```

In this API route, we've received info from the request body (defined in the above session interface `IronSessionData`, then set into session, and then save the session.

Notice, we have used `withIronSessionApiRoute` to store the session in the cookie with above defined options.

### (3) Create an API-Route to retrieve the user's session

Create a new file `index.ts` in `src/pages/api/session`

```typescript
/** File: src/pages/api/session/index.ts */

import type { NextApiRequest, NextApiResponse } from "next";
import { withIronSessionApiRoute } from "iron-session/next";

import { IUser } from "@/types/auth";
import { sessionIronOptions } from "@/configs/session";

export interface ISession {
  user?: IUser;
  isLogin: boolean;
}

const getSession = async (
  req: NextApiRequest,
  res: NextApiResponse<ISession>
) => {
  if (req.session.user) {
    res.json({
      ...req.session.user,
      isLogin: true,
    });
  } else {
    res.json({
      isLogin: false,
    });
  }
};

export default withIronSessionApiRoute(getSession, sessionIronOptions);
```

In this API route, we return the user from the session object along with `isLogin` property.

### (4) Create an API-Route to destroy the user's session

Create a new file `destroy.ts` in `src/pages/api/session`

```typescript
/** File: src/pages/api/session/destroy.ts */

import type { NextApiRequest, NextApiResponse } from "next";
import { withIronSessionApiRoute } from "iron-session/next";

import { sessionIronOptions } from "@/configs/session";

const destroySession = async (req: NextApiRequest, res: NextApiResponse) => {
  if (req.method === "POST") {
    req.session.destroy();
  }
  res.send({});
};

export default withIronSessionApiRoute(destroySession, sessionIronOptions);
```

### (5) Create a login page

In directory `pages`, create a new file `login.tsx`

```typescript
/** File: src/pages/login.tsx */
import { useRouter } from "next/router";

import { IUser } from "@/types/auth";

const setSession = async (data: IUser) =>
  await fetch("/api/session/store", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(data),
  });

const LoginPage = () => {
  const router = useRouter();

  const handleLogin = async () => {
    /** At here, we call to api login at your backend,
     * and the api could respond some info about the user upon success
     * so then we could set the data to session
     * */

    const user: IUser = {
      id: 23,
      username: "tvy",
      fullName: "tvydev",
      avatarUrl: "some_url.com",
    };

    await setSession(user);

    router.replace("/");
  };

  return (
    <>
      <h1>Login Page</h1>
      <button type="button" onClick={handleLogin}>
        Login
      </button>
    </>
  );
};

export default LoginPage;
```

For simplicity and focus on the session management thing, we mock things up! 😆

When a user clicks on the button "Login", we will call to API login at our backend and then set the responded data into our session.

We have come far, let's test this part a bit! 🧪

Power up the development server `> yarn dev` in your project, then go to `http://localhost:3000/login`. Then after you click the button "Login", you will see the cookie is set with `HttpOnly` flag.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687595740148/5b063d1e-4fd7-4648-b71b-0e129143bd14.png align="center")

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Because it is not production built, the cookie is not yet set with <code>secure</code> flag.</div>
</div>

Awesome 🚀🎉 ... Let's continue to get info about the user from the session.

### (5) Create a home page

In directory `pages`, create a file `index.ts` *(Basically, I just remove all the stuff scaffolded by* `next-create-app`*, primarily for simplicity* 🤣*)*

```typescript
/** File: src/pages/index.tsx */

import { useEffect, useState } from "react";

import { ISession } from "./api/session";

const getSession = async () => await fetch("/api/session");

const HomePage = () => {
  const [session, setSession] = useState<ISession>();

  useEffect(() => {
    const fetchData = async () => {
      const sessionRes = await getSession();
      const session = await sessionRes.json();
      setSession(session);
    };

    fetchData();
  }, []);

  return (
    <>
      <h1>Home Page</h1>
      <pre>{JSON.stringify(session)}</pre>
    </>
  );
};

export default HomePage;
```

On this page, we use `useEffect()` hook to get user data from the session on mounted.

So, if you go to `http://localhost:3000`, you will see the data. The data will be there even though you refresh the page *(that's all about the session of the user, right)* 😉

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687597367108/4f0ce23f-4e68-4bef-8136-15ea4fab40cb.png align="center")

<div data-node-type="callout">
<div data-node-type="callout-emoji">😨</div>
<div data-node-type="callout-text"><strong>Out of the topic a bit</strong>: you may have noticed the double call to session API in the console. Don't worry, it is because of <code>&lt;StrictMode&gt;</code> in React. It happens only in the development environment as the <code>&lt;StrictMode&gt;</code> helps us to catch bugs during development. Learn more here: <a target="_blank" rel="noopener noreferrer nofollow" href="https://youtu.be/j8s01ThR7bQ" style="pointer-events: none">https://youtu.be/j8s01ThR7bQ</a></div>
</div>

Fantastic ⚡️... now let's finish this ... logging out the user.

### (6) Add a button to the Home page for logging out the user

```typescript
/** File: src/pages/index.tsx */

import { useEffect, useState } from "react";
import { useRouter } from "next/router";

import { ISession } from "./api/session";

const getSession = async () => await fetch("/api/session");

const destroySession = async () =>
  await fetch("/api/session/destroy", { method: "POST" });

const HomePage = () => {
  const router = useRouter();
  const [session, setSession] = useState<ISession>();

  const handleLogout = async () => {
    await destroySession();
    router.replace("/login");
  };

  useEffect(() => {
    const fetchData = async () => {
      const sessionRes = await getSession();
      const session = await sessionRes.json();
      setSession(session);
    };

    fetchData();
  }, []);

  return (
    <>
      <h1>Home Page</h1>
      <pre>{JSON.stringify(session)}</pre>
      <div>
        <button onClick={handleLogout}>Logout</button>
      </div>
    </>
  );
};

export default HomePage;
```

For the "Home" page, we have added a button to log out. By clicking on the button "Logout", we will call to API to destroy the session and route the user back to the "Login" page.

## That is it!! 🚀

We've done it, let's revise our goals. We've achieved what we need.

1. ✅ Upon the user logging in, we store the user's session in `HttpOnly` cookie
    
2. ✅ Getting user's information from the session
    
3. ✅ Upon the user logging out, we clear the session
    

## Get the codes here 💻

You can find all codes written in this blog here at [GitHub Repo](https://github.com/TVYDev/nextjs-explore/tree/feature/1-session-management-httponly-cookie).

## Finally, don't get me wrong!!! 🥹

There is a lot more to be done to make full authentication for a web app. This is an important part that we've achieved. Way to go!!

So, stay tuned with this series **"Next.js Auth 🔐"**, which I will cover more in the upcoming posts.

## Thanks, cheers! 🤟