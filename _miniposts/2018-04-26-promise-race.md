---
layout   : post
title    : "Promise.race: who came back first ?"
date     : 2019-08-27
excerpt  : "An handy utility to make using Promise.race easy"
hidden   : true
---

A few days ago, I was playing with [Promise.race](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race), and I got stuck into one pattern I didn't like.

I was having a great volume of asynchronous call to execute in a "race" fashion, :
```js
const operation1 = async () => { await something() }
const operation2 = async () => { await something() }
const operation3 = async () => { await somethingElse() }
const operationKittyPic = async fn () => {
    return await axios.get('https://placekitten.com/200/300')
}

const main = async () => {
    const res = await Promise.race([
        operation1(),
        operation2(),
        operation3(),
        operationKittyPic()
    ])
}

```

Sadly, none had any sort of timeout, which I needed as a safety. First step was to craft a `delay` function:

```js
// ``timeout(ms)` `is going to reject after `ms` milliseconds,
// with a true value
const timeout = ms => new Promise((_, reject) => setTimeout(() => reject(true), ms))

const main = async () => {
    await Promise.race([
        timeout(1000),
        operation1(),
        operation2(),
        operation3(),
        operationKittyPic()
    ])
}
```

But I am now facing an issue: in case an operation fails, **there is no way to know which operation fails, and/or if a timeout occurred.**

One easy solution would be to align the return of **Promise.race** with **Promise.all**, where it would resolve or reject and array of elements all `undefined` but the one that cause the race to end.

```js
const race = (promises) => new Promise((resolve, reject) => {
    promises.forEach(
        (promise, i) => promise
            .then(res => resolve(res))
            .catch(err => {
                let out = [...Array(promises.length)]
                out[i] = err
                reject(out)
            })
    )
})
```

With this `race` utility, you can now easily track who came back first:
```js
const p = race([
    timeout(1000),
    operation1(),
    operation2(),
    operation3(),
    operationKittyPic()
])

p.then(result => console.log(`result = ${result}`))
p.catch(([timeout, ...errors]) => {
    if (timeout) {
        console.log('Timeout')
    } else {
        const uniqueError = errors.find(e => e !== undefined)
        console.log(`Treat the error: ${uniqueError}`)
    }
})
```
