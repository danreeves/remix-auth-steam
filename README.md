Fork published at https://www.npmjs.com/package/@danreeves/remix-auth-steam

# Remix Auth Steam Strategy

This is a [Steam](https://steamcommunity.com/) strategy for [remix-auth](https://github.com/sergiodxa/remix-auth) library.

## Supported runtimes

| Runtime    | Has Support |
| ---------- | ----------- |
| Node.js    | ✅          |
| Cloudflare | Not tested  |

## How to use

### Installation

Please, use

```bash
npm i remix-auth-steam
```

or

```bash
yarn add remix-auth-steam
```

### File structure

To properly use the library, you need to maintain these additional files in your `app` directory:

`app/services/auth.server.ts`:

```ts
import { Authenticator } from "remix-auth";
import { sessionStorage } from "~/services/session.server";
import { SteamStrategy, SteamStrategyVerifyParams } from "remix-auth-steam";

export type User = SteamStrategyVerifyParams;

export let authenticator = new Authenticator<User>(sessionStorage);

authenticator.use(
  new SteamStrategy(
    {
      returnURL: "http://localhost:3000/auth/steam/callback",
      apiKey: "YOUR_STEAM_API_KEY", // you can get it here: https://steamcommunity.com/dev/apikey
    },
    async (user) => user, // perform additional checks for user here, I just leave this to SteamStrategyVerifyParams value
  ),
);
```

`app/services/session.server.ts`:

```ts
import { createCookieSessionStorage } from "remix";

const calculateExpirationDate = (days: number) => {
  const expDate = new Date();
  expDate.setDate(expDate.getDate() + days);
  return expDate;
};

// export the whole sessionStorage object
export let sessionStorage = createCookieSessionStorage({
  cookie: {
    name: "_session", // use any name you want here
    sameSite: "lax", // this helps with CSRF
    path: "/", // remember to add this so the cookie will work in all routes
    httpOnly: true, // for security reasons, make this cookie http only
    secrets: ["s3cr3t"], // replace this with an actual secret
    secure: process.env.NODE_ENV === "production", // enable this in prod only
    expires: calculateExpirationDate(7), // expire cookie after 7 days
  },
});

// you can also export the methods individually for your own usage
export let { getSession, commitSession, destroySession } = sessionStorage;
```

`app/routes/auth/steam.tsx`:

```tsx
import { LoaderFunction } from "remix";
import { authenticator } from "~/services/auth.server";

export let loader: LoaderFunction = async ({ request }) => {
  await authenticator.authenticate("steam", request, {});
};
```

`app/routes/auth/steam/callback.tsx`:

```tsx
import { LoaderFunction } from "remix";
import { authenticator } from "~/services/auth.server";

export let loader: LoaderFunction = ({ request }) => {
  return authenticator.authenticate("steam", request, {
    successRedirect: "/",
    failureRedirect: "/login",
  });
};
```

### Utilization

After that, navigate to `localhost:3000/auth/steam` to check if it works. Here is an example of checking if user is authenticated:
`app/routes/index.tsx`:

```tsx
import { LoaderFunction, useLoaderData } from "remix";
import { authenticator, User } from "~/services/auth.server";

export let loader: LoaderFunction = async ({ request }) => {
  const user = await authenticator.isAuthenticated(request);
  return user;
};

export default function Index() {
  const user: User | null = useLoaderData();

  return (
    <div style={{ fontFamily: "system-ui, sans-serif", lineHeight: "1.4" }}>
      {user ? <h1>User name: {user!.nickname}</h1> : <h1>Not authenticated</h1>}
    </div>
  );
}
```

If you are logged in, you should see your username from Steam API, otherwise you will see `Not authenticated` message.

# Contributing

Your contributions are highly appreciated! Please, submit any pull requests or issues you found!
