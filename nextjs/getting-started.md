# Getting started with NextJS

Getting started with NextJS is quick and easy. Let's do it together.

## Create your account

This guide assumes you have already [created a Userstack account](https://userstack.app).

Once you have signed in, you should be prompted to create a project. Create a project, and note the `project key` -- we will need this.

## Install the client for NextJS

Using NPM:

```
npm install @userstack/nextjs
```

or with yarn:

```
yarn add @userstack/nextjs
```

## Add the Userstack provider

Wrap your app with the Userstack provider.

### Using the pages router

Here is an example `_app.tsx` file:

```tsx
import type { AppProps } from "next/app";
import { UserstackProvider } from "@userstack/nextjs";

export default function App({ Component, pageProps }: AppProps) {
  if (!process.env.NEXT_PUBLIC_USERSTACK_PROJECT_KEY) {
    throw new Error("Missing NEXT_PUBLIC_USERSTACK_PROJECT_KEY");
  }

  return (
    <UserstackProvider
      projectKey={process.env.NEXT_PUBLIC_USERSTACK_PROJECT_KEY}
    >
      <Component {...pageProps} />
    </UserstackProvider>
  );
}
```

This configuration assumes you have stored your Userstack project key in an environment variable called `NEXT_PUBLIC_USERSTACK_PROJECT_KEY` which is what I recommend you do.

### Using the app router

Using the app router, I recommend creating a file in the root of your `app/` called `providers.tsx`, which is a client component used to wrap your app. Here is a recommended example set up you can copy.

```tsx
"use client";

import { UserstackProvider } from "@userstack/nextjs";

export function Providers({ children }: { children: React.ReactNode }) {
  if (!process.env.NEXT_PUBLIC_USERSTACK_PROJECT_KEY) {
    throw new Error("Missing NEXT_PUBLIC_USERSTACK_PROJECT_KEY");
  }

  return (
    <UserstackProvider
      projectKey={process.env.NEXT_PUBLIC_USERSTACK_PROJECT_KEY}
    >
      {children}
    </UserstackProvider>
  );
}
```

Then you use this providers component in your root `layout.tsx` like this

```tsx
import { ReactNode } from "react";
import { Providers } from "@/app/providers";

type LayoutProps = {
  children: ReactNode;
};

const Layout = ({ children }: LayoutProps) => (
  <html>
    <head></head>
    <body>
      <Providers>{children}</Providers>
    </body>
  </html>
);

export default Layout;
```

## Use Userstack in your app

### Identify users

You can use the `identify` function to identify a user, which ensures that any events logged by the user are associated with their account. This should happen at login time, to include their account information.

There are currently two supported methods of identifying a user:

1. Manually specify the user information
2. Connect to Firebase authentication

_Note: Additional authentication providers coming soon_

#### Manually identify the user

Here is an example of identifying the user manually

```tsx
import { useUserstack } from "@userstack/nextjs";

function MyPage() {
  const { identify } = useUserstack();

  const signIn = async () => {
    // ... your sign in logic

    await identify({
      email: "user@example.com",
      userId: "abc123", // optional user ID
      company: "acme", // optional company
    });
  };

  return <div>Page here</div>;
}
```

#### Connect with Firebase

```tsx
import { useUserstack } from "@userstack/nextjs";

function MyPage() {
  const { identify } = useUserstack();

  const signIn = async () => {
    const auth = getAuth(app); // insert your initialized firebase app here
    const provider = new GoogleAuthProvider();

    try {
      const result = signInWithPopup(auth, provider);
      const credential = GoogleAuthProvider.credentialFromResult(result);

      if (credential?.idToken) {
        identify({
          firebaseToken: credential.idToken,
          company: "acme", // optional company
        });
      }
    } catch (error) {
      console.error(error);
    }
  };

  return <div>Page here</div>;
}
```

### Track events

You can use the `track` function to track feature usage and events. Here are a few examples you can copy.

#### Track simple usage of a feature

```tsx
import { useUserstack } from "@userstack/nextjs";

function MyPage() {
  const { track } = useUserstack();

  const doSomething = async () => {
    // ... your logic

    track("feature_name");
  };

  return <div>Page here</div>;
}
```

#### Track usage of a feature with event

```tsx
import { useUserstack } from "@userstack/nextjs";

function MyPage() {
  const { track } = useUserstack();

  const doSomething = async () => {
    // ... your logic

    track("feature_name", "click_feature_button");
  };

  return <div>Page here</div>;
}
```

#### Track feature with event and additional custom data

```tsx
import { useUserstack } from "@userstack/nextjs";

function MyPage() {
  const { track } = useUserstack();

  const doSomething = async () => {
    // ... your logic

    track("feature_name", "click_feature_button", {
      page: "/home",
      foo: "bar",
    });
  };

  return <div>Page here</div>;
}
```
