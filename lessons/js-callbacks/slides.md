---
marp: true
theme: default
paginate: true
---

# JavaScript Callbacks
### How do we tell JavaScript: *"only do this after that is done"?*

---

## Agenda

1. Opening problem
2. Prior knowledge check
3. Core concept: async execution
4. The callback pattern
5. Error handling
6. Callback hell
7. Application exercise
8. Reflection

---

## You're building a sports streaming site.

When the user clicks **"Watch match"**, the app must:

1. Load the video player script
2. Start buffering the stream

You write two lines of code, one after the other.

**It doesn't work.**

*Why not?*

---

## Quick check — what do you already think?

1. What is a function in JavaScript? Give an example in your own words.

2. Have you ever seen code that ran *later* — after a click, a delay, a fetch? What did it look like?

3. **True or false:** JavaScript runs your code line by line and always finishes one line before starting the next.

---

## The assumption that breaks everything

```js
loadPlayer('player.js');
startStream('match.m3u8'); // ❌ player not ready yet
```

`loadPlayer` adds a script tag — but the browser loads it **asynchronously**.

JavaScript doesn't wait. It fires the next line immediately.

> Asynchronous actions **initiate now** but **finish later**.

---

## So how do we say "run this *after*"?

We pass the next step as an argument — a **callback**.

```js
function loadPlayer(src, callback) {
  let script = document.createElement('script');
  script.src = src;
  script.onload = () => callback(script);
  document.head.append(script);
}

loadPlayer('player.js', function(script) {
  startStream('match.m3u8'); // ✅ runs only after load
});
```

A callback is just a **function passed as an argument**, to be called later.

---

## Handling errors — the error-first convention

```js
function loadPlayer(src, callback) {
  let script = document.createElement('script');
  script.src = src;
  script.onload  = () => callback(null, script);   // success
  script.onerror = () => callback(new Error(`Failed: ${src}`));
}

loadPlayer('player.js', function(err, script) {
  if (err) {
    console.error('Could not load player:', err);
  } else {
    startStream('match.m3u8');
  }
});
```

**Convention:** first argument = error (or `null`). Always.

---

## What if you need three things in a row?

```js
loadPlayer('player.js', function(err, script) {
  fetchMatchData('ajax-psv', function(err, match) {
    loadCommentary(match.id, function(err, audio) {
      startStream(match.url, audio);
    });
  });
});
```

---

## The pyramid of doom

```
callback(
  callback(
    callback(
      callback(...)
    )
  )
)
```

- Hard to read
- Hard to test
- Error handling repeated everywhere
- Adding a step = another level of nesting

---

## One fix: named functions

```js
function onCommentaryLoaded(err, audio, match) {
  if (err) return console.error(err);
  startStream(match.url, audio);
}

function onMatchFetched(err, match) {
  if (err) return console.error(err);
  loadCommentary(match.id, function(err, audio) {
    onCommentaryLoaded(err, audio, match);
  });
}

function onPlayerLoaded(err) {
  if (err) return console.error(err);
  fetchMatchData('ajax-psv', onMatchFetched);
}

loadPlayer('player.js', onPlayerLoaded);
```

*Better — but does it fully solve the problem?*

---

## Exercise time

**The Streaming Problem** (individual, ~10 min)

A sports streaming app is broken. Fix it using a callback.

→ See `exercises.md`, Exercise 1

Then: **Movie Night Gone Wrong** (pairs, ~15 min)

A nested movie app callback chain. Find the problems, refactor, compare.

→ See `exercises.md`, Exercise 2

---

## Reflection

Take 5 minutes. Write your answers:

1. At the start, you probably assumed JavaScript runs line-by-line. What do you think now? Where does that assumption hold — and where does it break?

2. Where in your own project work might callback-style code already be hiding?

3. What's one thing about callbacks that still feels unclear? **Name it precisely.**

---

## What's next?

Callbacks solve the timing problem — but they have limits.

**Next lesson:** Promises

> "What if instead of passing a callback *in*, the function returned something you could chain *off of*?"

---

## Key takeaways

- JavaScript is **non-blocking** — async operations finish later, not immediately
- A **callback** is a function passed as an argument to be called when the async work is done
- Use the **error-first convention**: `callback(err, result)`
- Deep nesting → **callback hell** — named functions help, but don't fully fix it
- **Promises** are the next step
