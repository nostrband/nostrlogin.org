# Why Nostr Login?

Why not your own database of emails and passwords? Why not a big provider like Google/Facebook/Twitter?

Both options have issues. 

Your own identity system is an empty database, no data, no social graph. 

A big provider may reduce friction for users, and might give you access to their data. But what if provider turns your app off? What if it starts charging for the data access?

So maybe you're looking for a permissionless identity provider that has a social graph, or has data to train your AIs, or could help your app go viral...

Nostr might be the answer!

It is quite simple.

Users generate a key pair.

The public key of the key pair is the globally unique user identifier. 

The private key of the key pair is used to cryptographically sign properly formatted json messages called [*Nostr events*](https://github.com/nostr-protocol/nips/blob/master/01.md#events-and-signatures). 

That's enough for a global decentralized identity system.

Nostr keys can be used to log into your app, to get tamper-proof data signed by the user, to do encryption. You could also publish Nostr events on [*relays*](https://github.com/nostr-protocol/nips/blob/master/01.md#communication-between-clients-and-relays), but that's a story for another day.

## How does it work?

Nostr keys are usually stored in browser extensions or in specialized key storage apps. 

The simplest way for web apps to access Nostr keys in a browser is using the [window.nostr](https://github.com/nostr-protocol/nips/blob/master/07.md) JavaScript object. It is a standard interface for Nostr key access, and has methods to get the user's public key, to sign Nostr events, to encrypt and decrypt data.

There are different providers of the `window.nostr` interface.

Users could install a browser extension, which would inject the `window.nostr` object into all browser tabs. That's a convenient solution for existing Nostr users, but it's quite hard to onboard new users into your app if you're instructing them to *install a browser extension*...

Another way to access Nostr keys is [Nostr Connect](https://github.com/nostr-protocol/nips/blob/master/46.md). It's a remote key access protocol using Nostr relays to communicate with a key storage app, and it can be used by both web and native apps. It also has OAuth-like flows specified for login and sign up, which makes it a familiar way for users to provide access to their identity.

Nostr Connect is not trivial to implement, but luckily - you don't have to. There are several libraries you can use for remote key access. Some of those libraries would even provide a `window.nostr` interface for you, and will forward your `window.nostr` calls to the remote key storage app.

## How do I start?

Let's start integrating Nostr into your web app.

First, install [one of the browser extensions](https://github.com/aljazceru/awesome-nostr#nip-07-browser-extensions) for Nostr key storage (start with [nos2x](https://chromewebstore.google.com/detail/nos2x/kpgefcfmnafjgpblomihpgmejjdanjjp) or [Nostore](https://apps.apple.com/us/app/nostore/id1666553677)). You'll be using it for testing.

Second, ask for the user's public key in your web app:

```
const publicKey = await window.nostr.getPublicKey();
```

That's it! 

Now you have an ID of the current user! 

You could show personalized data for this ID, you could use it to show the user profile, you could request various Nostr events related to this user from the Nostr network. 

Well, actually that's a weak form of authentication, it doesn't verify that the user owns the private key. User's `window.nostr` provider could be made to return anyone else's public key (those are public, right?). 

However, `getPublicKey` is still a useful and frictionless way to provide personalization within your app. Many Nostr apps have read-only modes where you can log in as *anyone* - how cool is that?!

But suppose read-only mode is not enough for us.

## How to use Nostr keys?

Suppose we need to be sure this user actually owns or has access to the private key corresponding to their public key. 

The way to go is to get the user to actually *sign some Nostr event*. If they produce a valid signature for the public key, then they are who they claim they are.

```
const signedEvent = await window.nostr.signEvent({
 kind: 12345, // event kind 
 created_at: Math.floor(Date.now() / 1000), // event timestamp
 tags: [],    // event tags
 content: ''  // event content
});
```

But what Nostr event should your app sign? What are those `kind`, `tags` and `content` fields for?

Well, ideally you would design your app's data model around Nostr events. Nostr events have `kind`, which is a number you would use to indicate the data type. You would store the data that users produce in a form of signed Nostr events, which would make it all tamper-proof. You could even store all that data on the Nostr relays...

That would be an actual *Nostr app*. You'd need to learn [how the Nostr protocol works](https://github.com/nostr-protocol/nips/blob/master/01.md), and learn to structure your data as Nostr events by studying [existing event kinds](https://github.com/nostr-protocol/nips/tree/master#event-kinds). 

However, you don't have to go that far.

## How to do authentication for API calls?

You can simply use Nostr to authenticate the HTTP API calls your app is making.

There is a [standard](https://github.com/nostr-protocol/nips/blob/master/98.md) way to sign a special Nostr event for your HTTP request and include it as `Authorization: Nostr ...` header. Your server would verify the signed event from the header, and would be sure the user is who they say they are.

Check out the examples for the client and the server. TBD

## How to authenticate using Nostr DMs? 

For some apps it might be too hard to arrange a continuous access to Nostr keys for users. For instance, if you're building a native app, you'd either have to implement *Nostr Connect*, which might be quite complex, or you would just ask users for their private key and learn how to handle it safely (hint: don't do that!).

In this case you could use [Nostr encrypted direct messages](https://github.com/nostr-protocol/nips/blob/master/04.md) for authentication.

Ask users for their public key (usually in the [npub](https://github.com/nostr-protocol/nips/blob/master/19.md) form), then send them a Nostr DM with a one-time code. Users could use any Nostr app that supports DMs to read the one-time code. If they enter the correct code back into your app, you can be sure they have access to the keys for the provided public key.

Your implementation would probably involve a database table of public keys and the matching one-time codes, with codes expiring after several minutes. You will also need to generate Nostr keys for your app, store them on your server and use them to sign and encrypt the DMs with one-time codes. You'll also need to learn to talk to relays, and to discover relays that the target user is reading from...

This is not a trivial option, but sometimes there's just no better way.

## What about the libraries?

First, we recommend you to try [nostr-login](https://github.com/nostrband/nostr-login) as the `window.nostr` provider. Just add `https://www.unpkg.com/nostr-login@latest/dist/unpkg.js` script into your html page and get a powerful UI for various Nostr key access methods (extensions, Nostr Connect, read-only, account switching, etc). Another option is [window.nostr.js](https://github.com/fiatjaf/window.nostr.js).

Second, take a look at [nostr-tools](https://github.com/nbd-wtf/nostr-tools/) and [NDK](https://github.com/nostr-dev-kit/ndk). The first one is a lower-level tool - verify signatures, generate keys, format events, talk to a relay, etc. The second one is higher-level, with sophisticated relay communication logic, Nostr Connect implementation, and other powerful tools.

For native apps, check out [awesome-nostr](https://github.com/aljazceru/awesome-nostr) or just run a github search for `nostr` + your language. Thousands of tools are already out there, you'll certainly find something useful for your particular needs. 

## Still have questions?

Submit an [issue](https://github.com/nostrband/nostrlogin.org/issues) for this page, we will do our best to figure things out for you. Or submit a PR if you have something to contribute.
