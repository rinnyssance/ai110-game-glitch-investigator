# 💭 Reflection: Game Glitch Investigator

## 1. What was broken when you started?

When I first ran the game, it looked like a normal number guessing game, but I quickly realized that the hints did not make sense. At one point, the game told me to go higher than 101 and lower than 105, but when I guessed 102, it told me to go lower. The developer information later showed that the secret number was actually 19, which made the hints even more confusing. I also noticed that the game said I had made 7 attempts even though my history only contained 6 guesses.

### Bug Reproduction Log

| Input | Expected Behavior | Actual Behavior | Console Output / Error |
|-------|-------------------|-----------------|------------------------|
| Guessed `102` after being told to go higher than `101` and lower than `105` | The hint should consistently narrow the range based on the secret number | The game told me to go lower even though the previous hints were contradictory | No console error; incorrect hint displayed in the Streamlit interface |
| Guesses around `100` while Developer Debug Info showed `Secret: 19` | Any guess greater than 19 should be identified as too high and tell me to go lower | The game produced hints that made it seem like the secret was near 100 | No console error; the secret was being converted between an integer and string |
| 6 guesses entered | Attempts should display `6` | Developer Debug Info displayed `Attempts: 7` | No console error; attempt counter started at 1 instead of 0 |

---

## 2. How did you use AI as a teammate?

I used ChatGPT as my AI teammate while debugging the project. One correct suggestion was to inspect the code that converted the secret number into a string on alternating attempts. After removing that behavior and keeping the secret as an integer, I manually played the game again and was able to receive consistent hints and eventually guess the correct number.

I also did not accept every suggestion as something that automatically belonged in the project. For example, while reviewing the code, AI identified additional possible improvements such as changing the hard-coded difficulty display and resetting more game state when starting a new game. I treated those as separate improvements instead of confusing them with the three bugs I personally reproduced during the initial glitch hunt. I verified the changes I did use by running the application myself and later running the automated tests.

---

## 3. Debugging and testing your fixes

I decided that a bug was fixed only after I could reproduce the situation again without getting the original incorrect behavior. I manually ran the Streamlit application and played the game after changing the logic, and I was able to correctly guess the secret number and win. I also used pytest to test `check_guess()` with a correct guess, a guess that was too high, and a guess that was too low.

At first, pytest did not run because it could not find `logic_utils`, so I had to troubleshoot the import configuration before I could test the actual game logic. After fixing that issue, all three tests passed. AI helped me understand that `check_guess()` returned both an outcome and a message, so my tests needed to verify both values instead of expecting only `"Win"`, `"Too High"`, or `"Too Low"`.

My final pytest result was:

```text
3 passed
```

---

## 4. What did you learn about Streamlit and state?

I would explain a Streamlit rerun as the application basically running the Python script again whenever the user interacts with something on the page. Because of that, normal variables cannot always be relied on to remember what happened during previous interactions. `st.session_state` gives the application a place to remember values such as the secret number, score, attempt count, game status, and guess history between reruns.

This project helped me understand why state management matters even in a relatively small application. A small mistake in how a state value is initialized or changed can affect the game across multiple interactions, which is what happened with the attempt counter.

---

## 5. Looking ahead: your developer habits

One habit I want to reuse is reproducing a bug before changing the code. Writing down what I entered, what I expected, and what actually happened made it much easier to compare the behavior with the source code. I also want to continue writing tests for important logic instead of relying only on whether an application appears to work.

Next time I work with AI on a coding task, I want to be even more deliberate about asking it to explain why a change is necessary before applying the change. I found that AI was useful for identifying suspicious code and helping with tests, but I still needed to run the program and decide whether its suggestions matched what I was actually seeing.

This project changed the way I think about AI-generated code because code can look reasonable and run successfully while still being logically wrong. AI can help debug another AI's code, but I still have to understand, test, and verify the result myself.