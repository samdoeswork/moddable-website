---
draft: false
title: Building a 'Login as Client' mod for HoneyComb
snippet: "How we extended our observability tool (HoneyComb) with a new feature: Login as Client"
image: {
    src: "https://images.unsplash.com/photo-1667372393119-3d4c48d07fc9?&fit=crop&w=430&h=240",
    alt: "frontend master"
}
publishDate: "2024-04-27 16:05"
category: "Tutorials"
author: "Sam Smith"
tags: [mod-support, dog-fooding, devtools, honeycomb]
---

My other company uses HoneyComb for observability. It's great but I've always wished for a custom "next step". After finding problems I want to jump straight into fixing them.

This is part 1 of a series on how we extended HoneyComb with specific mods to help us directly fix client issues. This post is about the "Login as Client" mod.

## The Problem
After finding a problematic client configuration, I've always wished I could log in as that user and see what they see.


## The Mod
The solution mod is simple: when there's account and instance in the HoneyComb UI, let me click to open that instance in a new tab. I need to be logged in as that user. (Only to be used with their permission, of course).


## Making HoneyComb Moddable
HoneyComb hasn't installed Moddable (yet). So I used the unofficial route.

With Moddable installed - the UI now has a "Manage Custom Features" button.


## Building the Mod
The mod itself is just 28 lines. About half of which are the default template (Moddable mods are self-contained little React apps).

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

    this is **synchronized**: as the user navigates this mod will be updated with the new data without reloading.

2. The `loginAsClient` handler is custom. It passes special superadmin key and receives back a token impersonating a specific user. This function then opens the client to that specific account & instance.


## The End Result:
The end result lets us find any issue in HoneyComb, and get to the running client's config in seconds.