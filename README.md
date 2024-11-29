Testing Python code is important to help ensure your code runs as expected and is maintainable over time. 
- [Best Practiced](#best-practiced)
  - [1. **Unit Testing**](#1-unit-testing)
  - [2. **Use Test-Driven Development (TDD)**](#2-use-test-driven-development-tdd)
  - [3. **Use Code Coverage Tools**](#3-use-code-coverage-tools)
  - [4. **Organize Tests Well**](#4-organize-tests-well)
  - [5. **Use Fixtures and Setup Functions**](#5-use-fixtures-and-setup-functions)
  - [6. **Run Tests Automatically with CI/CD**](#6-run-tests-automatically-with-cicd)
  - [7. **Leverage Static Code Analysis and Linters**](#7-leverage-static-code-analysis-and-linters)
  - [8. **Conduct Integration and End-to-End Testing**](#8-conduct-integration-and-end-to-end-testing)
  - [9. **Review and Refine Tests Regularly**](#9-review-and-refine-tests-regularly)
- [Edge Cases Testing](#edge-cases-testing)
  - [What are edge cases?](#what-are-edge-cases)
  - [1. **Identify Edge Cases Systematically**](#1-identify-edge-cases-systematically)
    - [1.1 **Handling Boundary Values**](#11-handling-boundary-values)
    - [1.2. **Testing Empty and Large Collections**](#12-testing-empty-and-large-collections)
    - [1.3. **Testing String Edge Cases**](#13-testing-string-edge-cases)
  - [2. **Use Parametrized Testing**](#2-use-parametrized-testing)
  - [3. **Generate Randomized and Fuzzed Data**](#3-generate-randomized-and-fuzzed-data)
  - [4. **Mock System Conditions and Dependencies**](#4-mock-system-conditions-and-dependencies)
  - [5. **Test for Exception Handling**](#5-test-for-exception-handling)
  - [6. **Use Assertions for Accuracy and Robustness**](#6-use-assertions-for-accuracy-and-robustness)
  - [7. **Monitor Code Paths with Coverage Analysis**](#7-monitor-code-paths-with-coverage-analysis)
  - [8. **Stress Test for Performance Edge Cases**](#8-stress-test-for-performance-edge-cases)
  - [7. **Testing Resource Constraints**](#7-testing-resource-constraints)

## Best Practiced ##

### 1. **Unit Testing**
   - **What is a Unit Test**: Units tests focus on small parts of the code (an "isolated" piece of code if you will, which does one thing and one thing only), typically individual functions or methods. Use the **`unittest`** module in the Python standard library or popular alternatives like **`pytest`**.

   In Unit Tests you can also:
   - **Mock Dependencies**: Use libraries like **`unittest.mock`** or **`pytest-mock`** to replace parts of the system that your code interacts with (e.g., APIs, databases) so you can isolate the function you're testing.
   - **Assert Statements**: Write clear assertions that check for expected outcomes as demonstrated below with assertion functions.

   Here is a fun example of unit test. We want to write a function that calculates the area of a circle. We can write it like this:
   ```python
   from math import pi

   def circle_area(r):
    return pi * (r ** 2)
   ```
   This may seems fine and dandy until we choose violence and run the funtion with the following values:
   ```python
   radii = [2, 0, -3, 2 + 5j, True, "radius"]
   message = "Area of circles with r = {radius} is {area}."

   for p in radii:
       A = circle_area(r)
       print (message. format (radius=r, area=A))
   ```
   At this point, we get the following output:
   ```
   Area of circles with r = 2 is 12.566370614359172.
   Area of circles with r = 0 is 0.0.
   Area of circles with r = -3 is 28.274333882308138.                           # negative radius?!
   Area of circles with r = (2+5j) is (-65.97344572538566+62.83185307179586j).  # tuple?!
   Area of circles with r = True is 3.141592653589793.                          # bool?!
   Traceback (most recent call last):                                           # get this error because you used a string
   File "circles.py", line 11, in < module >
     A = circle_area(r) 
   File "circles.py", line 4, in circle_area
     return pi*(p**2)
   TypeError: unsupported operand type(s) for ** or pow(): 'str' and 'int'
   ```

To avoid this, we can write unit tests to test the function. First, we create a separate file with the same name as the file which contains the function to test followed by _test.py. For instance, let's say that we have called out first file `circles.py` In this new file `circles_test.py`, we write this:

```python
import unittest
from circles import circle_area
from math import pi

class TestCircleArea (unittest.TestCase) :
    def test_area (self) :
    """ 
    Test areas when radius >= 0 for several circles using `assertAlmostEqual`. The first value is the output of the circle_area function, and the second is the correct answer. The test compares the two up to the 7th decimal place.
    """
        self.assertAlmostEqual (circle_area(1), pi)  
        self. assertAlmostEqual (circle_area(0), 0)
        self.assertAlmostEqual(circle_area(2.1), pi * 2.1**2)
        # If any of these comparisons fail, than the function fails the test.
```
To actually run this, we need to open the terminal and type `python -m unittest test_circles` where -m means that we are telling python to run the unittest module as a script.

Now let's see how we can handle invalid input. We write a new test in `circles_test.py`:
```python
def test_values (self) :
"""
Make sure value errors are raised when necessary. The first agrument is the exception class that should be raised, the second is the  function, the remaining are arguments to the function.
"""
    self.assertRaises (ValueError, circle_area, -2)
```
In this case, if we try to compute the area of a circle of radius -2, the function should raise a `ValueError` and in fact once we run the unit test, we get a screenfull of failure messages (really python, you made your point).
```
python -m unittest
.F
ニニニニニニニニニニニニニニニニニニニニニニニニニニニ
FAIL: test_values (test_circles. TestCircleArea)
ーーーーーーーーーーーーーーーーーーーーーーーーーーーーーー
Traceback (most recent call last): 
  File "test_circles.py", line 14, in test_values
    self.assertRaises (ValueError, circle_area, -2)
AssertionError: ValueError not raised by circle_area
ーーーーーーーーーーーーーーーーーーーーーーーーーーーーーー
Ran 2 tests in 0.000s

FAILED (failures=1)
```
BUT a `ValueError` was not raised when the input was negative so let's go back to `circles.py`. If a negative number is used, we need to raise a `ValueError` so that before computing the area we check to see if it's negative. If so, the error and error message are raised.
```python
from math import pi

def circle_area (r) :
    if r < 0:
        raise ValueError ("The radius cannot be negative. ")
    return pi*(n**2)
```
Now, if you run the unit test the error should be correctly raised.

To complete the testing and make sure a `TypeError` is raised when the input is not a real number, we go back to our resting script:
```python
import unittest
from circles import circle_area
from math import pi

class TestCircleArea (unittest. TestCase) :
    def test_area (self) :
    """ 
    Test areas when radius >= 0 for several circles. The first value is the out of the circle_area function, and the second is the correct answer. The test compares the two up to the 7th decimal place.
    """
        self.assertAlmostEqual (circle_area(1), pi)  
        self. assertAlmostEqual (circle_area(0), 0)
        self.assertAlmostEqual(circle_area(2.1), pi * 2.1**2)
        # If any of these comparisons fail, than the function fails the test.

    def test_values (self) :
    """
    Make sure value errors are raised when necessary. The first agrument is the exception class that should be raised, the second is the  function, the remaining are arguments to the function.
    """
        self.assertRaises (ValueError, circle_area, -2)  # if the radius is a negative number
    
    def test_types (self) :
    """
    Make sure type errors are raised when necessary
    """
        self.assertRaises (TypeError, circle_area, 3+5j)
        self.assertRaises(TypeError, circle_area, True)
        self assertRaises(TypeError, circle_area, "radius")  # if the radius is anything but an int, float
```
If we run the unittest again, we are alerted to out failures and we can fix them by returning to out circle_area function.
```python
from math import pi

def circle_area(r):
    if type(r) not in [int, float]:
        raise TypeError ("The radius must be a non-negative real number.")
    if r < 0:
        raise ValueError ("The radius cannot be negative.")
    return pi*(p**2)
```
If we run the test now, where should be no errors.

As an aside, in this example we have used the assertAlmostEqual and assertRaises methods, but python has many more assert methods for unit testing. 

### 2. **Use Test-Driven Development (TDD)**
Essentially, TDD means writing tests for your code before you write your code. The tests are __failing tests__ and you slowly write code to pass the failing tests.
   - **Write Tests First**: Start by writing tests for the behavior you want your code to have, then write the code to pass those tests. This approach helps ensure that your code meets the requirements and is less prone to bugs. The simplest approach is to write an automated unit-level test case that fails, then writing just enough code to make the test pass, then refactoring both the test code and the production code, then repeating with another new test case.
   - **Refactor and Retest**: After making your code pass, refactor if necessary, running tests each time to ensure no functionality is broken. This tends to be where stuff breaks so be careful.
   ![Alt text](https://upload.wikimedia.org/wikipedia/commons/0/0b/TDD_Global_Lifecycle.png)
   The TDD steps are generally as follows. These are based on the book Test-Driven Development by Example,and Kent Beck's Canon TDD article.

     1. List scenarios for the new feature
   List the expected variants in the new behavior. “There’s the basic case & then what-if this service times out & what-if the key isn’t in the database yet &…” The developer can discover these specifications by asking about use cases and user stories. A key benefit of TDD is that it makes the developer focus on requirements before writing code. This is in contrast with the usual practice, where unit tests are only written after code.
     2. Write a test for an item on the list
   Write an automated test that would pass if the variant in the new behavior is met.
     3. Run all tests. The new test should fail – for expected reasons
   This shows that new code is actually needed for the desired feature. It validates that the test harness is working correctly. It rules out the possibility that the new test is flawed and will always pass.
     4. Write the simplest code that passes the new test
   Inelegant code and hard coding is acceptable. The code will be honed in Step 6. No code should be added beyond the tested functionality.
     5. All tests should now pass
   If any fail, fix failing tests with minimal changes until all pass.
     6. Refactor as needed while ensuring all tests continue to pass
   Code is refactored for readability and maintainability. In particular, hard-coded test data should be removed from the production code. Running the test suite after each refactor ensures that no existing functionality is broken. Examples of refactoring:

        * moving code to where it most logically belongs
        * removing duplicate code
        * making names self-documenting
        * splitting methods into smaller pieces
        * re-arranging inheritance hierarchies

Let's make a short example of TDD. Let's say we want to add numbers together. However, __before__ we write the function, we will write a test to check if 1 + 1 equals 2.

```python
assert add(1, 1) == 2
```
The `assert` keyword here is used to write test cases and to confirm that the program conditions are `True`. In other words, it allows us to validate the behavior and outputs of our code. When an assert statement fails, it raises an `AssertionError`, signaling an issue with the code. Such error would look like this:

This will result in a NameError, indicating that no implementation exists, known as the "red" step. You get a NameError when you try to call a function or variable that has not been defined.

```
NameError             Traceback (most recent call last)
Cell In[3], line 1
    ---> 1 assert add(1, 1) == 2
NameError: name 'add' is not defined
```
Now, it's time to implement the function. We can do this by defining the function:

```python 
def add(no1, no2):
    return
```
Our function does not return anything yet. If we try running it, we now get a different error, and `AssertionError`:
```
AssertionError             Traceback (most recent call last)
Cell In[6], line 4
1 def (numi, num2) :
2     return
    ---> 4 assert add (1, 1) == 2
AssertionError:
```
This is because we defined a function that doesn't return anything, duh, so we get `AssertionError` because 1 + 1 does not equal 2 with our empty implementation. Finally, we can make it pass by correctly implementing the logic in the function as follows:
```python
def add (no1, no2):
    return no1 + no2

assert add(1, 1) == 2
```
This code finally runs and the test passes. Well done.

<!-- ## ↓This is still a WIP↓ ##-->
### 3. **Use Code Coverage Tools**
   - Code coverage tools (like **`coverage.py`** or **`pytest-cov`**) show you which parts of your code were exercised by your tests. Aim for high coverage, but be careful not to focus solely on coverage numbers. A 100% coverage doesn’t guarantee that the code works correctly, but it helps identify untested code paths. 

### 4. **Organize Tests Well**
   - **Separate Tests from Production Code**: Place your tests in a dedicated folder (often named `tests`), keeping the test files organized and easy to find.
   - **Use Naming Conventions**: Name your test files and test functions descriptively to make it clear what’s being tested.

### 5. **Use Fixtures and Setup Functions**
   - **Reusable Test Setup**: If you need to set up specific objects or environments for multiple tests, use setup functions or fixtures (especially in **`pytest`**). Fixtures help streamline tests and avoid repetitive code.

### 6. **Run Tests Automatically with CI/CD**
   - **Continuous Integration (CI)**: Tools like **GitHub Actions**, **Jenkins**, **GitLab CI**, or **Travis CI** can automate running your tests whenever you push code to a repository. This helps catch errors early and ensures your codebase remains stable.

### 7. **Leverage Static Code Analysis and Linters**
   - **Static Analysis**: Tools like **Pylint**, **flake8**, and **mypy** (for type checking) can help you find potential issues without running the code. Linters can enforce coding standards and catch common bugs.

### 8. **Conduct Integration and End-to-End Testing**
   - **Integration Tests**: Test how components of your code interact, such as testing a function that relies on database calls.
   - **End-to-End (E2E) Tests**: Simulate the complete workflow in a production-like environment. Tools like **Selenium** or **Playwright** are helpful if you’re testing applications with a web front end.

### 9. **Review and Refine Tests Regularly**
   - **Regularly Update Tests**: As code evolves, review and update tests to ensure they remain relevant. Refactor tests when necessary to maintain clarity and coverage.


## Edge Cases Testing ##

### What are edge cases? ### 
Edge cases refer to unusual or extreme scenarios that occur at the boundaries of the expected input or behavior. These cases are important to consider during programming and testing to ensure robust and reliable code.

Testing rare edge cases is crucial to making code resilient, especially when handling unusual inputs, edge conditions, or failure scenarios. Here’s how to effectively identify and test for edge cases:

### 1. **Identify Edge Cases Systematically**
   - **Input Ranges**: For numeric inputs, test values at the extremes of possible ranges, including very large or very small numbers, zero, and negative numbers if applicable.
   - **Boundary Testing**: Test at the boundaries of conditions that might affect behavior. For example, if your function behaves differently based on the length of a list, test with empty lists, single-element lists, and lists just above and below key thresholds.
   - **Special Characters and Data Types**: If handling strings, test for empty strings, long strings, strings with special characters, and unexpected encodings. For booleans, test both `True` and `False` conditions. For objects, test `None` as well.
   - **Concurrency/Timing**: For code with timing dependencies, edge cases may include rapid consecutive calls, delayed responses, or calls made out of the expected order.

Or course, there can be many types of Edge cases. Here are some examples and how to deal with them:
#### 1.1 **Handling Boundary Values**

Suppose we have a function that calculates the square root of a number. We should handle edge cases like negative numbers, zero, and very large numbers.

```python
import math

def safe_sqrt(x):
    if x < 0:
        raise ValueError("Cannot compute square root of a negative number.")
    return math.sqrt(x)

# Testing the edge cases
try:
    print(safe_sqrt(0))           # Edge case: zero
    print(safe_sqrt(1e10))        # Large number
    print(safe_sqrt(-10))         # Negative number, should raise an error
except ValueError as e:
    print(e)
```

#### 1.2. **Testing Empty and Large Collections**

Consider a function that computes the average of a list. We'll handle edge cases like an empty list, a single-item list, and a very large list.

```python
def average(numbers):
    if not numbers:
        raise ValueError("Cannot compute the average of an empty list.")
    return sum(numbers) / len(numbers)

# Edge case tests
print(average([]))                # Empty list
print(average([5]))               # Single-element list
print(average([1] * 1000000))     # Large list (one million elements)
```

#### 1.3. **Testing String Edge Cases**

For a function that processes strings, test with edge cases such as empty strings, very long strings, and special characters.

```python
def normalize_text(text):
    if not text:
        return ""
    return text.strip().lower()

# Edge case tests
print(normalize_text(""))                   # Empty string
print(normalize_text("   Text with spaces   "))  # Stripping spaces
print(normalize_text("1234567890" * 1000))  # Very long string
print(normalize_text("Hello!@#$%^&*()"))    # Special characters
```


### 2. **Use Parametrized Testing**
The idea is to write a single test method or function that can be executed multiple times with different input values rather than writing separate test cases for each specific input scenario. 
   - With **parametrized testing**, you can run the same test logic with a variety of edge case values. This is particularly easy with **pytest’s `@pytest.mark.parametrize`** decorator, which allows you to specify different inputs and expected outcomes in one test.

Here is a small example of parametrized testing with `@pytest.mark.parametrize`:

```python
import pytest

def is_even(number):
    return number % 2 == 0

@pytest.mark.parametrize("number, expected", [
    (0, True),            # Edge case: zero
    (-2, True),           # Negative even number
    (1, False),           # Odd number
    (10**10, True),       # Large number
    (10**10 + 1, False),  # Large odd number
])
def test_is_even(number, expected):
    assert is_even(number) == expected
```

Run this with pytest to test all cases automatically.

### 3. **Generate Randomized and Fuzzed Data**
   - **Fuzz Testing**: Generate random or unexpected input data to see if the code can handle it gracefully. Libraries like `Hypothesis` can automatically generate edge-case data based on test function parameters.
   - **Random Values**: Use random or pseudorandom values for testing, especially when testing functions with a wide range of valid inputs. Just be sure to fix the random seed if you need reproducible results.

Using **Hypothesis** to generate a wide variety of input values, including edge cases automatically.

```python
from hypothesis import given, strategies as st

@given(st.integers())
def test_is_even_property(number):
    assert is_even(number) == (number % 2 == 0)

# This will test is_even with a wide range of integers, including edge cases.
```

### 4. **Mock System Conditions and Dependencies**
   - **Resource Limits**: Simulate low-memory or low-disk space conditions, which can cause unexpected behavior.
   - **Network Issues**: If your code depends on network calls, simulate conditions like slow responses, dropped connections, and intermittent availability. Libraries like **responses** or **requests-mock** in Python can help simulate these scenarios.
   - **File and Database Access**: Test cases where files or database entries are missing, locked, corrupted, or inaccessible due to permissions.

For functions that depend on external resources, you can use mocking to simulate scenarios like network failures or missing files.

```python
from unittest.mock import patch
import requests

def fetch_data(url):
    response = requests.get(url)
    if response.status_code != 200:
        raise ValueError("Failed to fetch data.")
    return response.json()

# Test to handle network error simulation
with patch("requests.get") as mock_get:
    mock_get.side_effect = requests.exceptions.ConnectionError
    try:
        fetch_data("http://example.com")  # Should raise a connection error
    except requests.exceptions.ConnectionError as e:
        print(f"Caught network error: {e}")
```


### 5. **Test for Exception Handling**
   - Check that your code catches and handles specific exceptions appropriately. This includes testing error-prone scenarios, like dividing by zero, accessing a non-existent file, or making invalid API calls. In essence, you want to verify that the correct exceptions are raised for invalid inputs. Here’s how to do that.

```python
def divide(a, b):
    if b == 0:
        raise ZeroDivisionError("Cannot divide by zero.")
    return a / b

# Tests
try:
    print(divide(10, 2))   # Regular case
    print(divide(10, 0))   # Edge case: division by zero, should raise an error
except ZeroDivisionError as e:
    print(e)
```

### 6. **Use Assertions for Accuracy and Robustness**
   - Assertions help verify that edge cases produce expected outputs or errors. Combine them with try-except blocks to catch edge-case failures.

### 7. **Monitor Code Paths with Coverage Analysis**
   - Coverage analysis tools like **coverage.py** can show which parts of your code are exercised by your tests. For rare edge cases, focus on paths that are less likely to be tested but could fail unexpectedly.

### 8. **Stress Test for Performance Edge Cases**
   - **Load Testing**: For code that may face high loads (like web applications), simulate high-concurrency conditions. This might involve sending many requests or performing many operations in rapid succession to ensure stability.

By designing your tests around these practices, you can systematically cover edge cases and identify rare failures before they become real-world issues.

<!-- ## Edge Cases Testing Examples ##

Examples of handling and testing edge cases in Python. These examples encompass  common edge cases, including boundary values, unexpected inputs, large data などなど.

### 1. **Handling Boundary Values**

Suppose we have a function that calculates the square root of a number. We should handle edge cases like negative numbers, zero, and very large numbers.

```python
import math

def safe_sqrt(x):
    if x < 0:
        raise ValueError("Cannot compute square root of a negative number.")
    return math.sqrt(x)

# Testing the edge cases
try:
    print(safe_sqrt(0))           # Edge case: zero
    print(safe_sqrt(1e10))        # Large number
    print(safe_sqrt(-10))         # Negative number, should raise an error
except ValueError as e:
    print(e)
```

### 2. **Testing Empty and Large Collections**

Consider a function that computes the average of a list. We'll handle edge cases like an empty list, a single-item list, and a very large list.

```python
def average(numbers):
    if not numbers:
        raise ValueError("Cannot compute the average of an empty list.")
    return sum(numbers) / len(numbers)

# Edge case tests
print(average([]))                # Empty list
print(average([5]))               # Single-element list
print(average([1] * 1000000))     # Large list (one million elements)
```

### 3. **Testing String Edge Cases**

For a function that processes strings, test with edge cases such as empty strings, very long strings, and special characters.

```python
def normalize_text(text):
    if not text:
        return ""
    return text.strip().lower()

# Edge case tests
print(normalize_text(""))                   # Empty string
print(normalize_text("   Text with spaces   "))  # Stripping spaces
print(normalize_text("1234567890" * 1000))  # Very long string
print(normalize_text("Hello!@#$%^&*()"))    # Special characters
``` -->

<!-- ### 4. **Using Parameterized Tests for Multiple Edge Cases (pytest)**

To streamline edge case testing, use **pytest** to run multiple scenarios. Here’s an example with `@pytest.mark.parametrize`:

```python
import pytest

def is_even(number):
    return number % 2 == 0

@pytest.mark.parametrize("number, expected", [
    (0, True),            # Edge case: zero
    (-2, True),           # Negative even number
    (1, False),           # Odd number
    (10**10, True),       # Large number
    (10**10 + 1, False),  # Large odd number
])
def test_is_even(number, expected):
    assert is_even(number) == expected
```

Run this with pytest to test all cases automatically. -->
<!-- 
### 5. **Using Mocking to Simulate Network or I/O Errors**

For functions that depend on external resources, you can use mocking to simulate scenarios like network failures or missing files.

```python
from unittest.mock import patch
import requests

def fetch_data(url):
    response = requests.get(url)
    if response.status_code != 200:
        raise ValueError("Failed to fetch data.")
    return response.json()

# Test to handle network error simulation
with patch("requests.get") as mock_get:
    mock_get.side_effect = requests.exceptions.ConnectionError
    try:
        fetch_data("http://example.com")  # Should raise a connection error
    except requests.exceptions.ConnectionError as e:
        print(f"Caught network error: {e}")
``` -->

<!-- ### 6. **Testing for Exception Handling**

In some cases, you want to verify that the correct exceptions are raised for invalid inputs. Here’s how to do that.

```python
def divide(a, b):
    if b == 0:
        raise ZeroDivisionError("Cannot divide by zero.")
    return a / b

# Tests
try:
    print(divide(10, 2))   # Regular case
    print(divide(10, 0))   # Edge case: division by zero, should raise an error
except ZeroDivisionError as e:
    print(e)
``` -->

### 7. **Testing Resource Constraints**

Here’s a way to test for handling large data inputs, often using libraries to help monitor memory usage in real-world testing.

```python
def sum_large_list(lst):
    return sum(lst)

# Edge case test: Large input
large_list = [1] * 10**7  # List with 10 million elements
print(sum_large_list(large_list))
```

<!-- ### 8. **Fuzz Testing with Hypothesis**

Using **Hypothesis** to generate a wide variety of input values, including edge cases automatically.

```python
from hypothesis import given, strategies as st

@given(st.integers())
def test_is_even_property(number):
    assert is_even(number) == (number % 2 == 0)

# This will test is_even with a wide range of integers, including edge cases.
``` -->