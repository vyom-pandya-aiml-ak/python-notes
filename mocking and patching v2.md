# 🎬 Python Mocking & Patching: The Ultimate Master Document

> **“A test is only as good as its isolation. Master the mock, master the test.”**

-----

## 📋 Table of Contents

- [Introduction: The Movie Set Philosophy](#introduction-the-movie-set-philosophy)
- [Module 1: Mock vs. MagicMock — The Basics](#module-1-mock-vs-magicmock--the-basics)
- [Module 2: Strictness Control — spec & autospec](#module-2-strictness-control--spec--autospec)
- [Module 3: AsyncMock — Testing the Async World](#module-3-asyncmock--testing-the-async-world)
- [Module 4: The Patching Namespace — Where to Patch](#module-4-the-patching-namespace--where-to-patch)
- [Module 5: Scripting Behavior — return_value vs side_effect](#module-5-scripting-behavior--return_value-vs-side_effect)
- [Module 6: Spying with wraps](#module-6-spying-with-wraps)
- [Module 7: Monkeypatching — The pytest Surgical Tool](#module-7-monkeypatching--the-pytest-surgical-tool)
- [Module 8: Comparison Tables](#module-8-comparison-tables)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

-----

## Introduction: The Movie Set Philosophy

Before we write a single line of code, we need a mental model. Welcome to **The Movie Set Analogy** — the framework that makes every mocking concept click.

|Role                     |Testing Equivalent                 |What It Does                                   |
|-------------------------|-----------------------------------|-----------------------------------------------|
|🎬 **Director**           |`patch()` decorator/context manager|Controls *what* gets replaced and *when*       |
|🦸 **Lead Actor**         |Real production code/object        |The thing you’re actually testing              |
|🥋 **Stunt Double**       |`Mock` / `MagicMock` / `AsyncMock` |A safe stand-in that won’t break the scene     |
|📝 **Script Revision**    |`return_value` / `side_effect`     |Tells the stunt double exactly how to perform  |
|🔍 **On-Set Spy**         |`wraps` parameter                  |Watches the real actor without replacing them  |
|✂️ **Quick Script Change**|`monkeypatch` fixture              |A last-minute line change while filming is live|

Keep this map in your head. Every module below maps directly onto it.

-----

## Module 1: Mock vs. MagicMock — The Basics

### 📖 Plain English Definition

A **`Mock`** is a blank, programmable impersonator. You hand it to your code in place of a real object, and it silently accepts any attribute access or method call you throw at it — recording everything for later inspection. A **`MagicMock`** is the same thing, but pre-wired with support for Python’s special “dunder” methods (like `__len__`, `__iter__`, `__getitem__`), so it can also impersonate containers, context managers, and other protocol-heavy objects.

### 🛠️ Ways to Use This

- **Mocking a simple service call**: Replace a `PaymentService` object with a `Mock` to verify your checkout function calls `.charge()` without actually charging anyone.
- **Mocking a database result set**: Use a `MagicMock` when your code does `for row in db_cursor:` — the `__iter__` protocol makes iteration work out of the box.
- **Mocking a context manager**: Use `MagicMock` when testing code wrapped in a `with open(...) as f:` block — it supports `__enter__` and `__exit__` automatically.

### 🎬 Movie Set Analogy

A `Mock` is a **generic stunt double**: they can stand on the set and take direction, but if the scene requires them to swim, juggle, or speak fluent French, you need a specialist. A `MagicMock` is the **full-package stunt performer**: same stand-in capability, but pre-trained in every “special skill” Python’s data model requires.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module1_mock_vs_magicmock.py
# PURPOSE: Understand when to reach for Mock vs. MagicMock
# ============================================================

from unittest.mock import Mock, MagicMock


# --- SECTION 1: The Simple Mock ---
# Use Mock when the real object has no dunder/protocol magic.
# Think: service clients, API wrappers, simple utility classes.

def send_welcome_email(email_service, user_email):
    """A simple function that depends on an external email service."""
    # This is the unit under test. We don't want real emails flying around.
    email_service.send(to=user_email, subject="Welcome!")


def test_send_welcome_email_calls_service():
    # Create a blank impersonator — a stunt double with no special skills needed.
    mock_service = Mock()

    # Call our real function, injecting the fake service instead of the real one.
    send_welcome_email(mock_service, "user@example.com")

    # .assert_called_once_with() is the key payoff:
    # It proves our function actually called .send() with the RIGHT arguments.
    # Without this, the test would pass even if send() was never called!
    mock_service.send.assert_called_once_with(to="user@example.com", subject="Welcome!")


# --- SECTION 2: Why MagicMock? Enter the Dunder Methods ---
# Dunder methods (Double UNDERscore) are Python's protocol hooks:
# __len__ (used by len()), __iter__ (used by for loops), __getitem__ (used by []).
# A plain Mock does NOT have these pre-configured. MagicMock does.

def count_active_users(user_repository):
    """Returns how many users are in the repository."""
    # len() internally calls user_repository.__len__()
    # A plain Mock would raise: TypeError: object of type 'Mock' has no len()
    return len(user_repository)


def test_count_active_users_with_magicmock():
    # MagicMock pre-configures __len__ so len() works without extra setup.
    mock_repo = MagicMock()

    # Tell the mock what __len__ should return — i.e., "pretend there are 42 users."
    mock_repo.__len__.return_value = 42

    result = count_active_users(mock_repo)

    # Verify our function correctly returned the value from __len__
    assert result == 42


# --- SECTION 3: MagicMock as a Context Manager ---
# When your code uses `with some_object as x:`, that object must support
# __enter__ (called on entry) and __exit__ (called on exit).
# MagicMock handles both automatically, which is crucial for patching file I/O.

def read_config(file_path):
    """Reads a config file and returns its content."""
    # This `with open(...)` block requires __enter__ and __exit__ support.
    with open(file_path) as f:
        return f.read()


def test_read_config(mocker):
    # mocker.patch is pytest-mock's version of unittest.mock.patch.
    # It patches the built-in `open` function in the test's scope.
    mock_open = mocker.patch("builtins.open", MagicMock())

    # Configure what .read() should return when called inside the `with` block.
    # mock_open.return_value.__enter__.return_value is the `f` in `with open() as f`.
    mock_open.return_value.__enter__.return_value.read.return_value = "theme: dark"

    result = read_config("config.yaml")

    # Verify our function correctly passes through the file's content.
    assert result == "theme: dark"


# --- SECTION 4: Plain Mock FAILS on Dunder Methods ---
# This is the single most important distinction. Knowing this prevents hours of debugging.

def test_why_plain_mock_fails_on_len():
    plain_mock = Mock()

    try:
        # This will raise TypeError because plain Mock doesn't pre-configure __len__.
        # Python's len() function requires a real, callable __len__ method.
        length = len(plain_mock)
    except TypeError as e:
        # This is EXPECTED behavior — it confirms our understanding.
        print(f"Confirmed: Plain Mock fails with: {e}")

    # THE FIX: Use MagicMock, which pre-configures __len__.
    magic_mock = MagicMock()
    magic_mock.__len__.return_value = 5
    assert len(magic_mock) == 5  # This works perfectly.
```

> **💡 Expert Tip:** Default to `MagicMock` — it is a strict superset of `Mock`. The only reason to use plain `Mock` is when you *want* to explicitly confirm that no magic method protocols are needed, making your test intent clearer.

> **⚠️ Danger Zone:** Never rely on a `Mock` silently swallowing an unexpected attribute access as a sign that everything is okay. `mock.any_attribute_ever` will always return another `Mock` without raising an error. This is why Module 2 (spec/autospec) exists.

-----

## Module 2: Strictness Control — spec & autospec

### 📖 Plain English Definition

By default, a `Mock` will happily respond to *any* attribute or method call you throw at it — even ones that don’t exist on the real object. **`spec`** and **`autospec`** are guardrails that lock a mock down to only the interface that actually exists on the real class. If your test code accidentally calls a method that doesn’t exist in production, the mock raises an `AttributeError` immediately — catching the bug in the test suite rather than in production.

### 🛠️ Ways to Use This

- **Preventing typo-induced silent failures**: If your real class has `.connect()` but you accidentally call `.conect()` in your test, `autospec` catches it instantly. A plain `Mock` would silently create a new fake `.conect()` attribute and your test would pass incorrectly.
- **Enforcing correct method signatures**: `autospec` validates that mocked methods are called with the correct number and names of arguments, mirroring the real function’s signature.
- **Documenting the interface boundary**: Using `spec=RealClass` in a test communicates clearly to future readers: “This mock is supposed to behave like *this specific class*.”

### 🎬 Movie Set Analogy

A plain `Mock` is a stunt double with **no script constraints** — they’ll do anything you ask, even things that are physically impossible for the character. A `spec`’d mock is a stunt double who has actually **read the script**: they know exactly what the character can and cannot do, and will refuse (loudly) if you ask them to do something out-of-character.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module2_spec_autospec.py
# PURPOSE: Use spec and autospec to prevent silent test failures
# ============================================================

from unittest.mock import Mock, MagicMock, create_autospec, patch
import pytest


# --- The Real Class We're Mocking ---
class DatabaseConnection:
    """A real database connection class with a defined interface."""

    def __init__(self, host: str, port: int):
        self.host = host
        self.port = port

    def connect(self) -> bool:
        """Establishes the connection. Returns True on success."""
        pass  # In reality, this would open a socket.

    def query(self, sql: str, params: tuple = ()) -> list:
        """Executes a query and returns rows."""
        pass  # In reality, this would hit the database.

    def disconnect(self):
        """Closes the connection gracefully."""
        pass


# --- SECTION 1: The "Silent Failure" Problem ---
# This test PASSES but is WRONG. It gives false confidence.

def test_demonstrates_silent_failure_with_plain_mock():
    plain_mock = Mock()  # No spec — it's a blank impersonator.

    # BUG: The real method is `connect()`, not `conect()`.
    # A plain Mock silently creates a new fake attribute called `conect`.
    plain_mock.conect()  # This should ERROR but does NOT!

    # This call succeeds even though `conect` isn't in the real interface.
    # Our production code would blow up with AttributeError, but the test says green.
    plain_mock.conect.assert_called_once()  # Passes — but it's a lie.


# --- SECTION 2: Using spec= to Catch the Bug ---

def test_spec_catches_attribute_errors():
    # `spec=DatabaseConnection` tells the Mock: "only allow attributes
    # that actually exist on DatabaseConnection." This is the guard we need.
    specced_mock = Mock(spec=DatabaseConnection)

    # Now, calling the typo `.conect()` raises AttributeError immediately.
    # The bug is caught at test-time, not at production-time.
    with pytest.raises(AttributeError):
        specced_mock.conect()  # ✅ Correctly raises an error now.

    # The correct method name works fine.
    specced_mock.connect()  # ✅ This succeeds because `connect` exists.


# --- SECTION 3: create_autospec — The Gold Standard ---
# `spec=` checks attribute NAMES but NOT method SIGNATURES.
# `create_autospec` goes further: it validates the call SIGNATURE too.

def test_autospec_validates_signatures():
    # create_autospec mirrors the real class completely — names AND signatures.
    auto_mock = create_autospec(DatabaseConnection, instance=True)
    # `instance=True` means we're mocking an *instance* of the class,
    # not the class itself (so we don't need to pass `self` manually).

    # Correct call — this works fine.
    auto_mock.query("SELECT * FROM users", params=())

    # Wrong call — `query()` doesn't accept a `limit` keyword argument.
    # create_autospec enforces the real signature and raises TypeError.
    with pytest.raises(TypeError):
        auto_mock.query("SELECT *", limit=10)  # ✅ Correctly caught!


# --- SECTION 4: autospec via patch() in a real test ---

class UserService:
    """Our production service that uses DatabaseConnection."""

    def get_user(self, db: DatabaseConnection, user_id: int):
        """Fetches a single user from the database."""
        db.connect()
        rows = db.query("SELECT * FROM users WHERE id = ?", params=(user_id,))
        db.disconnect()
        return rows[0] if rows else None


def test_get_user_with_autospec(mocker):
    # Using autospec=True in patch() applies autospec automatically.
    # This is the most common, idiomatic pattern in professional codebases.
    mock_db = mocker.patch(
        "__main__.DatabaseConnection",  # Patch the class in this module's namespace.
        autospec=True
    )

    # Configure the mock instance's `query` to return a fake row.
    mock_db.return_value.query.return_value = [{"id": 1, "name": "Alice"}]

    service = UserService()
    db_instance = DatabaseConnection("localhost", 5432)  # Gets the mocked instance.
    result = service.get_user(db_instance, user_id=1)

    # Verify the correct methods were called in the correct order.
    db_instance.connect.assert_called_once()
    db_instance.query.assert_called_once_with(
        "SELECT * FROM users WHERE id = ?", params=(1,)
    )
    db_instance.disconnect.assert_called_once()
    assert result == {"id": 1, "name": "Alice"}
```

> **💡 Expert Tip:** Make `create_autospec` or `autospec=True` your *default* choice when patching classes. The tiny bit of extra setup pays dividends in test reliability. Reserve plain `Mock()` for the simplest, most obvious use cases.

> **⚠️ Danger Zone:** `spec=` only validates the *shape* of the interface (attribute names). It does **not** validate call signatures. Only `create_autospec` / `autospec=True` validates both. Don’t mistake one for the other.

-----

## Module 3: AsyncMock — Testing the Async World

### 📖 Plain English Definition

When Python runs `await some_function()`, it expects `some_function()` to return a **coroutine** — a special object that the event loop can pause and resume. A regular `Mock()` returns a plain value, not a coroutine, which causes Python to raise a `TypeError` the moment your `async` function tries to `await` it. **`AsyncMock`** is a mock specifically designed to return a coroutine when called, making it fully compatible with `async/await` code.

### 🛠️ Ways to Use This

- **Mocking async API clients**: Your function `async def fetch_weather(client):` awaits `client.get(url)` — use `AsyncMock` for the client so the `await` resolves correctly.
- **Simulating async database calls**: SQLAlchemy async sessions, asyncpg, and similar tools use `await`. Replace them with `AsyncMock` to test your data layer without a real database.
- **Testing async error handling**: Use `AsyncMock(side_effect=TimeoutError())` to verify your retry logic handles network timeouts correctly.

### 🎬 Movie Set Analogy

Regular `Mock` is a stunt double trained for **synchronous scenes** — they can stand, walk, and talk in normal time. But your async code is a **wire-work flying scene**: the stunt double needs special rigging equipment (the event loop) to even be on set. `AsyncMock` is the stunt double who comes pre-rigged for wire-work — they plug directly into the event loop without breaking the scene.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module3_asyncmock.py
# PURPOSE: Test async/await functions correctly using AsyncMock
# ============================================================

import asyncio
import pytest
from unittest.mock import AsyncMock, MagicMock, patch


# --- The Async Production Code We're Testing ---

class AsyncWeatherClient:
    """An async HTTP client for a weather API."""

    async def get_temperature(self, city: str) -> dict:
        """Makes a real async network call. We never want this in tests."""
        pass  # Would use aiohttp or httpx in reality.


async def get_weather_report(client: AsyncWeatherClient, city: str) -> str:
    """
    Our real async function under test.
    It calls an async dependency, so we need AsyncMock to test it.
    """
    # `await` suspends this coroutine until `get_temperature` resolves.
    data = await client.get_temperature(city)
    temp = data.get("temp_c", "unknown")
    return f"The temperature in {city} is {temp}°C"


# --- SECTION 1: Why Regular Mock FAILS in Async Code ---

@pytest.mark.asyncio  # Tells pytest this test runs an async event loop.
async def test_why_regular_mock_fails():
    """This test documents the failure — run it to see the TypeError."""
    regular_mock = MagicMock()  # A normal, non-async mock.

    try:
        # MagicMock().get_temperature() returns a MagicMock, NOT a coroutine.
        # `await` on a non-coroutine raises TypeError.
        await get_weather_report(regular_mock, "Paris")
    except TypeError as e:
        # This confirms exactly why we need AsyncMock.
        print(f"Confirmed failure: {e}")
        # TypeError: object MagicMock can't be used in 'await' expression


# --- SECTION 2: The Correct Fix — AsyncMock ---

@pytest.mark.asyncio
async def test_get_weather_report_success():
    # AsyncMock is designed specifically for async code.
    # When called, it returns a coroutine that resolves to `return_value`.
    mock_client = AsyncMock()

    # Configure what the awaited call should resolve to.
    # This is the "Script Revision" — we're telling the stunt double their lines.
    mock_client.get_temperature.return_value = {"temp_c": 22, "condition": "sunny"}

    # Now `await client.get_temperature(city)` will resolve cleanly.
    result = await get_weather_report(mock_client, "Paris")

    # Verify the correct output was produced from our mocked data.
    assert result == "The temperature in Paris is 22°C"

    # Verify the mock was called with the right argument.
    # This proves our function passed the correct city to the client.
    mock_client.get_temperature.assert_awaited_once_with("Paris")
    # Note: use assert_awaited_once_with(), NOT assert_called_once_with().
    # The `awaited` variant confirms the coroutine was actually awaited,
    # not just called (which would leave the coroutine un-executed).


# --- SECTION 3: AsyncMock with side_effect for Error Testing ---

@pytest.mark.asyncio
async def test_get_weather_report_handles_network_error():
    mock_client = AsyncMock()

    # side_effect on AsyncMock causes the coroutine to RAISE the exception
    # when it's awaited. This is how we test our error-handling paths.
    mock_client.get_temperature.side_effect = ConnectionError("Network unreachable")

    # Our function should propagate the error (or catch it — test either path).
    with pytest.raises(ConnectionError, match="Network unreachable"):
        await get_weather_report(mock_client, "London")


# --- SECTION 4: Using patch() with AsyncMock ---

@pytest.mark.asyncio
async def test_patching_async_method_with_patch():
    # When using patch() for async methods, you must explicitly pass
    # `new_callable=AsyncMock`, or the patch will create a regular Mock.
    # Since Python 3.8, patch() auto-detects async functions and uses AsyncMock,
    # but being explicit is clearer and more robust.
    with patch.object(
        AsyncWeatherClient,
        "get_temperature",
        new_callable=AsyncMock,  # Explicit is better than implicit.
        return_value={"temp_c": -5}
    ) as mock_method:
        client = AsyncWeatherClient()
        result = await get_weather_report(client, "Oslo")

        assert result == "The temperature in Oslo is -5°C"
        mock_method.assert_awaited_once_with("Oslo")


# --- SECTION 5: AsyncMock as a Full Async Context Manager ---

@pytest.mark.asyncio
async def test_async_context_manager():
    """Test code that uses `async with` — e.g., an async DB session."""

    # AsyncMock supports __aenter__ and __aexit__ for `async with` blocks.
    mock_session = AsyncMock()

    # Configure what the session returns when used as `async with session as s:`
    mock_session.__aenter__.return_value = mock_session
    mock_session.execute.return_value = [{"id": 1}]

    async with mock_session as session:
        rows = await session.execute("SELECT 1")
        assert rows == [{"id": 1}]

    # Verify the context manager was entered and exited cleanly.
    mock_session.__aenter__.assert_awaited_once()
    mock_session.__aexit__.assert_awaited_once()
```

> **💡 Expert Tip:** Use `assert_awaited_once_with()` (not `assert_called_once_with()`) for async mocks. The distinction matters: `called` only proves the function was invoked; `awaited` proves the coroutine actually ran. An un-awaited coroutine is a silent bug — Python will warn about it but won’t stop your test from passing.

> **⚠️ Danger Zone:** Forgetting `@pytest.mark.asyncio` on an async test function will cause it to silently succeed without ever executing the async body. Always use `asyncio.run()` in plain `unittest` tests, or `pytest-asyncio` with the marker in pytest.

-----

## Module 4: The Patching Namespace — Where to Patch

### 📖 Plain English Definition

Patching works by replacing a **name** in a **namespace** (a module’s dictionary of names). When you `import requests` in your code, Python creates the name `requests` *in your module’s namespace* — it’s like a local alias. When you patch, you must replace that local alias, not the original object in the `requests` module. The rule is: **patch where the name is used, not where it is defined**.

### 🛠️ Ways to Use This

- **Patching a top-level import**: Your module does `import os`. Patch `your_module.os`, not `os`.
- **Patching a function imported directly**: Your module does `from datetime import datetime`. Patch `your_module.datetime`, not `datetime.datetime`.
- **Patching a class method**: Your module does `from mylib import MyClass`. Patch `your_module.MyClass.method_name`.

### 🎬 Movie Set Analogy

Think of Python’s import system as a **film crew directory**. When the production office (your module) hires a stunt driver (imports `requests`), they get that driver’s *phone number stored in their own contact list*. If the Director (patcher) wants to swap in a different stunt driver, they must update **the production office’s contact list** — not the driver’s original listing in the talent agency’s database. Patching the source is like calling the talent agency instead: the production office’s copy is unchanged and they’ll still call the original driver.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: payment_processor.py (Production Code)
# PURPOSE: This is the module we want to TEST.
# ============================================================

# When Python executes this line, it creates the name `requests`
# in the `payment_processor` module's namespace.
# Think of it as: payment_processor.requests = <the requests library>
import requests

# Similarly, `datetime` is now a local name in this module.
from datetime import datetime


def charge_customer(amount: float, card_token: str) -> dict:
    """Sends a charge request to a payment API."""
    # This call uses `requests` as it exists in THIS module's namespace.
    response = requests.post(
        "https://api.payments.com/charge",
        json={"amount": amount, "token": card_token}
    )
    return response.json()


def get_current_timestamp() -> str:
    """Returns the current timestamp as a string."""
    # This uses `datetime` as it exists in THIS module's namespace.
    return datetime.now().isoformat()
```

```python
# ============================================================
# FILE: test_module4_patching_namespace.py
# PURPOSE: Demonstrate the "patch where it's USED" rule
# ============================================================

import pytest
from unittest.mock import patch, MagicMock


# --- SECTION 1: The WRONG Way to Patch (Common Beginner Mistake) ---

def test_WRONG_patching_the_source():
    # ❌ WRONG: Patching `requests.post` at the SOURCE module.
    # This changes the `post` attribute inside the `requests` library itself,
    # but `payment_processor.py` already has its OWN reference to `requests`.
    # The code in payment_processor will still use the original `requests.post`.
    with patch("requests.post") as mock_post:  # This OFTEN won't work!
        mock_post.return_value.json.return_value = {"status": "charged"}
        # If this test passes, it's probably coincidental for this structure.
        # It will silently FAIL if the import structure changes.
        pass


# --- SECTION 2: The RIGHT Way to Patch (The Golden Rule) ---

def test_CORRECT_patching_the_destination():
    # ✅ CORRECT: Patch `requests` as it exists in `payment_processor`'s namespace.
    # Format: "module_where_name_is_USED.name_to_replace"
    with patch("payment_processor.requests") as mock_requests:
        # Now we control the `requests` object that payment_processor.py sees.
        mock_response = MagicMock()
        mock_response.json.return_value = {"status": "charged", "id": "ch_123"}

        # `requests.post` in payment_processor will now return our mock response.
        mock_requests.post.return_value = mock_response

        # Import the function here to use the already-patched namespace.
        from payment_processor import charge_customer
        result = charge_customer(29.99, "tok_visa_abc")

        # Verify the mock was called with the correct API endpoint and payload.
        mock_requests.post.assert_called_once_with(
            "https://api.payments.com/charge",
            json={"amount": 29.99, "token": "tok_visa_abc"}
        )
        assert result == {"status": "charged", "id": "ch_123"}


# --- SECTION 3: Patching a `from X import Y` Style Import ---
# This is the trickiest case. `from datetime import datetime` creates a
# NEW local name `datetime` directly in payment_processor's namespace.
# You must patch THAT local name, not `datetime.datetime`.

def test_patching_from_import():
    # ✅ CORRECT: The local name `datetime` inside `payment_processor` is what we target.
    with patch("payment_processor.datetime") as mock_datetime:
        # Configure the mock's `.now()` method to return a fixed moment in time.
        # This gives us a "frozen clock" for deterministic testing.
        mock_datetime.now.return_value.isoformat.return_value = "2024-01-15T10:30:00"

        from payment_processor import get_current_timestamp
        result = get_current_timestamp()

        assert result == "2024-01-15T10:30:00"
        mock_datetime.now.assert_called_once()


# --- SECTION 4: Using the @patch Decorator (Equivalent to `with patch()`) ---
# The decorator form is syntactically cleaner for single-patch tests.
# Patches are applied bottom-up when stacking decorators.

@patch("payment_processor.requests")  # Applied second (outer)
@patch("payment_processor.datetime")  # Applied first (inner)
def test_multiple_patches_as_decorators(mock_datetime, mock_requests):
    # Note the argument order: BOTTOM decorator's mock comes FIRST as an argument.
    # This is a common source of confusion — always match order carefully.

    mock_datetime.now.return_value.isoformat.return_value = "2024-06-01T12:00:00"

    mock_response = MagicMock()
    mock_response.json.return_value = {"status": "ok"}
    mock_requests.post.return_value = mock_response

    from payment_processor import charge_customer
    result = charge_customer(9.99, "tok_amex_xyz")

    assert result == {"status": "ok"}
```

> **💡 Expert Tip:** When in doubt, print `your_module.__dict__` to see the exact names living in that module’s namespace. Whatever name you see there is exactly what you should patch. This removes all guesswork.

> **⚠️ Danger Zone:** The single most common mocking bug is patching `library.function` instead of `your_module.function`. If your mocks mysteriously don’t seem to be taking effect, this is the *first* thing to check.

-----

## Module 5: Scripting Behavior — `return_value` vs `side_effect`

### 📖 Plain English Definition

Once you have a mock in place, you need to tell it how to *behave*. **`return_value`** is the simplest directive: “always return this exact thing when called.” It’s a static, fixed response. **`side_effect`** is a dynamic power tool: it can make the mock raise an exception, call a real function, or return a different value on each successive call by cycling through a list.

### 🛠️ Ways to Use This

- **Static API response** (`return_value`): Your payment function always returns `{"status": "success"}` — use `return_value` to configure this constant response.
- **Simulating database failure** (`side_effect=Exception`): Use `side_effect=ConnectionError("DB down")` to test your retry or fallback logic.
- **Paginating API responses** (`side_effect=iterable`): Pass a list like `side_effect=[page1_data, page2_data, page3_data]` to simulate a paginated API that returns different data on each call.

### 🎬 Movie Set Analogy

`return_value` is a **pre-filmed cutaway shot**: every time the scene calls for it, the editor drops in the same clip. `side_effect` is a **live improv actor**: each time they’re cued, they can do something completely different — deliver a line, throw a punch, or collapse dramatically — based on dynamic rules.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module5_return_value_vs_side_effect.py
# PURPOSE: Master static vs. dynamic mock behavior
# ============================================================

import pytest
from unittest.mock import Mock, MagicMock, call


# --- SECTION 1: return_value — The Static Response ---
# Use this when the mock should always return the same thing.
# It's the simplest, most readable form of mock configuration.

def test_return_value_static_response():
    mock_api_client = Mock()

    # `return_value` is the value the mock returns when CALLED.
    # mock_api_client() → returns {"user_id": 42, "name": "Alice"}
    mock_api_client.get_user.return_value = {"user_id": 42, "name": "Alice"}

    # Call it multiple times — it always returns the same thing.
    result1 = mock_api_client.get_user(42)
    result2 = mock_api_client.get_user(42)

    assert result1 == {"user_id": 42, "name": "Alice"}
    assert result2 == {"user_id": 42, "name": "Alice"}  # Same result, every time.


# --- SECTION 2: side_effect with an Exception ---
# This is the primary way to test your error-handling paths.
# When the mock is called, it RAISES the exception instead of returning.

class DataFetcher:
    def fetch(self, url: str) -> dict:
        pass  # Real HTTP call in production.

    def fetch_with_retry(self, url: str, retries: int = 3) -> dict:
        """Fetches data, retrying on failure up to `retries` times."""
        for attempt in range(retries):
            try:
                return self.fetch(url)
            except ConnectionError:
                if attempt == retries - 1:
                    raise  # Re-raise on the last attempt.
        return {}


def test_side_effect_raises_exception():
    fetcher = DataFetcher()
    mock_fetch = Mock()

    # Assigning an Exception INSTANCE to side_effect means:
    # "when this mock is called, raise this exception."
    mock_fetch.side_effect = ConnectionError("Timeout after 30s")

    # Replace the real fetch method with our mock on this instance.
    fetcher.fetch = mock_fetch

    # Verify that our code surfaces the error correctly.
    with pytest.raises(ConnectionError, match="Timeout after 30s"):
        fetcher.fetch_with_retry("https://api.example.com/data", retries=1)


# --- SECTION 3: side_effect with a List (Cycling Through Values) ---
# When side_effect is a list, each call consumes the next item.
# If the item is an Exception, it's raised. If it's a value, it's returned.
# This is perfect for testing retry logic or paginated data.

def test_side_effect_list_for_retry_success():
    fetcher = DataFetcher()
    mock_fetch = Mock()

    # First call raises ConnectionError (simulating a transient failure).
    # Second call returns the data (simulating successful retry).
    mock_fetch.side_effect = [
        ConnectionError("First attempt failed"),  # Call 1: raises this
        {"data": "success"}                        # Call 2: returns this
    ]

    fetcher.fetch = mock_fetch
    result = fetcher.fetch_with_retry("https://api.example.com/data", retries=2)

    # Verify our retry logic worked: it failed once, then succeeded.
    assert result == {"data": "success"}
    assert mock_fetch.call_count == 2  # Confirms it was called exactly twice.


# --- SECTION 4: side_effect with a Callable (Dynamic Responses) ---
# You can assign any callable to side_effect.
# The mock will call it with the same arguments it received, using the return value.
# This is powerful for argument-dependent behavior.

def test_side_effect_callable_dynamic_response():
    mock_db = Mock()

    # This function dynamically decides what to return based on the input.
    def dynamic_query_response(sql: str, params: tuple = ()):
        """Simulate different query results based on which table is queried."""
        if "users" in sql:
            return [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]
        elif "orders" in sql:
            return [{"order_id": 101, "total": 99.99}]
        else:
            return []  # No results for unknown tables.

    # Assigning a callable makes the mock call it and return its result.
    mock_db.query.side_effect = dynamic_query_response

    users = mock_db.query("SELECT * FROM users")
    orders = mock_db.query("SELECT * FROM orders")
    unknown = mock_db.query("SELECT * FROM nonexistent_table")

    assert users == [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]
    assert orders == [{"order_id": 101, "total": 99.99}]
    assert unknown == []


# --- SECTION 5: Combining return_value and side_effect ---
# side_effect OVERRIDES return_value when both are set.
# Think of side_effect as a "pre-processor" — it runs first.
# Only if side_effect returns DEFAULT (the sentinel) will return_value be used.

def test_side_effect_overrides_return_value():
    mock = Mock()
    mock.return_value = "this will be ignored"
    mock.side_effect = ValueError("side_effect wins!")

    # side_effect takes priority — return_value is completely overridden.
    with pytest.raises(ValueError, match="side_effect wins!"):
        mock()


# --- SECTION 6: Asserting Call History ---
# Mocks record every call for later inspection.

def test_call_history_inspection():
    mock_notifier = Mock()

    mock_notifier.notify("alice@example.com", "Welcome!")
    mock_notifier.notify("bob@example.com", "Update!")
    mock_notifier.notify("carol@example.com", "Reminder!")

    # assert_called_with() only checks the MOST RECENT call.
    mock_notifier.notify.assert_called_with("carol@example.com", "Reminder!")

    # assert_any_call() checks that this call happened AT SOME POINT.
    mock_notifier.notify.assert_any_call("alice@example.com", "Welcome!")

    # call_args_list gives you the full ordered history.
    expected_calls = [
        call("alice@example.com", "Welcome!"),
        call("bob@example.com", "Update!"),
        call("carol@example.com", "Reminder!"),
    ]
    mock_notifier.notify.assert_has_calls(expected_calls, any_order=False)

    # call_count is the simplest way to verify "was this called N times?"
    assert mock_notifier.notify.call_count == 3
```

> **💡 Expert Tip:** For testing multiple return values in sequence, `side_effect=[val1, val2, val3]` is far cleaner than calling `return_value` multiple times. Once the list is exhausted, the mock raises `StopIteration` — so make sure your list is at least as long as the number of calls your test will make.

> **⚠️ Danger Zone:** `assert_called_with()` only checks the **last** call. If you call a mock 5 times and only care about the 3rd, use `assert_any_call()` or inspect `call_args_list[2]` directly. Using `assert_called_with()` on a multi-call mock is a subtle but dangerous trap.

-----

## Module 6: Spying with `wraps`

### 📖 Plain English Definition

The `wraps` parameter lets you create a mock that **delegates to the real implementation** while still recording all call activity. The actual function executes and returns its real result, but you gain full introspection: you can assert it was called, check its arguments, and verify call counts. It’s the “spy” pattern — observe without interfering.

### 🛠️ Ways to Use This

- **Verifying a real utility function is called correctly**: You want to confirm your sorting function calls Python’s built-in `sorted()` with the right `key=` argument, without replacing `sorted()`’s behavior.
- **Integration-adjacent tests**: You want the real caching layer to run, but also want to confirm cache hits vs. cache misses in your test assertions.
- **Auditing third-party library calls**: Wrap a logging function to verify it’s being invoked at the right points, while still letting the real log entries be written.

### 🎬 Movie Set Analogy

`wraps` is your **on-set documentary camera**. The Lead Actor (real function) performs the scene for real — no stunt double. But you’ve got a camera crew shadowing them, capturing every move, every word, every moment. The performance is authentic; the documentation is complete.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module6_wraps_spy.py
# PURPOSE: Use wraps to spy on real functions without replacing them
# ============================================================

from unittest.mock import Mock, MagicMock, patch, call
import pytest


# --- The Real Functions We Want to Spy On ---

def calculate_discount(price: float, discount_percent: float) -> float:
    """A real utility function with real business logic."""
    if discount_percent < 0 or discount_percent > 100:
        raise ValueError("Discount must be between 0 and 100.")
    return round(price * (1 - discount_percent / 100), 2)


def apply_bulk_discounts(prices: list, discount_percent: float) -> list:
    """Applies a discount to a list of prices, using calculate_discount internally."""
    # This calls our real calculate_discount function internally.
    return [calculate_discount(p, discount_percent) for p in prices]


# --- SECTION 1: Basic wraps Usage ---

def test_spy_on_calculate_discount():
    # `wraps=calculate_discount` means:
    # 1. When our mock is called, it calls the REAL calculate_discount.
    # 2. The REAL result is returned.
    # 3. The call is RECORDED on the mock for inspection.
    spy = Mock(wraps=calculate_discount)

    # The real function runs — no stubbing, no fake return value.
    result = spy(100.0, 20.0)

    # Verify the REAL business logic produced the correct output.
    assert result == 80.0  # 100 - 20% = 80

    # Verify the spy captured the call correctly.
    spy.assert_called_once_with(100.0, 20.0)


# --- SECTION 2: Spying on Real Exceptions ---
# The real function's real exceptions still propagate through the spy.

def test_spy_propagates_real_exceptions():
    spy = Mock(wraps=calculate_discount)

    # The real calculate_discount raises ValueError for invalid discounts.
    # The spy lets this exception pass through — it doesn't swallow it.
    with pytest.raises(ValueError, match="Discount must be between 0 and 100"):
        spy(100.0, -10.0)  # Invalid: negative discount.

    # Even though it raised, the call was still recorded.
    spy.assert_called_once_with(100.0, -10.0)


# --- SECTION 3: Spying via patch() to Monitor Internal Calls ---
# The most powerful use: inject a spy to monitor what happens INSIDE a function.

def test_spy_on_internal_calculate_discount_calls():
    # Patch `calculate_discount` in THIS module's namespace with a spy.
    # The spy wraps the real function, so results are unchanged.
    with patch(
        "__main__.calculate_discount",  # Target the name in this module.
        wraps=calculate_discount          # But wrap the real function so it still runs.
    ) as spy_discount:
        prices = [100.0, 200.0, 50.0]
        results = apply_bulk_discounts(prices, 10.0)

        # Verify the REAL calculation was applied to each price.
        assert results == [90.0, 180.0, 45.0]

        # Now inspect HOW apply_bulk_discounts used calculate_discount internally.
        # This gives us visibility into the function's internals without altering them.
        assert spy_discount.call_count == 3  # Called once per price.

        # Verify the exact arguments for each internal call.
        spy_discount.assert_has_calls([
            call(100.0, 10.0),
            call(200.0, 10.0),
            call(50.0, 10.0),
        ])


# --- SECTION 4: wraps vs. return_value — The Key Difference ---
# When `wraps` is set, `return_value` is IGNORED.
# The real function's return value is always used.

def test_wraps_ignores_return_value():
    spy = Mock(wraps=calculate_discount)
    spy.return_value = 9999  # This is set but will be completely ignored.

    result = spy(100.0, 50.0)

    # The real calculate_discount returns 50.0 (100 - 50%).
    # Our fake `return_value = 9999` was ignored because `wraps` takes precedence.
    assert result == 50.0
    assert result != 9999


# --- SECTION 5: Partial Spy — Wrapping Only Some Methods ---
# You can wrap a whole object but override specific methods.

class PricingEngine:
    def base_price(self, product_id: int) -> float:
        return 100.0  # In reality, a DB lookup.

    def apply_tax(self, price: float, rate: float) -> float:
        return round(price * (1 + rate), 2)  # Real tax calculation.


def test_partial_spy_on_class():
    real_engine = PricingEngine()

    # Wrap the entire real instance — all methods delegate to the real ones.
    spy_engine = MagicMock(wraps=real_engine)

    # Override ONLY base_price — stub its return without affecting apply_tax.
    spy_engine.base_price.return_value = 200.0  # Override the DB lookup.
    # Note: setting return_value on a wrapped mock's method OVERRIDES the wrap for THAT method.

    base = spy_engine.base_price(product_id=42)
    final_price = spy_engine.apply_tax(base, rate=0.1)

    # base_price was stubbed, so it returns 200.0.
    assert base == 200.0
    # apply_tax still ran the REAL logic: 200.0 * 1.1 = 220.0
    assert final_price == 220.0

    # Both calls were recorded.
    spy_engine.base_price.assert_called_once_with(product_id=42)
    spy_engine.apply_tax.assert_called_once_with(200.0, rate=0.1)
```

> **💡 Expert Tip:** `wraps` is ideal for “characterization tests” — tests that document and lock down what existing (potentially poorly-understood) code actually does, before you refactor it. You observe the real behavior, capture it in assertions, then safely change the internals.

> **⚠️ Danger Zone:** Don’t use `wraps` when the real function has network calls, file I/O, or other true side effects. In that case, you *want* a pure mock that doesn’t execute the real code. `wraps` is for lightweight, pure-function spying — not as a backdoor to let dangerous side effects through.

-----

## Module 7: Monkeypatching — The pytest Surgical Tool

### 📖 Plain English Definition

`monkeypatch` is a pytest fixture that provides a **safe, scoped mechanism to directly modify attributes, environment variables, and dictionary entries** during a single test. Unlike `patch()` which works by replacing names with mock objects, `monkeypatch` works by directly setting or removing values on real objects. Crucially, every change it makes is **automatically reversed** after the test completes — no manual cleanup required.

### 🎬 Movie Set Analogy: The Quick Script Change

You’re mid-scene and the director shouts “Quick — change that line!” The script supervisor (monkeypatch) steps in, makes a surgical edit to the page the actor is holding right now, and the scene continues. After the take, the script supervisor quietly restores the original page. The actor (the real function) was performing all along; only the specific line they were reading changed.

### 🛠️ Ways to Use This

- **`setattr` — Modifying Functions on Objects**: Replace `os.path.exists` with a lambda that always returns `True`, testing code that branches on file existence without touching the filesystem.
- **`setenv` — Modifying Environment Variables**: Inject `DATABASE_URL=sqlite:///:memory:` or `API_KEY=test-key-123` without modifying the actual OS environment — perfect for testing configuration-dependent code.
- **`delattr` — Testing Missing Attributes**: Remove an attribute that your code uses as a feature flag, testing the degraded fallback path.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module7_monkeypatch.py
# PURPOSE: Use pytest's monkeypatch fixture for surgical test modifications
# ============================================================

import os
import pytest


# --- Production Code Under Test ---

def load_database_url() -> str:
    """Reads database connection info from environment variables."""
    url = os.environ.get("DATABASE_URL")
    if not url:
        raise EnvironmentError("DATABASE_URL environment variable is not set.")
    return url


def check_file_exists(path: str) -> str:
    """Returns a message based on whether a file exists."""
    if os.path.exists(path):
        return f"File found: {path}"
    else:
        return f"File missing: {path}"


class FeatureFlags:
    """Manages feature flags that may or may not be defined."""
    NEW_DASHBOARD = True  # Feature flag as a class attribute.


def is_new_dashboard_enabled() -> bool:
    """Checks if the new dashboard feature is active."""
    return getattr(FeatureFlags, "NEW_DASHBOARD", False)


# ============================================================
# SECTION 1: setenv — Injecting Environment Variables
# Use when your code reads from os.environ.
# ============================================================

def test_load_database_url_with_setenv(monkeypatch):
    # `monkeypatch.setenv()` sets an environment variable FOR THIS TEST ONLY.
    # After the test, the original value (or absence) is restored automatically.
    monkeypatch.setenv("DATABASE_URL", "postgresql://localhost/testdb")

    # Now our function will find the env var we just injected.
    result = load_database_url()

    assert result == "postgresql://localhost/testdb"


def test_load_database_url_missing_raises(monkeypatch):
    # `monkeypatch.delenv()` removes an environment variable for this test.
    # `raising=False` prevents an error if the variable didn't exist to begin with.
    monkeypatch.delenv("DATABASE_URL", raising=False)

    # With the env var gone, our function should raise EnvironmentError.
    with pytest.raises(EnvironmentError, match="DATABASE_URL environment variable is not set"):
        load_database_url()


# ============================================================
# SECTION 2: setattr — Replacing Functions and Attributes
# Use when you want to replace a method or attribute on any object.
# ============================================================

def test_check_file_exists_when_file_is_present(monkeypatch):
    # Replace `os.path.exists` with a lambda that always returns True.
    # This lets us test the "file found" branch without creating real files.
    monkeypatch.setattr(os.path, "exists", lambda path: True)
    # After this test, os.path.exists is restored to its real implementation.

    result = check_file_exists("/some/made/up/path.txt")

    assert result == "File found: /some/made/up/path.txt"


def test_check_file_exists_when_file_is_absent(monkeypatch):
    # Now simulate the "file not found" case.
    monkeypatch.setattr(os.path, "exists", lambda path: False)

    result = check_file_exists("/nonexistent/path.log")

    assert result == "File missing: /nonexistent/path.log"


# ============================================================
# SECTION 3: setattr with a module-level function
# ============================================================

# A module-level function we want to replace in a test.
def send_email(to: str, subject: str, body: str):
    """Real function — would send an actual email."""
    pass  # Production: uses smtplib or an email API.


def notify_user(user_email: str, event_name: str):
    """Production function that calls send_email internally."""
    send_email(
        to=user_email,
        subject=f"Event: {event_name}",
        body=f"Hello! Your event '{event_name}' was triggered."
    )


def test_notify_user_calls_send_email(monkeypatch):
    # Track whether our fake send_email was called and with what args.
    captured_calls = []

    def fake_send_email(to, subject, body):
        # This fake records the call instead of sending a real email.
        captured_calls.append({"to": to, "subject": subject, "body": body})

    # Replace the module-level `send_email` in this test module.
    # Format: monkeypatch.setattr(module, "function_name", replacement)
    import test_module7_monkeypatch as this_module  # Import self to reference.
    monkeypatch.setattr(this_module, "send_email", fake_send_email)

    notify_user("alice@example.com", "UserSignup")

    # Verify the notification was routed correctly.
    assert len(captured_calls) == 1
    assert captured_calls[0]["to"] == "alice@example.com"
    assert captured_calls[0]["subject"] == "Event: UserSignup"


# ============================================================
# SECTION 4: delattr — Testing Missing Attribute Paths
# Use to test degraded/fallback behavior when a feature is disabled.
# ============================================================

def test_new_dashboard_feature_flag_missing(monkeypatch):
    # Remove `NEW_DASHBOARD` from FeatureFlags to simulate the flag being undefined.
    # `raising=True` (default) would raise AttributeError if the attr doesn't exist.
    monkeypatch.delattr(FeatureFlags, "NEW_DASHBOARD")

    # Our function uses `getattr(..., False)` as a default — it should return False.
    result = is_new_dashboard_enabled()

    assert result is False  # Confirms the fallback default works correctly.


def test_new_dashboard_feature_flag_present(monkeypatch):
    # Ensure the flag is explicitly set to True for this test.
    monkeypatch.setattr(FeatureFlags, "NEW_DASHBOARD", True)

    result = is_new_dashboard_enabled()

    assert result is True


# ============================================================
# SECTION 5: setitem / delitem — Modifying Dictionaries
# ============================================================

APP_CONFIG = {
    "debug_mode": False,
    "max_connections": 10,
    "feature_flags": {"new_ui": False}
}


def is_debug_enabled() -> bool:
    """Reads debug mode from the global config dictionary."""
    return APP_CONFIG.get("debug_mode", False)


def test_debug_mode_enabled_via_setitem(monkeypatch):
    # `monkeypatch.setitem()` safely modifies a dictionary key for this test only.
    # The original value is restored after the test automatically.
    monkeypatch.setitem(APP_CONFIG, "debug_mode", True)

    result = is_debug_enabled()

    assert result is True


def test_debug_mode_disabled_when_key_deleted(monkeypatch):
    # `monkeypatch.delitem()` removes a key from the dictionary for this test.
    monkeypatch.delitem(APP_CONFIG, "debug_mode")

    # Without the key, `APP_CONFIG.get("debug_mode", False)` returns the default.
    result = is_debug_enabled()

    assert result is False
```

> **💡 Expert Tip:** `monkeypatch` is ideal for modifying *configuration* (env vars, config dicts, feature flags) while `patch()` is better for replacing *dependencies* (external services, databases, file I/O). When you’re changing what an object *is*, use `monkeypatch`. When you’re replacing what your code *calls*, use `patch()`.

> **⚠️ Danger Zone:** `monkeypatch` is a *fixture* — it only works in pytest functions that declare it as a parameter. It won’t work in `unittest.TestCase` methods. If your test class inherits from `unittest.TestCase`, use `patch()` instead.

-----

## Module 8: Comparison Tables

### Table 1: `unittest.mock.patch` vs. `pytest.monkeypatch`

|Dimension                       |`unittest.mock.patch`                             |`pytest.monkeypatch`                              |
|--------------------------------|--------------------------------------------------|--------------------------------------------------|
|**Primary Use**                 |Replacing objects/functions with `Mock` instances |Directly modifying attributes, env vars, dicts    |
|**Interface**                   |Decorator, context manager, or manual start/stop  |pytest fixture injected as a test parameter       |
|**Scope**                       |Duration of the decorated function or `with` block|Duration of a single test (auto-reverted)         |
|**Environment Variables**       |Use `patch.dict(os.environ, {...})`               |`monkeypatch.setenv()` / `delenv()` — cleaner API |
|**Dictionary Modification**     |`patch.dict(my_dict, {"key": "val"})`             |`monkeypatch.setitem()` / `delitem()`             |
|**Attribute Replacement**       |`patch.object(obj, "attr", Mock())`               |`monkeypatch.setattr(obj, "attr", value)`         |
|**Works with unittest.TestCase**|✅ Yes — designed for it                           |❌ No — pytest fixtures only                       |
|**Auto-cleanup**                |✅ Yes (context manager / decorator handles it)    |✅ Yes (fixture teardown handles it)               |
|**Replace with Mock Object**    |✅ Primary purpose                                 |⚠️ Possible but not idiomatic                      |
|**Async Support**               |✅ Via `AsyncMock`                                 |✅ Can patch async functions via `setattr`         |
|**Best For**                    |Mocking service dependencies, external I/O        |Modifying config, env, feature flags, global state|
|**Readability**                 |Medium — string-based target paths can be opaque  |High — fixture injection is explicit              |
|**Typical Syntax**              |`@patch("module.ClassName.method")`               |`monkeypatch.setattr(module, "attr", value)`      |

-----

### Table 2: `Mock` vs. `MagicMock` vs. `AsyncMock`

|Dimension                        |`Mock`                                                                           |`MagicMock`                                                                        |`AsyncMock`                                            |
|---------------------------------|---------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|-------------------------------------------------------|
|**Best For**                     |Simple objects with no protocol magic                                            |Containers, context managers, protocol-heavy objects                               |Async functions and methods using `async/await`        |
|**Dunder Methods Pre-configured**|❌ None                                                                           |✅ Yes — `__len__`, `__iter__`, `__getitem__`, `__enter__`, `__exit__`, and 20+ more|✅ Yes — plus `__aenter__`, `__aexit__`, `__aiter__`    |
|**Returns Coroutine When Called**|❌ No — raises `TypeError` when awaited                                           |❌ No — raises `TypeError` when awaited                                             |✅ Yes — designed for this                              |
|**`for x in mock:` works**       |❌ Raises `TypeError`                                                             |✅ Works (via `__iter__`)                                                           |✅ Works                                                |
|**`len(mock)` works**            |❌ Raises `TypeError`                                                             |✅ Works (via `__len__`)                                                            |✅ Works                                                |
|**`with mock as x:` works**      |❌ Raises `TypeError`                                                             |✅ Works (via `__enter__`/`__exit__`)                                               |✅ Works                                                |
|**`async with mock:` works**     |❌ No                                                                             |❌ No                                                                               |✅ Works (via `__aenter__`/`__aexit__`)                 |
|**`await mock()` works**         |❌ Raises `TypeError`                                                             |❌ Raises `TypeError`                                                               |✅ Works                                                |
|**Call Assertions**              |`assert_called_with()`, `assert_called_once()`                                   |Same as Mock                                                                       |Same + `assert_awaited_with()`, `assert_awaited_once()`|
|**Default Child Mocks**          |Returns `Mock` on attribute access                                               |Returns `MagicMock` on attribute access                                            |Returns `AsyncMock` for async methods                  |
|**`spec=` Compatible**           |✅ Yes                                                                            |✅ Yes                                                                              |✅ Yes                                                  |
|**`autospec=` Compatible**       |✅ Yes                                                                            |✅ Yes                                                                              |✅ Yes — auto-detects async methods                     |
|**Performance**                  |Slightly lighter weight                                                          |Slightly heavier (more pre-configured)                                             |Requires event loop context                            |
|**When to Prefer**               |Simple services, pure function mocks, when you want to be explicit about no-magic|Default choice for most cases                                                      |Any time you see `async def` in the code under test    |

-----

## Quick Reference Cheat Sheet

```python
# ── CREATING MOCKS ──────────────────────────────────────────────
from unittest.mock import Mock, MagicMock, AsyncMock, create_autospec

m = Mock()                          # Blank mock, no magic methods
m = MagicMock()                     # Mock + all dunder methods
m = AsyncMock()                     # Mock for async/await code
m = create_autospec(MyClass, instance=True)  # Spec-validated mock

# ── CONFIGURING BEHAVIOR ────────────────────────────────────────
m.method.return_value = 42          # Always return 42
m.method.side_effect = ValueError() # Raise this exception
m.method.side_effect = [1, 2, 3]   # Return 1, then 2, then 3
m.method.side_effect = my_func      # Call my_func with the same args

# ── ASSERTIONS ──────────────────────────────────────────────────
m.method.assert_called()                         # Was called at least once
m.method.assert_called_once()                    # Was called exactly once
m.method.assert_called_with(arg1, kwarg=val)     # Last call had these args
m.method.assert_called_once_with(arg1, kwarg=val)# Only call had these args
m.method.assert_any_call(arg1)                   # Some call had this arg
m.method.assert_not_called()                     # Was never called
m.method.call_count                              # Number of calls (int)
m.method.call_args_list                          # List of all call objects

# ASYNC-SPECIFIC (AsyncMock)
await_mock.assert_awaited()
await_mock.assert_awaited_once_with(arg1)

# ── PATCHING ────────────────────────────────────────────────────
from unittest.mock import patch

# As context manager
with patch("mymodule.ClassName") as mock_cls:
    ...

# As decorator
@patch("mymodule.ClassName")
def test_func(mock_cls):
    ...

# Patch an object's attribute
with patch.object(MyClass, "method_name") as mock_method:
    ...

# Patch a dictionary
with patch.dict(os.environ, {"API_KEY": "test-123"}):
    ...

# With autospec
with patch("mymodule.MyClass", autospec=True) as mock_cls:
    ...

# ── MONKEYPATCH (pytest fixture) ────────────────────────────────
def test_example(monkeypatch):
    monkeypatch.setattr(obj, "attr", new_value)       # Replace attribute
    monkeypatch.setattr("module.func", fake_func)     # Replace by string path
    monkeypatch.delattr(obj, "attr")                  # Remove attribute
    monkeypatch.setenv("VAR_NAME", "value")           # Set env var
    monkeypatch.delenv("VAR_NAME", raising=False)     # Remove env var
    monkeypatch.setitem(my_dict, "key", "value")      # Set dict item
    monkeypatch.delitem(my_dict, "key")               # Remove dict item

# ── THE GOLDEN RULES ────────────────────────────────────────────
# 1. Patch WHERE the name is USED, not where it's defined.
#    ✅ patch("mymodule.requests")   ❌ patch("requests.get")
#
# 2. Use MagicMock by default; Mock only when you want no magic.
#
# 3. Use AsyncMock for anything with `async def` or `await`.
#
# 4. Use create_autospec / autospec=True to catch interface drift.
#
# 5. Use wraps= to spy without replacing real behavior.
#
# 6. Use monkeypatch for env vars, config, and feature flags.
#    Use patch() for service dependencies and external I/O.
```

-----

*Built for mastery. Patch confidently. Test fearlessly.*
