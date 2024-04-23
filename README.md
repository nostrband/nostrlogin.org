# Why Nostr Login?

Were you looking for a cryptographic identity system for your app or website? 

Nostr might be the answer!

It is quite simple.

Users generate a key pair.

The public key of the key pair is the globally unique user identifier. 

The private key of the key pair is used to cryptographically sign properly formatted json messages called [*Nostr events*](https://github.com/nostr-protocol/nips/blob/master/01.md#events-and-signatures). 

That's enough for a global decentralized identity system.

Nostr keys can be used to log into your app, to get tamper-proof data signed by the user, to do encryption. You could also publish Nostr events on [*relays*](https://github.com/nostr-protocol/nips/blob/master/01.md#communication-between-clients-and-relays), but that's a story for another day.

## How does it work?

Nostr keys are usually stored in browser extensions or in specialized key storage apps. 

The simplest way for web apps to access Nostr keys in a browser is using the [*window.nostr*](https://github.com/nostr-protocol/nips/blob/master/07.md) JavaScript object. It is a standard interface for Nostr key access, and has methods to get the user's public key, to sign Nostr events, to encrypt and decrypt data.

The provider of the window.nostr interface may differ.

Users could install a browser extension, which would inject the window.nostr object into all browser tabs. That's a convenient solution for existing Nostr users, but it's quite hard to onboard new users into your app if you're instructing them to *install a browser extension*...

Another way to access Nostr keys is [Nostr Connect](https://github.com/nostr-protocol/nips/blob/master/46.md). It's a remote key access protocol using Nostr relays to communicate with a key storage app, and it can be used by web and by native apps.

Nostr Connect is not trivial to implement, but luckily - you don't have to. There are several libraries you can use for remote key access, and we'll cover them below.

## How do I start?

Let's start integrating Nostr into your web app.

First, install one of the browser extensions for Nostr key storage.

Second, ask for the user's public key in your app:

const publicKey = await window.nostr.getPublicKey();

That's it! 

Now you have an ID of the current user! You could show personalized data for this ID, you could use it to show the user profile, you could request various Nostr events related to this user from the Nostr network. 

