# Lesson: JavaScript Callbacks

**Target audience:** HBO-ICT first-year students (mixed HAVO, MBO, non-traditional intake)
**Duration:** 90 minutes
**Learning objectives (as questions):**
- Why can't JavaScript always execute code in the order it's written?
- What is a callback, and why does passing a function as an argument solve the timing problem?
- How do you handle errors in a callback-based pattern?
- What goes wrong when callbacks are nested too deeply, and what can you do about it?

---

## 1. Opening — Challenging Problem or Question

**Duration:** ~10 minutes

Present this scenario without any code:

> You're building a web app. When the page loads, it needs to fetch user data from a server, then use that data to load their profile picture, then use the picture URL to display a personalised greeting. You write three lines of code, one after another. You run it — nothing works. Why?

Ask students to discuss in pairs: *"What do you think is going wrong? What does the browser actually do when it runs those three lines?"*

Take responses in plenary. Surface the intuition that the browser doesn't wait — it fires all three operations at once, before any of them finish.

**Framing question for the lesson:**
> "If JavaScript doesn't wait, how do we tell it: 'Only do this *after* that other thing is done'?"

<!-- Persona note:
- Fatima: The server-fetch scenario mirrors real helpdesk/web work she's encountered. Frame it as "you've probably seen this break in practice — now we'll understand *why*."
- Lars: Concrete visual scenario before any code. Make the problem feel real before introducing terminology. Offer a worked breakdown of the scenario on the slide.
- Daan: The question "what does the browser *actually* do?" invites his curiosity into the runtime mechanics. Don't close the question too fast — let him speculate.
-->

---

## 2. Prior Knowledge Check

**Duration:** ~10 minutes

Quick ungraded written/verbal check (can use Mentimeter or whiteboard):

1. "What is a function in JavaScript? Can you give an example in your own words?"
2. "Have you ever seen code where something happened *later* — like after a click, or after a delay? What did that look like?"
3. "True or false: JavaScript runs your code line by line and always finishes one line before starting the next." (Expected: most say true — this is the key misconception to surface.)

Debrief: Acknowledge the true/false question openly. Most people assume synchronous execution. That assumption breaks down the moment networks, timers, or files are involved.

<!-- Persona note:
- Lars: Question 1 gives him a low-stakes entry point. He's likely seen functions in theory but may not have a practical example ready. Reassure: no wrong answers here.
- Fatima: Question 2 plays to her experience. She may have used `addEventListener` or `setTimeout` without knowing the name for what she was doing. Validate that explicitly.
- Daan: May already know some of this. Invite him to articulate *why* the synchronous assumption breaks — not just *that* it does.
-->

---

## 3. Core Concepts

**Duration:** ~25 minutes

### 3.1 — Asynchronous execution and the problem it creates

Introduce the `loadScript()` example from javascript.info:

```js
function loadScript(src) {
  let script = document.createElement('script');
  script.src = src;
  document.head.append(script);
}

loadScript('/my/script.js');
newFunction(); // ❌ script not loaded yet!
```

Ask: *"What happens if `newFunction` is defined in the script we just loaded?"*

Key concept: **asynchronous actions initiate now but finish later**. The line after `loadScript` runs immediately — it doesn't wait.

### 3.2 — The callback pattern

Show how adding a callback parameter solves the timing problem:

```js
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;
  script.onload = () => callback(script);
  document.head.append(script);
}

loadScript('/my/script.js', function(script) {
  newFunction(); // ✅ runs only after load
});
```

Key insight: **a callback is just a function you pass as an argument, to be called later**. There is no new syntax — only a new pattern.

### 3.3 — Error-first callbacks

Introduce the error-handling convention:

```js
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;
  script.onload  = () => callback(null, script);  // success
  script.onerror = () => callback(new Error(`Load error for ${src}`));
}

loadScript('/my/script.js', function(error, script) {
  if (error) {
    // handle error
  } else {
    // use script
  }
});
```

Convention: **first argument is always the error** (or `null`). This is a Node.js and JavaScript community standard.

### 3.4 — Callback hell / pyramid of doom

Show what happens when you need sequential async operations:

```js
loadScript('1.js', function(error, script) {
  loadScript('2.js', function(error, script) {
    loadScript('3.js', function(error, script) {
      // ...
    });
  });
});
```

Ask: *"What problems do you foresee with this structure as it grows?"*

Surface: readability, error handling repetition, difficulty testing individual steps.

Briefly show the named-function refactor as a partial solution. Close with a forward pointer: Promises (next lesson) solve this structurally.

<!-- Persona note:
- Lars: Use the visual indentation of callback hell explicitly — draw it or animate it on the slide. The "pyramid of doom" metaphor is memorable. Provide a worked example before asking him to spot the problem.
- Fatima: She's likely written something like this and felt the frustration without knowing what to call it. Naming it ("callback hell") is validating, not condescending. Ask: "Has anyone written something that looked like this?"
- Daan: Ask him to consider *why* named functions only partially solve the problem — lead him toward recognising the need for a different abstraction (Promises). Don't give the answer; ask the question.
-->

---

## 4. Application

**Duration:** ~30 minutes

### In-class task (individual or pair)

Students receive a stub: a `fetchData(url, callback)` function that simulates an async fetch using `setTimeout`. Their task:

1. **Low floor:** Call `fetchData` and log the result inside the callback. Make it work.
2. **Mid level:** Chain two `fetchData` calls — use the result of the first to determine the URL for the second.
3. **High ceiling:** Add error handling using the error-first convention. Deliberately trigger an error and handle it gracefully. Then: refactor the nested chain using named functions.

Teacher note: Students can stop at whichever level feels meaningful. Encourage pairs to compare their structures — there is no single right answer for the refactor.

<!-- Persona note:
- Lars: Steps 1–2 are structured and have clear success criteria. He should be able to complete them with the worked examples from section 3. Pair him with someone who can talk through what's happening.
- Fatima: Will likely move quickly through steps 1–2. Step 3 (refactor) is where she earns the depth this lesson is designed to give her. Acknowledge: "this is the part where it starts to matter architecturally."
- Daan: Give him an optional extension: "Can you imagine a way to handle N sequential fetches without N levels of nesting? Sketch an idea." This foreshadows Promises without handing it to him.
-->

---

## 5. Reflection

**Duration:** ~15 minutes

Metacognitive closing — students write or discuss:

1. "At the start of the lesson, you probably assumed JavaScript runs line-by-line. What do you think now? Where does that assumption still hold, and where does it break down?"
2. "Where in your own project work (current or past) might callback-style code already be hiding — without you realising it?"
3. "What's one thing about callbacks that still feels unclear or uncomfortable? Name it precisely."

Group debrief: ask 2–3 students to share question 3. Write the named uncertainties on the board. Validate: "These are real open questions. We'll return to them when we cover Promises."

<!-- Persona note:
- Daan: Question 1 rewards his speculative thinking. Invite him to draw the line more precisely.
- Fatima: Question 2 directly honours her experience. Give her space to speak — she may surface patterns that make the concept click for Lars.
- Lars: Question 3 gives him a safe exit point. Externalising what he doesn't understand yet is a skill in itself. Normalise it explicitly.
-->

---

## Sources Used

- javascript.info/callbacks — Primary technical source
- *What the Best College Teachers Do* by Ken Bain — Educational principles (via `educational_context.MD`)
- `persona.md` — Student archetype profiles for persona-aware design
