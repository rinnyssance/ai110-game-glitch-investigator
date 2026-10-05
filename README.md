# 🎮 Game Glitch Investigator

**Submitted by:** Erin Joel Moore

Game Glitch Investigator is a CodePath AI Engineering debugging project where I investigated an AI-generated number guessing game that contained several logic errors.

Instead of building the game from scratch, my job was to play the broken game, identify unexpected behavior, investigate the code responsible for the glitches, use AI as a debugging assistant, refactor the game logic, and verify my fixes with automated and manual testing.

---

## 🔎 Project Overview

The application is a number guessing game built with Python and Streamlit.

The player selects a difficulty and tries to guess a randomly generated secret number. The game provides feedback indicating whether the guess is too high, too low, or correct.

However, the starter code intentionally contained bugs.

My goal was to investigate those bugs rather than immediately replacing the code.

My debugging process was:

1. Run and play the original game.
2. Observe unexpected behavior.
3. Record reproducible glitches.
4. Inspect the source code.
5. Identify likely causes.
6. Use AI assistance to evaluate possible fixes.
7. Refactor reusable logic into `logic_utils.py`.
8. Write automated tests.
9. Run `pytest`.
10. Play the repaired game manually to verify the final behavior.

---

## 🐛 Glitches I Found

### Bug 1: Contradictory Guess Hints

While playing the original game, I received hints that did not make logical sense.

For example, the game told me to:

- Guess higher than `101`
- Guess lower than `105`

I then guessed:

```text
102
```

and the game told me to go lower.

The feedback contradicted the previous clues.

After inspecting `check_guess()`, I found that the messages associated with `"Too High"` and `"Too Low"` were pointing in the wrong directions.

A guess that was too high could tell the player to go higher instead of lower.

---

### Bug 2: Secret Number Type Changed During the Game

The Developer Debug Info revealed an even larger problem.

During one game, the debug information showed:

```text
Secret: 19
```

but the game was giving me hints that made it seem like the secret was around `100`.

I found this code in the original game:

```python
if st.session_state.attempts % 2 == 0:
    secret = str(st.session_state.secret)
else:
    secret = st.session_state.secret
```

This converted the secret number into a string on alternating attempts.

Instead of consistently comparing integers, the program could end up trying to compare values such as:

```python
105
```

and:

```python
"19"
```

The starter code then caught the resulting `TypeError` and converted the guess into a string as well.

That caused some comparisons to behave like text comparisons instead of numeric comparisons.

I removed this behavior so that the secret remains numeric throughout the game.

---

### Bug 3: Attempt Counter Was Off by One

The Developer Debug Info also showed:

```text
Attempts: 7
```

while the history contained only six guesses:

```text
50
75
105
100
102
101
```

The cause was the initial attempt value:

```python
st.session_state.attempts = 1
```

The game therefore counted one attempt before the player had submitted any guesses.

I changed the initial value to:

```python
st.session_state.attempts = 0
```

Now the attempt counter reflects the number of guesses actually submitted.

---

## 🛠️ Additional Improvements

While investigating the original bugs, I found several related areas that could be made more consistent.

### Difficulty-Aware Range Display

The interface originally displayed:

```text
Guess a number between 1 and 100.
```

even though different difficulty levels use different ranges.

The application now displays the actual range using:

```python
f"Guess a number between {low} and {high}."
```

---

### New Game Reset

The New Game button originally generated a number between `1` and `100` regardless of the selected difficulty.

The repaired version uses:

```python
random.randint(low, high)
```

The New Game action also resets:

- attempts
- score
- game status
- guess history

This gives the player a clean state for each new game.

---

### Invalid Input

Input parsing remains separate from the interface logic.

Invalid input displays an error instead of being processed as a valid numeric guess.

---

## 🧩 Refactoring

The original application contained reusable game logic directly inside `app.py`.

I moved the following functions into:

```text
logic_utils.py
```

The refactored functions are:

```python
get_range_for_difficulty()
parse_guess()
check_guess()
update_score()
```

`app.py` now imports these functions:

```python
from logic_utils import (
    get_range_for_difficulty,
    parse_guess,
    check_guess,
    update_score,
)
```

This separates the game rules from the Streamlit interface and makes the logic easier to test independently.

---

## 🧪 Automated Testing

I used `pytest` to test the refactored `check_guess()` function.

The tests verify three important situations.

### Correct Guess

```python
outcome, message = check_guess(50, 50)

assert outcome == "Win"
assert message == "🎉 Correct!"
```

### Guess Too High

```python
outcome, message = check_guess(60, 50)

assert outcome == "Too High"
assert message == "📉 Go LOWER!"
```

### Guess Too Low

```python
outcome, message = check_guess(40, 50)

assert outcome == "Too Low"
assert message == "📈 Go HIGHER!"
```

---

## ⚙️ Pytest Configuration

When I initially ran:

```bash
pytest
```

pytest could find the test file but could not import `logic_utils`.

The error was:

```text
ModuleNotFoundError: No module named 'logic_utils'
```

I added a `pytest.ini` file to the project root:

```ini
[pytest]
pythonpath = .
```

After correcting the import configuration, pytest successfully discovered and executed the tests.

---

## ✅ Test Results

Final automated test result:

```text
3 passed
```

I also manually ran the Streamlit application after the repairs and successfully guessed the correct secret number and won the game.

This allowed me to verify the project in two ways:

- automated testing with `pytest`
- manual testing through the Streamlit interface

---

## 🤖 How I Used AI

I used ChatGPT as a debugging assistant during the project.

AI helped me:

- reason through unexpected game behavior
- inspect suspicious sections of the starter code
- understand the integer-versus-string comparison problem
- refactor reusable functions into `logic_utils.py`
- improve the existing pytest tests
- diagnose the pytest import error
- review whether proposed changes matched the behavior I observed

I did not treat the AI's suggestions as automatically correct.

I compared the suggestions with the code, tested the application myself, ran automated tests, and verified the final behavior manually.

More information about my use of AI for test generation is documented in:

```text
ai_interactions.md
```

---

## 💡 What I Learned

This project showed me that code can run without actually behaving correctly.

The original game launched successfully, accepted guesses, displayed messages, and tracked state. However, several parts of its internal logic were incorrect.

The most useful part of the assignment was learning to separate:

```text
"The program runs"
```

from:

```text
"The program behaves correctly"
```

I also gained more experience with:

- debugging Python
- reading existing code
- identifying logic errors
- Python data types
- Streamlit session state
- refactoring
- modular Python code
- writing pytest tests
- troubleshooting imports
- validating AI-generated code
- Git and GitHub workflows

AI was useful for suggesting explanations and possible repairs, but testing was what demonstrated whether those repairs actually worked.

---

## 📁 Project Structure

```text
ai110-module1show-gameglitchinvestigator-starter/
│
├── app.py
├── logic_utils.py
├── pytest.ini
├── requirements.txt
├── README.md
├── reflection.md
├── ai_interactions.md
│
└── tests/
    └── test_game_logic.py
```

---

## 🚀 Running the Project

### 1. Clone the repository

```bash
git clone <repository-url>
```

Enter the project:

```bash
cd ai110-game-glitch-investigator
```

---

### 2. Create a virtual environment

```bash
python -m venv .venv
```

On Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

---

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

---

### 4. Run the game

```bash
streamlit run app.py
```

---

## 🧪 Running the Tests

From the project root, run:

```bash
pytest
```

My final result was:

```text
3 passed
```

---

## 🧰 Technologies Used

- Python
- Streamlit
- pytest
- Git
- GitHub
- VS Code
- ChatGPT

---

## 🎯 Final Result

I successfully investigated the intentionally buggy starter application, identified multiple glitches, repaired the game logic, separated reusable logic from the Streamlit interface, added automated tests, and manually verified that the repaired game works.

The final test suite passes:

```text
3 passed
```

and I was able to successfully complete and win the repaired game.