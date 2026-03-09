# Exercises — JavaScript Callbacks

*These exercises are formative — there are no grades. The goal is to think, experiment, and discuss.*

---

## During the Lesson (In-class)

### Exercise 1 — The Streaming Problem (Individual, ~10 min)

**Context:** You're building a mini sports streaming site. When a user clicks "Watch match", the app needs to:
1. Load the video player script
2. Only *then* start buffering the stream

Here's the starting code — but it's broken:

```js
function loadPlayer(src) {
  let script = document.createElement('script');
  script.src = src;
  document.head.append(script);
}

loadPlayer('player.js');
startStream('champions-league-final.m3u8'); // ❌ player not ready yet
```

**Your task:**
1. Explain in your own words *why* this code doesn't work.
2. Rewrite it so `startStream` only runs after the player script has loaded. Use a callback.
3. Add a `console.log` inside the callback to confirm the order of execution.

*Targets: making meaning / surfacing misconceptions about synchronous execution*

> **Teacher note:** Watch for students who make `startStream` a nested function call but forget to actually *pass* it as a callback argument. That mistake is worth discussing in plenary.

---

### Exercise 2 — Movie Night Gone Wrong (Pair, ~15 min)

**Context:** A movie app fetches a film title, then fetches its trailer URL, then plays the trailer. A teammate wrote this:

```js
fetchFilm('inception', function(err, film) {
  fetchTrailer(film.id, function(err, trailer) {
    fetchSubtitles(trailer.id, function(err, subtitles) {
      playTrailer(trailer.url, subtitles);
    });
  });
});
```

**Your task (with a partner):**
1. This code works — but what makes it hard to maintain? List at least two concrete problems.
2. Refactor it using named functions (`step1`, `step2`, `step3` or meaningful names). Each named function should handle one step and call the next.
3. Compare your refactored version with another pair's. Are they the same? What's different? Which do you prefer and why?

*Targets: deep learning / recognising structural problems / transfer through comparison*

> **Teacher note:** Encourage pairs to argue for their design choice. There's no single right answer — the goal is to surface the trade-offs between approaches.

---

### Exercise 3 — The Broken Score Fetcher (Individual or pair, ~10 min)

**Context:** A live sports scores app uses the error-first callback convention. Something's off in this code:

```js
function getScore(matchId, callback) {
  fetchFromApi(matchId, function(data) {
    if (!data) {
      callback('No data found'); // 🤔
    } else {
      callback(data);            // 🤔
    }
  });
}

getScore('ajax-psv', function(err, score) {
  if (err) {
    console.log('Error:', err);
  } else {
    console.log('Score:', score);
  }
});
```

**Your task:**
1. Spot the bug(s). There are two violations of the error-first callback convention.
2. Fix the code so it follows the convention correctly.
3. Why does the *order* of the arguments matter? What could go wrong if it were flipped?

*Targets: misconception correction / conventions and why they exist*

> **Teacher note:** Students often mix up the argument order. This exercise is designed to make that mistake the lesson, not the embarrassment.

---

## After the Lesson (Homework)

### Homework 1 — Build Your Own Callback Chain (Individual)

**Context:** You're building a CLI tool for a film database. When a user searches for a director, the tool should:
1. Fetch the director's profile
2. Use the director's ID to fetch their filmography
3. Filter the results to only films after 2010
4. Log the titles

**Your task:**
1. Implement this using `simulateFetch(data, delay, callback)` — a function that mimics an async API call using `setTimeout`. You write this function yourself.
2. Chain the three steps using error-first callbacks.
3. Deliberately trigger an error at step 2 (e.g., return `null` as data) and make sure your error handling catches it and stops the chain.

Write at least 3–4 sentences reflecting: *What was the hardest part? What would make this easier?*

*Targets: transfer / independent construction / metacognition*

---

### Homework 2 — Callback vs. Real Life (Individual, open question)

Think of a real-world process outside of programming that works like a callback pattern — something where you *hand off* a task and say "call me back when it's done."

Some starting ideas: a pizza delivery notification, a sports betting alert, a cinema ticket confirmation email.

**Answer these questions:**
1. Describe the real-world example. Who is the "caller"? Who is the "callback"? What is the "async operation"?
2. What would go wrong if the process were synchronous — if you had to *stand and wait* for the result before doing anything else?
3. Now translate your example into pseudocode using the callback pattern. You don't need to write working JavaScript — readable pseudocode is fine.

*Targets: conceptual transfer / making meaning personal / argumentation*

---

### Homework 3 — Callback Hell Critique (Individual, open question)

Read this code:

```js
login(user, pass, function(err, session) {
  getProfile(session.id, function(err, profile) {
    getPreferences(profile.id, function(err, prefs) {
      getRecommendations(prefs, function(err, recs) {
        displayDashboard(recs);
      });
    });
  });
});
```

**Answer these questions:**
1. What are *three* concrete problems with this code? Be specific — don't just say "it's messy".
2. Refactor it using named functions. Show your solution.
3. Does the named-function refactor fully solve the problems you listed? Explain why or why not.
4. (Optional stretch) What would this look like with Promises? You don't need to get it perfect — sketch the idea.

*Targets: analysis / argumentation / foreshadowing Promises*
