---
draft: false
title: The Moddable emulator ("unofficial" mod support in any software?)
snippet: "Ornare cum cursus laoreet sagittis nunc fusce posuere per euismod dis vehicula a, semper fames lacus maecenas dictumst pulvinar neque enim non potenti. Torquent hac sociosqu eleifend potenti."
image: {
    src: "https://images.unsplash.com/photo-1593720213428-28a5b9e94613?&fit=crop&w=430&h=240",
    alt: "full stack web development"
}
publishDate: "2024-04-28 14:37"
category: "Engineering"
author: "Sam Smith"
tags: [mod-support, browser-extension]
---

Moddable adds incredible mod-support to business software. But there's a catch... the application has to install our component.

The problem is - we want to show them what it looks like **before** they install it. So we built the "Moddable Emulator".

As the name suggests, this makes products act like Moddable is installed.

This post is about how we found a way to make this happen: injecting Moddable into third-party platforms that haven't yet installed it.


## But why?

I believe in "show not tell".

Specifically we want to show product-managers, etc. how powerful mod support could be **in their product**.

The demo I wanted looks like this:
 - We load up a version of **their platform** with Moddable installed.
 - We click "Add custom feature", build & use a new custom feature just like a user would.

The ideal demo is Moddable running **in their platform** and us showing the end-to-end process of a user building their own custom features.



## Four Theoretical Methods
In theory you can inject a library like Moddable into a SaaS product in a few ways.

| When            | How                | Technique |
| :-------------- | :--------------    | :---------- |
| At runtime      | Browser extension  |    Use `chrome.scripting` API    |
| At runtime      | Proxy server       |    Rewrite index file to inject a script into `<head>`    |
| At build time   | Browser extension  |    Use `chrome.webRequest` to modify bundled files    |
| At build time   | Proxy server       |    Replace the bundle with a proxied + modified bundle    |

Basically it's a combination of when (runtime or build time) and how (proxy server or browser extension).

The **when** is obvious: runtime is simpler than modifying the built files. Theoretically you would get a cleaner result modifying the built files, but it's harder as they're minified and bundled. 

So runtime became "Plan A" and we would only look at modifying the built files if we hit a roadblock.

The **how** is an interesting trade off:
- Browser extensions are (to me at least) easier to build.
- A proxy server is easiest to distribute. You can share it with a URL, and you don't need the browser's approval.

I decided to start with a browser-extension, as that's what I am most comfortable with. I don't need to distribute it widely (for now). I just need  to show it on demo calls.


## Building a Prototype
Browser extensions can use the `chrome.scripting` API to interact with the page after it's loaded. Or they can use the `chrome.webRequest` API to intercept and modify the files themselves before it's loaded.

As discussed above, to avoid deadling with bundled files we're going to use the `chrome.scripting` API. (Only looking at `chrome.webRequest` if we hit a roadblock).


I started a new [Plasmo](https://plasmo.com) extension. They make setting up a good developer experience with HMR easy & deserve a shout-out.


A minimal prototype to inject Moddable was actually quite simple to build. It's maybe a couple of thousand lines. Here's the core logic:

1. Find a target DOM node to insert into/alongside, etc.

2. Build the Moddable UI and insert it into the DOM node. The UI has a "Manage Mods" button & some links to open any built mods.

3. Watch this for changes and rebuild as appropriate.

4. The links in the inserted content use the Moddable API to get the list of mods, the links, etc. These links are self-autheticating. The end-user never needs to login or create a Moddable account.

5. Insert a stylesheet to match the platform's styles. This is purely cosmetic but it's important to make it look like this belongs. The whole point is to help platforms see what mod support might **look like** in their platform. And it's pretty trivial to implement.

Great! This worked for our first platform. Time to scale it up.


## Building MANY Extensions

The next step is to build a lot of these. This calls for a refactor.

It was important to us that every possible qualified team who [requests a demo](https://meetings-eu1.hubspot.com/sam14/moddable) sees it running live in their platform. So we needed an abstraction that would make it easy to set up for a new platform.

Here's what we ended up with:
    
```typescript
class NewPlatformExtension extends InsertModdableUI {
  protected DOM_SELECTOR = '[data-testid="query-sidebar"]'; // where should we insert stuff
  protected API_KEY = "ABC123"; // for the Moddable API
  protected PLATFORM_ID = "XYZ123";  // for the Moddable API
  protected CustomStyles = `/** CSS goes here **/`;

  constructor() {
    super({
      formatInsertedHTML: (html: string) => `<h2>Custom Features</h2>${html}`, // to customize the layout around the moddable UI
    });

    const watcher = new ContextWatcher({
      extractContent: (element: Element) => { }  // code goes here to emulate the Context sent by the platform       
    });
    watcher.startWatching((val) => {
      this.setPlatformContext(val); 
    }, "[data-testid='display']") // where to watch for changes
  }
}
```

To unpack this abstraction:
1. `InsertModdableUI` is a base class that handles the core logic of mock-inserting the Moddable UI.
2. `CustomStyles` and `formatInsertedHTML` allow for visual customization of the Moddable UI to match the platform's style.
3. `ContextWatcher` is used for emulating the Context which the platform supplies to the moddable SDK. This means that for example in a CRM system the custom mods know which contact is open, etc. (Whatever context makes sense for the given platform).



## Next Steps

 - If you're a user of a platform that doesn't yet support Moddable, you can [request mod-support](https://moddable.com).
 - If you're a platform you can [ask for a demo](https://meetings-eu1.hubspot.com/sam14/moddable) of mod-support installed in your platform.