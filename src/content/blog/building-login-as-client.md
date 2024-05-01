---
draft: false
title: Building a 'Login as Client' mod
snippet: "How we extended our observability tool (HoneyComb) with a new feature: Login as Client"
image: {
    src: "/blog/incognito.png",
    alt: "Building a 'Login as any client' mod"
}
publishDate: "2024-04-27 16:05"
category: "Tutorials"
author: "Sam Smith"
tags: [mod-support, dog-fooding, devtools, honeycomb]
---

I have been a [HoneyComb](https://www.honeycomb.io/) user for years. It's awesome! 

...But I've always wanted something more. HoneyComb helps find problematic events. But after finding a problem I want to jump straight into fixing it.

In this series, I'll show you how I built some HoneyComb mods to add new power-user features. This part covers how I built a "Login as Any Client" mod.

## Background
Whenever I find some interesting/problematic event in HoneyComb, I always want to see what the client sees. I want to know what settings, etc. they have configured to reach that state.


## The Mod
The mod I built lets me find a specific account/instance in HoneyComb. I can click "Login as Client" and select a row - it opens as a new tab, logged in as that user.

This way I can instantly see what they see when I'm troubleshooting. (With their permission, of course).


## Building the Mod
HoneyComb hasn't installed Moddable (yet). So I used Moddable Emulator.

Moddable mods are mini React apps. Here's the full code (28 lines):

```jsx
import {useAppContext} from "./useAppContext";
import "./style.css";

export default function App() {
    const appContext = useAppContext() as IModdableContext;
    if (appContext) return (<div className="p-4">
        <h2>Login as Client</h2>
        <p>Select an account/instance:</p><br />

    <div className="flex justify-between w-full flex-row">
  <ul className="space-y-2 w-full tableOuter">
  {appContext.platform?.rows?.map((row) => (
    <li onClick={() => loginAsClient(row.account, row.instance)} className="flex justify-between tableRow cursor-pointer">
      <span>{row.account}</span> <span>{row.instance}</span> <span>({row.count})</span>
    </li>
  ))}
</ul>
</div>
    </div>);
    return <>Loading...</>;
}

async function loginAsClient(account: string, instance: string) {
    const tokenRaw = await fetch(`https://app.instant.marketing/superadmin_only/account_login/${account}?token=SUPER_ADMIN_TOKEN`); // the actual URL looks similar to this
    const tokenResponse = await tokenRaw.json() as { user: any; auth: string; };
    const url = `https://client.instant.marketing/eloqua/action/${account}/${instance}?auth=${encodeURIComponent(tokenResponse.auth)}`;
    window.open(url);
}
```


Let's discuss some key parts.

1. `useAppContext` is a hook provided by moddable. This gives whatever context is important from the embedding platform. In the (unofficial) HoneyComb implementation this is a rows array with whatever data is shown in the main panel.

    This is **synchronized**: as the user navigates this mod will be updated with the new data without reloading.

2. The `loginAsClient` handler is custom. It passes special superadmin key and receives back a token impersonating a specific user. This function then opens the client to that specific account & instance.

### About tokens
Implementing the actual endpoint was pretty easy. If you use JWTs it's a simple case of `jwt.sign({ // modified payload })`.

If you use managed auth it might vary but it's generally a line or two once you know the target account/user.

### Note on security
Pay attention to security! Endpoints like this should only *downgrade* permissions, not *upgrade* or *transfer* between accounts. In this example we swap a global admin token for a lower priviledge user. You should think about *audit trails* too.


## The End Result:
The end result lets us find any issue in HoneyComb, and get to the running client's config in seconds.