# AI Interactions Log

> **Stretch features only.** This file documents the stretch feature I attempted while completing Game Glitch Investigator.

---

## Test Generation (SF7)

### Overview

I used ChatGPT to help review and improve the tests for the refactored game logic.

During the debugging process, I moved reusable functions from `app.py` into `logic_utils.py`. One of those functions was `check_guess()`, which compares the player's guess with the secret number and returns both an outcome and a message.

The function returns a tuple in this format:

```python
(outcome, message)
```

For example:

```python
("Too High", "📉 Go LOWER!")
```

The original tests expected `check_guess()` to return only a string such as:

```python
"Win"
```

However, the actual function returns:

```python
("Win", "🎉 Correct!")
```

I used AI assistance to identify this mismatch and rewrite the tests so that they check both the outcome and the user-facing hint.

---

## Tests Generated and Improved

| Edge Case | Prompt / Question Used | AI-Suggested Test | Did It Pass? | My Reasoning |
|-----------|------------------------|-------------------|--------------|--------------|
| Correct guess | I asked AI to help update the existing `check_guess()` tests after refactoring the game logic. | Verify that `check_guess(50, 50)` returns `"Win"` and `"🎉 Correct!"`. | Yes | If the guess and secret are equal, the game should recognize the guess as correct. |
| Guess too high | I asked AI to correct the test for a guess that is greater than the secret. | Verify that `check_guess(60, 50)` returns `"Too High"` and `"📉 Go LOWER!"`. | Yes | This directly tests one of the glitches I observed. A guess above the secret should tell the player to guess lower. |
| Guess too low | I asked AI to correct the test for a guess that is less than the secret. | Verify that `check_guess(40, 50)` returns `"Too Low"` and `"📈 Go HIGHER!"`. | Yes | A guess below the secret should tell the player to guess higher. |

---

## Final Test Code

The final tests in `tests/test_game_logic.py` were:

```python
from logic_utils import check_guess


def test_winning_guess():
    outcome, message = check_guess(50, 50)

    assert outcome == "Win"
    assert message == "🎉 Correct!"


def test_guess_too_high():
    outcome, message = check_guess(60, 50)

    assert outcome == "Too High"
    assert message == "📉 Go LOWER!"


def test_guess_too_low():
    outcome, message = check_guess(40, 50)

    assert outcome == "Too Low"
    assert message == "📈 Go HIGHER!"
```

---

## Pytest Setup Issue

When I first ran:

```bash
pytest
```

pytest found the test file but could not import `logic_utils.py`.

The error included:

```text
ModuleNotFoundError: No module named 'logic_utils'
```

I used AI assistance to investigate the import problem.

I added a `pytest.ini` file to the project root containing:

```ini
[pytest]
pythonpath = .
```

This tells pytest to include the project root in Python's import path.

After making this change, pytest was able to import `logic_utils` successfully.

---

## Final Test Result

I ran:

```bash
pytest
```

The final result was:

```text
3 passed
```

All three tests passed successfully.

---

## Manual Verification

I also manually tested the Streamlit application after refactoring and repairing the game logic.

I ran:

```bash
streamlit run app.py
```

I played the game using the repaired version and was able to correctly guess the secret number and win.

This gave me two forms of verification:

1. Automated testing with `pytest`
2. Manual testing through the Streamlit interface

---

## What I Accepted From AI

I accepted the suggestion to test both values returned by `check_guess()` instead of testing only the outcome.

For example, instead of only checking:

```python
assert outcome == "Too High"
```

the test also checks:

```python
assert message == "📉 Go LOWER!"
```

I thought this was useful because one of the bugs I discovered was specifically related to incorrect hint messages. Testing the message helps prevent that bug from returning later.

I also accepted the suggestion to configure pytest so that it could locate `logic_utils.py`.

---

## What I Verified Myself

I did not rely only on the AI-generated suggestions.

I verified that:

- `check_guess()` actually returns two values.
- A guess greater than the secret produces `"Too High"`.
- A high guess tells the player to go lower.
- A guess less than the secret produces `"Too Low"`.
- A low guess tells the player to go higher.
- A correct guess produces `"Win"`.
- The Streamlit game still runs after the refactor.
- I could successfully win the repaired game.
- All three automated tests passed.

---

## What I Learned

This process showed me why AI-generated code and AI-generated tests still need human review.

The original game could run even though its behavior was incorrect. I had to play the game, compare the output with what I expected, inspect the underlying logic, and then use tests to verify the repairs.

I also learned that a test has to match the actual interface of the function it is testing. The original tests expected a single string, while `check_guess()` returned a tuple containing both an outcome and a message.

AI helped me identify and repair the problems, but I still had to understand the suggestions, decide whether they made sense, run the program, and verify the results myself.