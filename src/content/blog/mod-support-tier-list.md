---
draft: false
title: Mod Support Tier List - Comparing major SaaS platforms
snippet: "How we added mod support to business software, even when Moddable isn't installed."
image: {
    src: "/blog/mod_support_tier_list.png",
    alt: "Tier list for common SaaS platforms with mod support"
}
publishDate: "2024-05-02 20:25"
category: "Engineering"
author: "Sam Smith"
tags: [mod-support, browser-extension]
---

Mod support is the defining feature that makes something a platform rather than simply a product.

Here's a ranked tier-list of how some of the biggest SaaS platforms are tackling mod support.

---

# Overall Findings
There were three major takeaways from this. (Or you can skip straight to the [tier list](#tier-list)).

### 1. There's a clear "good", "better" and "best" capability:
 - "Good" is the absolute basics: a robust API (create/read/update/delete data) and webhooks (to trigger code in response to an event).
 - "Better" platforms **also** offer some kind of UI extension. This is where you can add a button, a sidebar, etc. to the platform's UI.
 - "Best" platforms let you write full interactive JavaScript/React apps as UI extensions, with full access to the platform's API and the user's context.

### 2. Getting to "Hello World" is *absurdly* hard
 - The best platforms (Salesforce, HubSpot, Intercom) automatically give you a free dev instance without having to speak to anyone. The others literally make you beg or pay to help them extend their platform! Some require a LOT of paperwork, contracts, NDAs and a significant fee.

 - Almost every platform requires you to provision a server (strongly gatekept by IT), setup a new repo, build tooling, etc. This is kind of insane in 2024. It is SO difficult to get approval to do this in an enterprise. This requirement makes mods only realistically possible by professional system ingreators.

 - The best platforms simplify the local setup with a CLI.

**Nobody has built a zero-setup approach:** giving you a built-in editor so you can just hit an "add a custom feature" button and start instantly. (See our demo for how this can work).


### 3. Never make a DSL.
There's a false-start in mod-support which *seems* like a good idea but really isn't.

Some platforms have built their own domain-specific-language. For instance Salesforce Marketing Cloud has "AMPScript". Intercom has their XML webhook based "Intercom Apps" framework. And main Salesforce has "SOQL" (a worse version of SQL).

These are a huge mistake! I understand how these evolved, but it reminds me of that weird "WAP" mobile web that existed before the iPhone. They really are just a stop-gap. 

I think the justification for these is control. But in reality it's probably because implementing a DSL is easy and building a true Monday-style extension SDK is a very, very hard problem.


---


# Tier List

## S-Tier: Best in the World

### Monday
It might surprise you but Monday (the flexible project management tool) is the best I've seen for mod-support. I think this is a major reason for their breakout success.

Here's why:
- ✔️ You can extend the UI with actual full custom React apps(!)
- ✔️ These extension apps have the user's context (the board they're looking at, data in that table, etc.)
- ✔️ The extension apps have API access as the user who's logged in (to call Monday's own API, update the board, etc.)
- ✔️ Their framework around automation extensions, etc is also top-tier.

It's not perfect:
- ❌ Getting started is a challenge. It requires a pretty motivated dev to get to "Hello World".
- ❌ They don't offer free dev instances. This causes friction and insults their partners.
- ❌ Iterating locally is awkward. You have to proxy to localhost and it tends to hang, is kind of slow, hard to configure, etc.


**Fun fact: Moddable itself is inspired by Monday's mod-support!**
Monday used to run hackathon competitions. I won $5k in one of them. In this experience I was so impressed with their mod support. This definitely influenced my decision to build Moddable. (I wanted to bring those strengths but with a much easier developer experience and zero setup).


### Shopify
Shopify is S-tier. They are **clearly** a best-in-class engineering org. I could easily write pages on their approach to mod-support. It needs its own post.

They have innovated in the space more than anyone (probably including us).

Let me be clear: their approach could be improved with a zero-tooling setup. But what they have done is still VERY interesting.

Some highlights:
 - ✔️ [They literally acquired Remix](https://remix.run/blog/remixing-shopify) (the promising React meta-framework).
 - ✔️ [Their hydrogen framework](https://hydrogen.shopify.dev/) for headless ecommerce is very well done. The concept of a headless framework is extremely interesting. It kind of goes beyond mod support even; letting you take over the whole front-end with strong primitives. (Hydrogen is built on Remix).
 - ❓ While doing anything here is clearly a major project, they provide some brilliant tooling e.g. `npm create @shopify/hydrogen@latest`.
 - ❌ Shopify seems like a modern version of Salesforce's approach. Extremely powerful, but targeting only hardcore dev teams.

Shopify seems like a modern version of Salesforce's approach. Extremely powerful, but targeting only hardcore dev teams.


You might be asking why does Hydrogen not count as a DSL? It is a **superset** of React.


---


## A-Tier: Excellent

### Salesforce

People think Salesforce is a CRM. It really isn't. It's much more the original enterprise platform for building apps.

Their strength is the power and the "correctness". Downside is there's so much legacy that everything is pretty clunky & requires specialized consultants. They've made good progress in unifying things towards "normal" standards recently.

- ✔️ Extremely capable.
- ✔️ You can extend the UI with actual full custom React apps(!)
- ✔️ Salesforce does a great job of giving dev instances instantly
- ❓ Half way between software and a full on PaaS. Not sure how I feel about this one.
- ❌ There's a lot of "legacy". Nothing feels modern.
- ❌ A LOT of setup. You need an instance, a thorough grounding in 'the Salesforce way of doing things' and a ton of tooling and patience just to get started
- ❌ Everything is very Salesforce-specific. (For example: SOQL - their "SQL-like" query language, which is a completely Salesforce specific and vastly inferior dialect.)

There will never be another Salesforce. They pioneered a lot. But the world has moved on. So it has its own very unique ecosystem that sustains it through its sheer size.

---


## B-Tier: Good
### HubSpot
- ✔️ Quite capable.
- ✔️ They give you a dev instance instantly & painlessly
- ✔️ You can extend the UI with a small set of components & a subset of React.
- ❓ Feels more modern but way less capable than Salesforce. They are catching up (e.g. adding custom data objects - but it's very immature compared to Salesforce).
- ❌ See below. Their UI extensions are really limited. Halfway between a DSL and full React.

HubSpot recently added UI extensions to their CRM. (This is their answer to Salesforce's APEX.). Turns out I have a lot to say about this. It's well done but hugely restricted: only a set of allowed components, and a huge list of limitations (even simple basic stuff just won't work).

A note on "security" here. This is intended to help safety but it doesn't *actually* stop a malicious extension from harvesting your data. It's a bit like the TSA: it's mostly for making you feel safer.


More importantly, they completely miss a major use-case: power users extending their own instance with one-off mods:
 - It's hard. For professionals doing major projects.
 - The harsh security model applies *even when you wrote the code yourself*. Most platforms let you run stuff yourself then have a different model for distribution post-review.

 The needs of getting something published for the masses (strict security, review, complexity) are very different from the needs of a power user extending their own instance (speed, ease, flexibility). 

---

## C-Tier: Honorable mention

### Intercom
Intercom's mod support is a very, very good implementation of a fundamentally outdated method.

- ✔️ They give you an instant, free dev instance.
- ✔️ What they have is very well documented & designed.
- ❓ They have a UI extension framework BUT it's a domain specific XML based weird approach.
- ❌ Their framework makes setup and maintenance particularly challenging.

Looking at those pros and cons why is this just C-tier..?

I think it's the fundamental approach. Their XML framework will only ever let you stitch together the primitives they provide you with.

*The craftsmanship is excellent*. It's just the wrong method. (In fairness to them, the modern methods were not practical when they built this.)


### Eloqua
I know Eloqua (Oracle's marketing cloud) very well. I recommend it. Their mod support is a powerful but complex framework.

- ✔️ Very powerful for "backend" stuff. E.g. you can write custom steps on campaigns
- ✔️ Their framework while hard to use was definitely ahead of its time. They do brilliant things like let you specify exactly which fields you want to be sent, and make it possible to build apps that automatically authenticate users.
- ❓ You can **sort of** extend the UI - but only by adding links to load an iFrame.
- ❌ Incredibly complex. Not recommended even for professionals.
- ❌ Getting a partner instance requires a LOT of paperwork/contracts and a major fee. (HUGE strategic mistake.)

Their mod-support is very powerful. But very difficult to use.


Disclaimer/interesting note: [one of my companies](https://instant.marketing) sells add-ons which build a code-editor into Eloqua itself to simplify building custom steps/integrations. This probably set me down the path towards eventually building [Moddable](https://moddable.app)!


---


## D Tier: Not like this...

### Salesforce Marketing Cloud
Mod support in Salesforce Marketing Cloud was ahead of its time. But it has not aged well.

To do almost anything in this platform (even as a user) you need to write in one of their two native scripting languages: AMPScript or SSJS.
 - AMPScript is a weird, limited, and very old scripting language. It exists only in SFMC. I don't think it's been updated in 15 years.
 - SSJS stands for "server side JavaScript". To be clear I do *not* mean NodeJS. This seems to literally pre-date NodeJS. It's a super old subset of JavaScript seemingly from 1990.

 **No error messages 🤮**
 When a program in AMPScript/SSJS fails, it simply dies. No error message. No stack trace. No logs. Nothing.

 I've included SFMC here not to bash them - but to show the perils of implementing a DSL.

 What was genuinely ahead of its time has diverged from standards and stood still while the web platform has evolved to the point it makes SFMC feel like working with punch-cards and tape.


- ❌ AMPScript should be burned by fire
- ❌ SSJS is much better than AMPScript... but so is everything. Burn it with fire.
- ❌ No UI extensions of any kind (that I know of)
- ❌ No dev instances. To get access to a sandbox is a mix of paperwork and begging and takes months if ever