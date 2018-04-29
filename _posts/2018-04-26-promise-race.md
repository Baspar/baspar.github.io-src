---
layout   : post
title    : "Promise.race: who came back first ?"
date     : 2018-04-26
category : mini
excerpt  : "Promise.race"
image    : "/images/2018-04-26-telegram/thumb.jpg"
---

A few days ago, I was playing with [Promise.race](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race), and I got stuck into one pattern I didn't like.

I had a really simple scenario: I had quite a lot of asynchronous operation to do, in the form of Promises. Sadly most of them didn't include a timeout, *BUT* I was using them though a unique wrapper.

Brilliant, I could wrap each of them in a `Promise.race`, next to a **setTimeout**, pretty easy right ?

**No.** I mean, yeah.. Depends.

### Reject way

`Promise.all` can do the trick, but you can not differentiate who came back first.
Usually, you can *reject* when the timeout is coming first, and *resolve*, in the other case.

```js
// Our real API
const apiCall = axios.get('https://placekitten.com/200/300')

// A delay helper
const delay = (ms = 1000, error = 'Timeout') => new Promise(reject => setTimeout(
    () => resolve(error),
    ms
))

// This should -normally- print a cute kitten
Promise.race([apiCall, delay(3000)]).then(console.log)

// THis should, on the other hand reject with a 'timeout' text
Promise.race([apiCall, delay(1)]).catch(console.log)
```

But what will happen if we receive a rejection from the real API ?

Or if we want to race 15.000 APIs ? (Okay, not really probable)

### The tweak
