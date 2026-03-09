# Quiz — JavaScript Callbacks

*Use these questions to check your understanding. There are no grades — just honest self-reflection.*

---

## Q1 — What is a callback?
**Persona:** Lars (recall anchor)
**Type:** Recall

**Question:**
In your own words: what is a callback function, and why do we use one instead of just calling the next function directly on the line below?

**Self-check:**
A callback is a function you *pass as an argument* to another function, to be called later — typically after an asynchronous operation completes. We can't just call the next function on the next line because JavaScript doesn't wait: by the time that line runs, the async operation (loading a script, fetching data) hasn't finished yet.

If you can explain this without using the word "callback" — in plain language — you've got it.

---

## Q2 — Timing at the cinema
**Persona:** Fatima (application)
**Type:** Application

**Question:**
A developer is building a cinema booking app. Here's their code:

```js
function reserveSeat(seatId, userId) {
  saveToDatabase(seatId, userId);
  sendConfirmationEmail(userId); // runs immediately after
}
```

`saveToDatabase` is asynchronous. What could go wrong? Rewrite `reserveSeat` using a callback so the email is only sent after the seat is confirmed saved.

**Self-check:**
`sendConfirmationEmail` fires before `saveToDatabase` has finished — the seat might not actually be saved yet, so the confirmation email would be premature (or worse, sent for a booking that failed).

Corrected version:

```js
function reserveSeat(seatId, userId, callback) {
  saveToDatabase(seatId, userId, function(err) {
    if (err) return callback(err);
    sendConfirmationEmail(userId);
    callback(null);
  });
}
```

If you spotted the timing problem *and* rewrote it correctly, you're in good shape.

---

## Q3 — Why error-first?
**Persona:** Daan (conceptual)
**Type:** Conceptual

**Question:**
The error-first callback convention says the *first* argument is always the error (or `null`). Why does the *position* of the error argument matter? What could break if different libraries each chose their own order?

**Self-check:**
Position carries meaning in JavaScript function arguments — there are no named parameters in the conventional sense. If each library put the error in a different position (first, second, last), you couldn't write generic helper functions or wrappers that work across libraries. You'd have to read the docs for every single async function to know what to check. The convention exists to make callback-based code *composable and predictable* across the ecosystem — especially important in Node.js, which was built on this pattern.

A strong answer names a concrete consequence of inconsistency, not just "it would be confusing."

---

## Q4 — Where have you seen this before?
**Persona:** All
**Type:** Transfer

**Question:**
The callback pattern — "do this, and when it's done, call me back" — appears in many places outside of JavaScript. Think of one example from your own life, work, or another field. Describe it, and explain which part maps to the callback function and which part maps to the asynchronous operation.

**Self-check:**
There's no single right answer. Strong responses are *precise*: they identify who initiates the operation, what takes time independently, and what the "callback" action is (the thing triggered on completion). Vague answers like "waiting for something" miss the key point — the callback is specifically the action that happens *as a result of* the async operation finishing, not just any waiting.

Some examples that work well: a sports score alert on your phone, a food delivery notification, a CI/CD pipeline that triggers a deploy after tests pass, a doctor's appointment reminder.
