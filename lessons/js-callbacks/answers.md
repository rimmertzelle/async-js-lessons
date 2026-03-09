# Answer Key — JavaScript Callbacks

*Teacher-facing document. Model answers and grading notes for all exercises.*

---

## During the Lesson

### Exercise 1 — The Streaming Problem

**1. Why doesn't the code work?**

Model answer: `loadPlayer` adds a `<script>` tag to the DOM, but the browser loads the script *asynchronously* — it doesn't pause execution. So `startStream` is called immediately after `loadPlayer` returns, before the player script has finished loading and executed. At that point, `startStream` (or whatever the player defines) doesn't exist yet.

**2. Corrected code:**

```js
function loadPlayer(src, callback) {
  let script = document.createElement('script');
  script.src = src;
  script.onload = () => callback(script);
  document.head.append(script);
}

loadPlayer('player.js', function(script) {
  console.log('Player loaded:', script.src);
  startStream('champions-league-final.m3u8');
});
```

**3. Console output order:**
The log should appear *before* any output from `startStream`, confirming that the callback fires after the script loads.

> **Grading note:**
> - **Weak:** Student moves `startStream` inside the callback but doesn't understand *why* — just copies the pattern.
> - **Adequate:** Student explains the async timing issue and rewrites correctly.
> - **Strong:** Student can articulate what `script.onload` does and why passing `startStream` as the callback (rather than calling it directly) is the key insight.

---

### Exercise 2 — Movie Night Gone Wrong

**1. Problems with the original code (at least two):**
- *Readability*: Deep nesting makes it hard to follow the flow at a glance — you have to read inward to understand the sequence.
- *Error handling*: Each callback has its own `err` parameter but none are checked — errors would silently fail or cause confusing undefined-access crashes downstream.
- *Maintainability*: Adding a fourth step requires adding another level of nesting. Removing or reordering a step is error-prone.
- *Testability*: Each step is an anonymous function buried inside another — they can't be unit-tested in isolation.

**2. Named-function refactor (example):**

```js
function onSubtitlesFetched(err, subtitles, trailer) {
  if (err) return console.error(err);
  playTrailer(trailer.url, subtitles);
}

function onTrailerFetched(err, trailer) {
  if (err) return console.error(err);
  fetchSubtitles(trailer.id, function(err, subtitles) {
    onSubtitlesFetched(err, subtitles, trailer);
  });
}

function onFilmFetched(err, film) {
  if (err) return console.error(err);
  fetchTrailer(film.id, onTrailerFetched);
}

fetchFilm('inception', onFilmFetched);
```

**3. Comparison note:**
There is no single right answer for the refactor. Key things to look for: Are functions meaningfully named? Is error handling present? Does the code read top-to-bottom without deep nesting?

> **Grading note:**
> - **Weak:** Student renames functions but keeps nesting structure.
> - **Adequate:** Student correctly flattens nesting with named functions and can identify at least two problems with the original.
> - **Strong:** Student can articulate *why* named functions don't fully eliminate the problem (still callback-based, still hard to read when chains grow) and hints that a different abstraction might be needed.

---

### Exercise 3 — The Broken Score Fetcher

**Two bugs:**

1. Error argument passed as a string, not as `null` for success:
   - `callback('No data found')` is correct as the error case.
   - `callback(data)` is **wrong** — it passes `data` as the *first* argument (the error position), not as the second. Should be `callback(null, data)`.

2. The `getScore` caller checks `err` and `score` as separate arguments, but the broken implementation never passes `null` as the first argument on success — so `score` is always `undefined`.

**Fixed code:**

```js
function getScore(matchId, callback) {
  fetchFromApi(matchId, function(data) {
    if (!data) {
      callback(new Error('No data found'), null);
    } else {
      callback(null, data);
    }
  });
}
```

**Why argument order matters:**
The error-first convention is a community standard (used extensively in Node.js). If you flip the arguments, any code that follows the convention will treat your result as an error and your error as a result. It's a contract: the *position* carries meaning, not just the name.

> **Grading note:**
> - **Weak:** Student finds one bug but misses the other.
> - **Adequate:** Student finds both bugs and fixes them correctly.
> - **Strong:** Student can explain the *convention* and why it exists — it enables consistent, composable error handling across libraries and teams.

---

## Homework

### Homework 1 — Build Your Own Callback Chain

**Model `simulateFetch` implementation:**

```js
function simulateFetch(data, delay, callback) {
  setTimeout(function() {
    if (!data) {
      callback(new Error('Fetch failed: no data'));
    } else {
      callback(null, data);
    }
  }, delay);
}
```

**Example chain:**

```js
simulateFetch({ name: 'Christopher Nolan', id: 'nolan-42' }, 300, function(err, director) {
  if (err) return console.error('Step 1 failed:', err.message);

  simulateFetch(null, 200, function(err, filmography) { // deliberate error
    if (err) return console.error('Step 2 failed:', err.message);

    const recentFilms = filmography.filter(f => f.year > 2010);
    recentFilms.forEach(f => console.log(f.title));
  });
});
```

**Key elements of a strong reflection:**
- Identifies where the error propagation breaks down (returning early vs. letting it fall through)
- Notes the repetitive `if (err) return` pattern
- Makes a genuine observation about what would make this easier (e.g., "some way to handle errors once instead of in every step")

> **Grading note:**
> - **Weak:** Code runs but error handling is missing or incomplete. Reflection is superficial.
> - **Adequate:** Working chain with correct error-first convention. Reflection names a real difficulty.
> - **Strong:** Student independently notes that Promises or async/await would centralise error handling. Reflection shows genuine metacognitive awareness.

---

### Homework 2 — Callback vs. Real Life

**Key elements of a strong answer:**

*Part 1:* The analogy is precise. Student can clearly identify the three roles: who initiates (caller), what the async operation is (something that takes time independently), and what the callback is (the action taken on completion). Vague analogies like "waiting for something" score lower.

*Part 2:* The synchronous scenario is genuinely absurd or costly in the real-world context — this shows understanding of *why* async matters, not just *what* it is.

*Part 3:* Pseudocode correctly captures: passing a function reference (not calling it immediately), and the callback being invoked *by* the async operation (not the caller) when done.

**Example strong answer (pizza delivery):**
```
orderPizza(myAddress, function onDeliveryArrived(pizza) {
  eatPizza(pizza);
});
// meanwhile I keep watching the match
watchMatch();
```
Synchronous problem: I'd have to stand at the door until the pizza arrives, doing nothing else.

> **Grading note:**
> - **Weak:** Analogy is loose or pseudocode just calls the function immediately rather than passing it.
> - **Adequate:** Clear real-world mapping with a correct pseudocode structure.
> - **Strong:** Student independently notes the role of the *pizza shop* (the async operation) as the one who "calls back" — not the customer. This is the key mental model shift.

---

### Homework 3 — Callback Hell Critique

**1. Three concrete problems:**
- *No error handling*: Every `err` parameter is silently ignored. A failed login would cascade into undefined-access errors deeper in the chain.
- *Impossible to test in isolation*: `getProfile`, `getPreferences`, etc. are embedded as anonymous functions — you can't import and test them separately.
- *Reading direction*: To understand the sequence, you read inward (right), not downward. This is opposite to how we normally read code and increases cognitive load.
- *(Also valid):* Hard to add steps, hard to reuse individual steps, variable scope bleeds between levels.

**2. Named-function refactor:**

```js
function onRecommendations(err, recs) {
  if (err) return console.error(err);
  displayDashboard(recs);
}

function onPreferences(err, prefs) {
  if (err) return console.error(err);
  getRecommendations(prefs, onRecommendations);
}

function onProfile(err, profile) {
  if (err) return console.error(err);
  getPreferences(profile.id, onPreferences);
}

function onLogin(err, session) {
  if (err) return console.error(err);
  getProfile(session.id, onProfile);
}

login(user, pass, onLogin);
```

**3. Does it fully solve the problems?**
No. Named functions fix readability and testability, but:
- Error handling is still repetitive (`if (err) return...` in every function)
- The chain is still linear and brittle — adding or reordering steps requires touching multiple functions
- The callback-based pattern itself is the root of these limitations

**4. Stretch — Promise sketch:**

```js
login(user, pass)
  .then(session => getProfile(session.id))
  .then(profile => getPreferences(profile.id))
  .then(prefs => getRecommendations(prefs))
  .then(recs => displayDashboard(recs))
  .catch(err => console.error(err));
```

> **Grading note:**
> - **Weak:** Problems listed are vague ("it's hard to read"). Refactor keeps nesting.
> - **Adequate:** Three specific problems identified, correct flat refactor, honest assessment of its limits.
> - **Strong:** Student articulates that the *pattern* (not just the syntax) is the limitation, and the Promise sketch demonstrates genuine understanding of what a better abstraction looks like.
