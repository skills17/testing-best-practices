# Testing Best Practices

A collection of best practices for automated testing in a championship environment.

This repository is championship independent and focuses on writing tests and the testing frameworks developed at [skills17](https://github.com/skills17).

## Table of Contents

- [General](#general)
- [Marks](#marks)
- [Extra Tests](#extra-tests)
- [Backend Specifics](#backend-specifics)
- [Cypress Specifics](#cypress-specifics)

## General

### Allow different solutions

Keep the tests as general and relaxed as possible, and don't focus on a specific way to implement a task as there are probably many other ways to do it.
Competitors should be able to implement a task as they want and in the way they think is the best for the current challenge.
Tests should not restrict competitors too much.

When possible, integration tests should be used over unit tests.
Integration tests verify the boundaries and test if a specific input leads to the expected output. They don't rely on implementation details.

### Keep tests simple

During a championship, time is very limited and competitors are in a hurry.
Therefore, it is important that tests are kept simple and easy to understand.
This also includes comments that explain what gets tested or additional assertion messages that make it easier to understand in case an assertion fails.

The goal is that a competitor understands what gets tested within a few seconds when looking at a test.

<details>
  <summary>Example</summary>

  ```js
  // test position of the title
  expect(title.left).to.be.closeTo(artist.left, 1, 'Expected the title to be aligned with the artist');
  expect(title.top).to.be.lessThan(artist.bottom, 'Expected the title to be underneath the artist');
  ```
</details>

### Repetition over dynamically generated tests/asserts

It goes into the same point as the previous one.
Sometimes it is faster for writing the tests to dynamically generate tests or asserts, e.g. by using a for-loop.
But this makes it harder to understand for competitors, and it may not always be obvious where tests and asserts come from or for which values they are failing.

Because of that, it is preferred to repeat tests and asserts instead of dynamically generating them.
It may be more work while creating tests but makes life easier for the competitors.

<details>
  <summary>Example</summary>

  ```js
  // this should be avoided
  for (const user of users) {
    expect(user.name).to.equal(aliases[user]);
    if (user.name.startsWith('admin')) {
      expect(isAdmin(user)).to.equal(true);
    } else {
      expect(isAdmin(user)).to.equal(false);
    }
  }
  
  // this is preferred instead
  expect(users[0].name).to.equal('john');
  expect(isAdmin(users[0])).to.equal(false);
  
  expect(users[1].name).to.equal('doe');
  expect(isAdmin(users[1])).to.equal(false);
  
  expect(users[2].name).to.equal('admin-john');
  expect(isAdmin(users[2])).to.equal(true);
  ```
</details>

### Keep tests independently

Each test should only test a single feature and should not depend on features that are also tested in other tests.
If a competitor fails to implement one feature, it should only fail one test and not others.

One prominent example is the login functionality.
If all tests only work if the login is implemented correctly, a competitor would receive 0 points if they were not able to implement the login functionality correctly.
But they may have implemented all other features of the tasks. 

In that sense, tasks should be designed from the ground up, such that nearly every feature can be tested without a correct implementation of another feature.

### Avoid asserts in setup/teardown

If an assertion fails within setup/teardown (e.g. `beforeEach`), then _all_ tests will fail.
Therefore, it should be avoided to have assertions in such methods or if necessary, carefully check if there is any unexpected way that could fail those assertions.

## Marks

### Difficulty distribution

An ideal task, when solved by many competitors, creates a point distribution that satisfies these points:

- The best competitor achieves close to, but not quite 100%. This both gives the best competitor satisfaction of being
  able to almost finish, while knowing that there still are some percentage points to gain. It also lets everyone else 
  know that the task was not outrageously hard and impossible, as the winner almost completed 100%.
- The best competitors achieve distinguishable scores that separate them. To be able to figure out the very best from 
  the best, the top scores should be incrementally harder to achieve.
- Every serious competitor reaches at least roughly 30% of the marks. This means there are roughly 30% of points achievable by 
  knowing the bare minimum. This gives everyone that participated a feeling that they were able to do at least "something".
- The midfield of scores should have an equidistant difficulty. This makes competitors feel the progression through the 
  points and separates faster vs. slower competitors. 
- There are no jumps in scores at certain thresholds. The difficulty should never jump so that most competitors get 
  stuck at the same place, ending up with the same score are most others. 

### Points should be awarded equally over the whole championship

Across the whole championship, points for tests should be awarded equally for the effort and time required.
A redistribution of the points when all tasks have been written is probably needed, since the tasks are often crafted
by individual people lacking an overview of all points awarded.

## Extra Tests

Extra tests are a way to identify possible cheating by returning static values that satisfy the tests without actually implementing logic.
Those extra tests are not distributed to competitors but only used by experts during marking.

If a normal test succeeds while an extra test fails, experts should manually verify the competitor's solution to check for possible cheating attempts.

### Don't test additional requirements

Extra tests should not test additional requirements that were not already tested by a normal (non-extra) test.
Instead, extra tests should be copies of the normal tests but with different assertion values.

Competitors only get the normal tests, so they don't know about other requirements that are possibly checked in an extra test.
If any valid implementation passes the normal tests, it should also pass the extra tests and extra tests should only fail for cheating attempts, not wrong implementations.

## Backend Specifics

### Anticipate infrastructure instability

The underlying infrastructure that runs the server might be under heavy load and produce test timeouts if they are too tight.
Generally, timeouts should be set to __very__ high durations like 2 minutes for simple calls.

### Handle HTTP 429 Too Many Requests and others

The testing framework should automatically retry certain tests with a backoff time if they fail. Reasons for failing
might be framework built-in request throttling or timeouts (Laravel contains API throttling as standard).

### Paths and URLs configurable

The competitors might implement a correctly working backend, but the URLs or paths are slightly off. E.g. a PHP developer
might have every URL ending in `.php`. To be able to still award points in an automated fashion, the tests should be
configurable with base URL and path pre- and postfix. Such solutions should however still have points deducted for not
adhering to the URL and path scheme.

## Cypress Specifics

### Specify entry points

To reduce the amount of tests that dependent on previous functionality, tasks should be designed that each page can be
reached via its own URL entrypoint. It should not be required to step through a process in a web app to reach a certain point
where more features are tested. If the competitors fail at implementing the first step of the process, but successfully
implement the second step, the test for the second step still fails if it has to click through the first step to reach step two.

### Use `find` instead of `children`

The `children` function will only find immediate children but the competitors may additionally nest the HTML elements.
`find` searches in the whole subtree of the element instead of just the immediate children and is therefore preferred.

This [allows different solutions](#allow-different-solutions) for end-to-end tests.

<details>
  <summary>Example</summary>

  ```js
  // this should be avoided
  cy.get('.breadcrumbs').children('.active');
  
  // this is preferred instead
  cy.get('.breadcrumbs').find('.active');
  ```
</details>

### Append `:visible` to avoid checking hidden elements

Competitors may have hidden elements in the DOM that match the same selector.
This can especially be the case for elements with visibility that can be toggled or when DOM elements are dynamically generated, e.g. from data loaded from an API.

Appending `:visible` will make sure that no such hidden elements will match and only the elements that are actually visible to the user are tested.
