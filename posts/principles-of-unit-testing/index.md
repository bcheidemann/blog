---
title: Principles of Unit Testing
categories:
  - Testing
articleId: 3
---

In this article we'll do a deep dive into the art and science of unit testing. This is a topic fraught with controversy, with popular developer-influencers taking diametrically opposed viewpoints. Many advocating for 100% code coverage, and many more advocating for no unit tests at all. We'll explore the basic principles behind unit testing and discuss the implications for some practical scenarios, so that you can make the right decision on unit testing for yourself and your project.

---

## Introduction

Much of the controversy around unit testing stems from the human desire to attribute concrete answers to vaguely defined questions. People might, and indeed often do, ask questions like "should my code be covered by unit tests?", and "are unit tests good?". These types of questions tend to spark heated debates in online forums, as people with vastly different backgrounds tend to weigh in, neglecting to disclose their personal biases.

In reality, these kinds of questions are far to vague to have any real meaning independent of an individuals context. Occasionally, commenters will acknowledge this and attempt to caveat their answers, or bound them within a specific context. For instance, a commenter might respond to the question of whether one should unit test their code with "yes, if you're working on mission critical code" or "not if you're working at a startup".

It is my opinion that the usual questions around unit testing and the answers to those questions generally stem from a lack of knowledge of software design principles. This is to be expected, since the principles behind software design are often ignored in the practical setting of real world companies.

In this article, I aim to take a step back from the concrete and consider the principles of software development as they apply to unit tests. Although I may occasionally reference research and other literature, this is very much an opinion piece. I very much encourage anyone reading this to do further reading beyond this article to get a fuller picture of the topics discussed. To this end, I have added a [further reading](#further-reading) section.

I think it's also important to note that I am approaching this article with my own personal biases; I am a software developer who has worked mostly on bleeding edge greenfield projects with a few stints working on legacy software projects. I work on both the front-end and backend, and occasionally dabble with infrastructure. My day job primarily involves writing TypeScript, though I enjoy working with a range of other languages including Rust and Java in my free time. I will try to keep my biases from coloring this article too much, but please bear in mind that I am but human when you come across an opinion you find objectionable 😇

This article covers some of the important principles behind unit testing. Many of these principles are general principles of software development, but we will be viewing them through the lense of unit testing.

---

## Test Structure

Although it's not particularly important in the context of this article, we'll briefly cover the test structure we'll be using in the code snippets included in this article. Not that all code snippets will be in the style of [Jest](https://jestjs.io/).

### BDD Style

Generally, I prefer to structure my tests in a BDD style, as is common in the JavaScript ecosystem. This is characterized by `describe` and `it` blocks, where each `it` block represents one test case and a `describe` block wraps one or more `it` blocks which relate to the same unit of code. For example:

```ts
describe('someFunction', () => {
  it('should do x', () => {
    // Test implementation
  });

  it('should do x', () => {
    // Test implementation
  });
});
```

### Arrange, Act, Assert

I also prefer to separate the test implementation into distinct parts, in an effort to improve readability.

1. Arrange - Prepare all of the data, state and mocks for the test
2. Act - Do some action
3. Assert - Test that the action resulted in the expected output and side-effects

For example:

```ts
describe('isUserActive', () => {
  it('should return true when the user status is active', () => {
    // Arrange
    const user = userFactory.build({ status: 'active' });

    // Act
    const result = isUserActive(user);

    // Assert
    expect(result).toEqual(true);
  });
});
```

---

## Coupling

Software coupling refers to the level of interdependence between two or more parts of a software system which interact with one another. It is an important concept to be aware of at any scale, and applies to the relationships between everything from microservices down to the individual functions which make up your application.

Two interacting components can either be tightly or loosely coupled. Neither is necessarily a better or worse, and the level of coupling that is desirable between components depends largely on the context. A good litmus test for coupling between two components is the likelihood that a change in one requires a change in the other.

Generally speaking, we want our unit tests to be loosely coupled to the code they are testing. This way, when we change the implementation of our code, we don't need to significantly change our unit tests. A good test to determine if your unit test is sufficiently loosely coupled would be to replace the implementation with a stub. Ideally, your unit test should still passes.

This requires us to place certain constraints on the implementation of unit tests.

1. We should only test the public interface (i.e. the methods and function which are invoked by other parts of our code)
2. We should only assert outputs and side-effects

Respecting these constraints ensures that our test will only fail when the behavior of the code being tested changes.

Although conceptually quite simple, in practice it is easy to fall foul of these principles for a variety of reasons. Some common ways in which developers inadvertently cause coupling between test cases and implementation are:

1. Over reliance on mocking (e.g. mocking private methods)
2. Asserting against implementation details (e.g. asserting internal state or private methods invocations)
3. Implementation duplication (e.g. copying parts of our implementation)

It should be noted that in certain cases, a degree of coupling is unavoidable - or at least a worthwhile/necessary tradeoff vs structuring your code differently. In these cases, unit tests should not test any more of the implementation than is absolutely necessary.

### Knowing when (not) to Mock

Mocking is an important part of unit testing. This is because it is required to test any unit of code which has one or more interaction points with another part of our system, or with an external system.

For instance, a method to create a user would be fairly useless if it didn't make a call to our database to create a user record. It is not sufficient to assert the outputs of this method (i.e. the return value), we must also test the side-effects (i.e. the database action).

One way to test the side effects would be to spin up an instance of our database every time we run our tests. However, this is neither practical, nor desirable because our unit test should not depend on external parts of our system. Instead, these should be tested by other unit tests. This has the following benefits.

1. #### Isolation
   Mocking enables us to isolate our test cases from unrelated changes to other parts of our application. These should be covered by different unit tests, so there is no need for unit tests covering consuming code to also fail. Isolating our tests in such a way that they only fail when the behavior of the unit under test changes ensures developers can quickly and efficiently identify issues and avoid spending time debugging working parts of the system.

   *Note: It is important to test for changes in the interfaces between code, particularly when writing isolated tests. Ideally, this should be handled by static type checking, rather than by unit tests.*

2. #### Simplicity
   By mocking external components of our system, we simplify our test setup and tear down. For example, if we want to test the resilience of our code to exceptions from an external component (e.g. database errors), we can simply mock a rejection, rather than trying to find a way to cause a rejection some other way.

Despite the benefits of mocking certain parts of our code, mocking the wrong parts can cause problems. The most common example is mocking methods which are not part of the public interface. This can be tempting, since mocking a private method can sometimes simplify test cases. However, it has the following downsides.

1. #### Implementation Coupling
   An over-reliance on mocking couples our test case to a specific implementation. This is particularly pronounced when we mock parts of our internal API (e.g. private methods), since we are much more likely to refactor this than our public interface.

2. #### Degraded Test Sensitivity
   Mocking has the downside that it "pretends" to our test that parts of our code behave in a certain way. When the behavior of the mocked code changes, this may have an impact on the behavior of the tested code. However, because we mocked the changing code, our test may not detect unintentional changes in behavior. This is why we typically only want to mock parts of our code which are: covered by their own unit tests, loosely coupled to the software unit under test, and where the interface is strictly enforced (e.g. by static type checking). For this reason, it is best to avoid mocking internal implementation details such as private methods. 

In summary, mocking certain parts of our code is essential to ensure our unit tests do not become tightly coupled to unrelated parts of our code. However, excessive mocking can result in tightly coupling our code to the implementation of our code and degrade the quality of our test.

*Note: Following the principles set out above means our unit tests cannot ensure that all our individual components come together to form a functional system. Although this is also important, testing the macro-level behavior is the remit of end-to-end testing, not unit testing.*

### Implementation Assertions

In it's least harmful form, test coupling typically involves asserting the implementation of the unit under test, often by asserting certain private methods are called (e.g. with spies). This is unnecessary and adds no value to our test, but is not per-se harmful.

The primary downside of this kind of implementation testing is that it makes our tests more fragile, causing them to fail unnecessarily when refactoring our code, or otherwise making changes to it.

However, arguably the most harmful cases of test coupling are those in which some, or all, of the implementation of the tested unit is duplicated in the test.

### Implementation Duplication

Implementation duplication involves copying (either verbatim or semantically) the implementation details from the tested unit into our unit test.

This kind of coupling, while fooling code coverage metrics, often fails to assert anything about our code, other than that it does what it does.

To better understand this, let's pretend we're building a new messaging app. It's super exclusive, so users can only sign up if they are invited by an existing user. However, in order to mitigate against bots being created which send out invitations to anyone, we want to be able to revoke a users permission to send invites. So we decide to implement a permissions system.

```ts
// src/permissions.ts
export enum UserPermissions {
  InviteUsers = 'InviteUsers',
  SendMessages = 'SendMessages',
}
```

```ts
// src/services.ts
export class UserService {
  private usersRepository: IUsersRepository;

  public async createUser(name: string) {
    await this.usersRepository.put({
      name,
      permissions: Object.values(UserPermissions),
    });
  }
}
```

By default, all new users should be able to send messages and invite new users. Since these are the only permissions we have at the moment, we decide to make use of `Object.values` so that any new user automatically has all the permissions enabled.

Now let's write our test case:

```ts
// src/__test__/services.test.ts
describe('UserService', () => {
  let instance: UserService;

  beforeEach(() => {
    // Instantiate user service with mock users repository
  });

  describe('createUser', () => {
    it('Should create the user in the users repository', async () => {
      // Arrange
      const name = 'Mr Test';

      // Act
      instance.createUser(name);

      // Assert
      expect(instance.usersRepository.put).toHaveBeenCalledWith({
        name,
        permissions: Object.values(UserPermissions),
      })
    });
  });
});
```

Can you spot the mistake in our test? It's quite subtle so maybe it's not obvious yet. Let's walk through a scenario which will hopefully reveal the issue.

Let's assume we've launched our messaging app. Naturally, it's a roaring success! However, for internal purposes, we want to introduce a "super-admin" user type which can delete or ban existing user accounts.

Since we've already implemented a permissions system, we decide to re-use it by implementing a "super-admin" user permission.

```ts
// src/permissions.ts
export enum UserPermissions {
  InviteUsers = 'InviteUsers',
  SendMessages = 'SendMessages',
  SuperAdmin = 'SuperAdmin',
}
```

In a separate part of the code, we then check for the presence of this permission whenever a user tries to delete or ban another user.

Upon finishing the implementation of this feature and testing it locally, we run our unit tests to make sure we didn't break any existing functionality inadvertently. Since everything passes, we raise a PR and deploy our code to our test environment.

Having now deployed our code to the test environment, QA confirms that an existing user account (which doesn't have the permission) is unable to delete or ban other users. When the permission is added, the user becomes a "super-admin" with the power to delete or ban anyone on the platform. Tests passed, next stop, production!

*Success! ... right?*

Not so fast! Some time after deploying the new feature to production, we decide to run an audit. Surprisingly, we find that we have many more super admins than expected. In fact, it seems that every single user account created after we released the super-admin feature, has the new permission. What wen't wrong?

Well, as you may already have guessed, the `UserService.createUser` method is at fault here, since it blindly adds every single permission to any newly created user.

```ts
// src/services.ts
await this.usersRepository.put({
  name,
  permissions: Object.values(UserPermissions),
  //          |____________bug_______________|
});
```

This probably isn't an ideal implementation, but it's virtually a 100% guarantee that not all of the code in a sufficiently sized project will be implemented in an ideal manner. This is an inescapable reality of software development, and a problem outside the scope of this article.

The bigger issue here is that ***our unit test didn't catch this error***.

If we examine the unit test, we can see that it duplicates the exact same logic as the `createUser` service method.

```ts
// src/__test__/services.test.ts
expect(instance.usersRepository.put).toHaveBeenCalledWith({
  name,
  permissions: Object.values(UserPermissions),
  //          |____________bug_______________|
})
```

At first glance, this seems fairly innocuous, and could easily slip past a code review. However, because our test is duplicating the logic of our service method, when the functionality of the service method changes, what our test is testing will also change.

Occasionally, when writing a unit test, we don't care about certain parts of our output. At this time, we may be tempted to match the implementation logic, precisely so that our test doesn't fail when the tested method changes in a way unrelated to the test.

This is a bad practice, since most testing frameworks offer solutions for this out of the box. Using these solutions, we make it explicit that we don't care about a certain output, and it becomes easier to spot mistakes in the test implementation.

For example, Jest offers the `expect.any()` helper, which can be used as follows:

```ts
describe('ExampleService', () => {
  describe('exampleMethod', () => {
    it('Should test that some data we care about is returned', async () => {
      // Arrange
      const input = 'example input';

      // Act
      const result = instance.exampleMethod(input);

      // Assert
      expect(result).toEqual({
        someDataWeCareAbout: 'explicit value',
        someDataWeDoNotCareAbout: expect.any(),
      })
    });
  });
});
```

In the above test case, we explicitly signal that the test ***does not*** care about the value of `result.someDataWeDoNotCareAbout`.

## Precision

I'm sure most developers have encountered long sprawling test cases which make multiple assertions. To some extent, this may be fine for other forms of testing (e.g. integration/end-to-end testing), but unit tests should be very precise in their assertions. This improves reporting, reduces duplication, allows us to quickly narrow down the cause of test failures, and keeps our tests focused and readable.

---

## Writing Cohesive Tests

Software cohesion is a term used to describe how well elements of a software unit relate to each other. Generally, we want our software to be cohesive, as it makes it more intuitive and easier to understand.

Much like other parts of our codebase, we should aim to organize our test cases in a cohesive manner. This means that we should group test cases so that similar tests are in close proximity. BDD style is one way to organize our unit tests cohesively, as it allows us to group our test cases (`it` blocks) into groups which `describe` the behavior of a unit of code.

Generally, we want to mirror the structure of our application when structuring our unit tests, so assuming our application has high cohesion, our test suite will also tend to have high cohesion.

---

## Writing Isolated Test Cases

Test interdependence is an issue that i've occasionally run into in almost every test suite I've worked on. When tests are dependent on one-another, it means one test depends on another test to run, in order for it to run correctly. This means that when tests cases are dependent on one another, we might find that certain tests pass when executed in one order, but when we execute our test cases in a different order, the some test cases might fail. This commonly becomes apparent when trying to debug a single test case.

One trivial example of how this may happen, is when mocks are not properly reset between test cases. Take the following example:

```ts
describe('UserService', () => {
  const mockUserRepository = {
    getUserByUsername: jest.fn(),
    createUser: jest.fn(),
  };

  describe('createUser', () => {
    it('should create a user', async () => {
      mockUserRepository.getUserByUsername.mockResolvedValue(null);

      await userService.createUser('user123');

      expect(mockUserRepository.createUser).toHaveBeenCalled();
    });

    it('should not create a user when a user with the same username already exists', async () => {
      const user = userFactory.build();
      mockUserRepository.getUserByUsername.mockResolvedValue(user);

      await userService.createUser(user.username);

      expect(mockUserRepository.createUser).not.toHaveBeenCalled();
    });
  });
});
```

In this example, our second test will pass when run in isolation. However, if we run our first test case before the second, it will fail. This is because the `mockUserRepository.createUser` mock function has not been reset after our first test case, and the assertion that it was not called will no longer hold.

This is a fairly harmless example, since it will be caught as soon as we run our full test suite. However, this kind of interdependence can be established whenever our test cases rely on some shared mutable state. This can have a significant impact on developer productivity, since it is possible for test interdependence to go unnoticed for a long time. Given poor testing hygiene, these issues can accumulate, until someone removes or modifies a dependency, which may cause many downstream test cases to fail inexplicably.

There are strategies which can be employed to ensure test cases are isolated from one-another. The simplest way to guarantee test isolation, is to never depend on shared state. However, depending on the language used and size of the test suite, repeatedly creating identical mock objects may have a non-negligible performance impact, and can also make our test cases less DRY. The next best option is to never mutate shared state. This is a good option, since it also guarantees isolation. However, there are reasons why it might be desirable to temporarily mutate shared state (e.g. mock functions). In these cases, it is important to ensure that shared state is always properly reset between test cases. Ideally, this would be done by some automated mechanism, since it ensures that developers cannot forget to reset state. Most good testing frameworks will have some functionality for this built in. For instance, [Jest](https://jestjs.io/) has options like `resetMocks` and `restoreMocks` which ensure that any mock functions created using `jest.fn()` will be properly reset between test cases, and their default implementations restored.

---

## Writing Encapsulated Tests

Encapsulation, in the context of software development, means to group data with the operations that act on it. It is a feature of object-oriented languages like Java and Python, but is not exclusive to those languages. It is important to note that encapsulation is a general concept; it is not the same thing as classes, though classes are one example of encapsulation.

In the context of software testing, we can consider a test case to demonstrate encapsulation, when it has ownership of all of the data that it operates on. For example, the following test cases demonstrate encapsulation:

```ts
describe('User', () => {
  describe('joinGroup', () => {
    it('should add the user to the group', () => {
      const user = new User();
      const group = new MockGroup({
        users: [],
      });

      user.joinGroup(group);

      expect(group.users).toHaveLength(1);
    });

    it('should not add the user to the group if they have already joined it', () => {
      const user = new User();
      const group = new MockGroup({
        users: [user],
      });

      user.joinGroup(group);

      expect(group.users).toHaveLength(1);
    });
  });
})
```

While the following test cases do not demonstrate encapsulation:

```ts
describe('User', () => {
  describe('joinGroup', () => {
    const user = new User();
    const group = new MockGroup({
      users: [],
    });

    afterEach(() => group.reset());

    it('should add the user to the group', () => {
      user.joinGroup(group);

      expect(group.users).toHaveLength(1);
    });

    it('should not add the user to the group if they have already joined it', () => {
      user.joinGroup(group);
      user.joinGroup(group);

      expect(group.users).toHaveLength(1);
    });
  });
})
```

The primary advantages of test encapsulation are: ensuring [test isolation](#writing-isolated-test-cases), test clarity/readability, and reduced fragility.

I personally think it's useful to further abstract our concept of encapsulation, to also include the declaration of state. To understand how this might be beneficial, let's look at the following example:

```ts
describe('AuthService', () => {
  const mockUserRepository = {
    getUserByUsername: jest.fn(),
  };

  describe('login', () => {
    it('should return an auth token and a refresh token when an active user attempts to login', async () => {
      const user = userFactory.build();
      mockUserRepository.getUserByUsername.mockResolvedValue(user);

      const result = await authService.login(user.username, user.passwordHash);

      expect(result.authToken).toBeJwtToken();
      expect(result.refreshToken).toBeJwtToken();
    });
  });
});
```

Although this test technically encapsulates all of the state on which it operates, the declaration of the state is being performed by code outside of the test case (the `userFactory`). This is fine for data which is not relevant to our test case (e.g. username, passwordHash, phoneNumber, etc), but because our test case depends on the users status being active, this data should be explicitly declared by our test case. We can modify our invocation of `userFactory` to explicitly declare the users status within our test case:

```ts
describe('AuthService', () => {
  const mockUserRepository = {
    getUserByUsername: jest.fn(),
  };

  describe('login', () => {
    it('should return an auth token and a refresh token when an active user attempts to login', async () => {
      const user = userFactory.build({
        status: 'active',
      });
      mockUserRepository.getUserByUsername.mockResolvedValue(user);

      const result = await authService.login(user.username, user.passwordHash);

      expect(result.authToken).toBeJwtToken();
      expect(result.refreshToken).toBeJwtToken();
    });
  });
});
```

This make the meaning of our test explicit, and reduces the fragility of the test, since changing the default status of users generated with the `userFactory` will no longer cause the test to fail.

All things being equal, it is generally a good thing for tests to encapsulate all of the data on which they depend and operate on. However, judgement should be exercised to balance the benefits of encapsulation against test suite performance and verbosity.

---

## Efficiency & Iteration Speed

<!--

Tests should not take long to run. If it's worth getting a coffee while you wait for your test suite to run then it's too slow. This could be an indication that your application is overly coupled since ideally a system with this many unit tests would be modular and loosely coupled and so the test suite could be broken down into smaller test suites for each unit. Alternatively, you may be doing unnecessary work as part of your unit tests (e.g. type checking in typescript).

Tests must also run quickly enough that you can efficiently iterate. This is essential for practicing TDD but still important even when writing tests after the fact. The goal here is that on any given change, developers are able to re run only the affected tests. This way developers can commit frequently and only run a small number of tests for any given change, encouraging writing tests alongside code changes rather than as a reluctant afterthought. The entire test suite can be run either on PR or on merging to the trunk branch.

 -->

---

## Sensitivity & Specificity

Sensitivity and specificity are terms I first learned in medical school. However, they apply equally well to software testing as they do to medicine. They are defined as follows.

 - **Sensitivity** (a.k.a. true positive rate) is the probability of a positive test result, given the condition being tested for is truly positive.
 - **Specificity** (a.k.a. true negative rate) is the probability of a negative test result, given the condition being tested for is truly negative.

![Sensitivity & Specificity](./specificity-sensitivity.svg)

The above diagram is a visualization of sensitivity and specificity, and two closely related terms, positive predictive value (a.k.a precision) and negative predictive value.

In medicine, tests check for a specific ***undesirable*** result (e.g. that a patient has a disease), while software tests check for a specific ***desirable*** result (e.g. that for certain inputs we get the correct outputs). However, with a little mental gymnastics, we can reframe software tests to better fit into the sensitivity/specificity model. Instead of thinking of software tests as asserting a specific behavior, we can think them as checking if a system ***does not*** behave as expected, given certain inputs. In this sense, a positive test result would be when the test fails, and a negative result would be a passing test.

An ideal unit test would always fail (return a positive result) when the unit of code being tested behaves incorrectly, and always pass (return a negative result) when the unit of code behaves correctly. In other words, much like medical tests, it should have high sensitivity and specificity.

Achieving a balance between sensitivity and specificity is key to writing a good unit tests, and both metrics are important to a healthy test suite. Unfortunately, these metrics are non-trivial to measure objectively in software development and often overlooked, in favour of optimizing arbitrary metrics like code coverage, which tell us little about the quality or usefulness of our test suite. In the rare cases that these metrics are considered, emphasis will generally be given to sensitivity, with little regard for specificity.

Either way, this can lead to fragile test suites. To understand why this is a problem, we'll pretend we're dealing with medical diagnostic tests for a moment.

Consider a diagnostic test for a hypothetical disease X. Our test has a sensitivity of 90%, specificity of 50% and disease X has a prevalence of 10%. Assuming we administer our test 100 times, we would get the following results:

```
Actual Number of Cases = 100 * 10% = 10
Positive Test Results = 10 * 90% + 90 * 50% = 9 + 45 = 54
```

This means that in 54% of our cases we need to do further evaluation to confirm that the patient actually has disease X, only to find that for 45 of these cases were false positives.

In software testing, this effect is even more pronounced, since we are often running hundreds (or even thousands) of different test cases. Even when you consider that not every test will be relevant to every code change, we are still likely to have dozens of tests which could be relevant to any given code change.

Although not efficient, one might make the argument that this is still an effective test suite, because at least we're catching almost all the bugs before they get to production. But a test case failing is not the same thing as catching a bug; when a majority of tests failing are not indicative of a bug, our brain quickly becomes desensitized to failing test cases. This risks developers prematurely dismissing failing test cases as "broken" and simply "fixing the test", instead of taking the time to understand why it's failing.

Furthermore, when low specificity test cases are repeatedly refactored, it risks changing the meaning of the test in unintended ways. It's true that refactoring a bad test can be an opportunity to improve the test, but equally it can degrade the quality of the test when the developer doing the refactoring is unfamiliar with the code unit being tested. Depending on the level of familiarity and the care taken by the developer(s) refactoring failing test cases, over time, repeated refactoring can degrade the quality of our test suite and reduce its sensitivity.

Aside from the direct consequences of low specificity tests, as described above, there are also a number of indirect consequences. Some examples include:

 1. Reduced team morale due to more time spent on test maintenance
 2. New features take longer to implement due to more time spent on test maintenance
 3. PRs tend to contain more code changes due to unnecessary refactoring of existing low specificity tests
 4. PRs tend to include changes to test cases which are not directly related to the code change, making them harder to review
 5. Developers are less likely to spend time writing high quality tests, to compensate for the time they need to spend refactoring existing tests

---

## Metrics for Unit Testing

Software development metrics come in many shapes and sizes, from simple metrics like lines of code, to more complex metrics like cyclomatic complexity [[1]](#references). These can be categorized into internal metrics (e.g. coupling, modularity, test coverage level) and external metrics (e.g. cost, quality, reliability). Generally, software teams want to optimize for external metrics, since these generally determine the success or failure of a commercial project. However, it is generally easier to directly measure and control internal metrics [[2]](#references).

Internal metrics such as test code coverage have been observed to be predictive of external metrics such as quality [[3]](#references). This can lead  development teams to try and optimize for certain internal metrics, with the hope that it will improve external metrics. However, Goodhart's law suggests that treating a measure as a target can have unintended consequences and reduce the validity of the measure [[4]](#references).

Although I am not aware of any published research papers into the validity of Goodhart's law in the context of software development, my personal experience across a range of projects leads me to believe that enforcing strict code coverage requirements tends to lead to lower quality tests.

With that said, I am generally a proponent of metrics informed development, and in this regard, testing is no exception. My recommendation would be to collect and publish relevant metrics, but resist enforcing arbitrary cutoffs. Developers should have easy and detailed access to metrics while having the freedom to exercise their discretion as to their application.

This is especially important during code review, since an abnormal metric may be an indication of a problem. For instance, if the code coverage is unexpectedly low for a given change, this could be an indication that an important test case is missing.

It is generally more useful to have ready access to a fine-grained breakdown of the metric in question, than to have the CI pipeline fail due to crossing some arbitrary project-wide cutoff. For instance, a line-by-line view of code coverage may help to identify any code which is not covered by a test case and allow code reviewers to make a judgement on wether covering that code would provide sufficient value to justify the cost of implementing and maintaining a test case for it.

Furthermore, in some cases, project-wide metrics can be detrimental. For instance, when removing a well tested, but overly complex module, in favour of a new, leaner module with half the lines of code, this may cause the overall project code coverage to go down in coverage, even if the new code has 100% coverage. This may cause developers to "forget" to delete unused code, and can cause a codebase to become cluttered over time, or to introduce tests for unrelated code to try and boost the code coverage, which delays delivering the feature being developed unnecessarily.

Metrics which have particular relevance to unit testing are covered in the following subsections.

### Code Coverage

As already discussed, code coverage is an important metric as it helps to identify where parts of a codebase are not sufficiently covered by test cases.

There are several types of code coverage [[5]](#references).

1. **Function coverage** - the number of functions executed by one or more test case
2. **Statement coverage** - the number of statements executed by one or more test case
3. **Branch coverage** - the number of branches executed by one or more test case
4. **Condition coverage** - the number of conditions who's truthy and falsy cases have been covered by one or more test case
5. **Line coverage** - the number of lines executed by one or more test cases

Each of these provide useful information about our software system and the unit tests we have implemented for it.

### Cyclomatic Complexity

Cyclomatic complexity is a measure of the number of linearly independent paths in a program module [[6]](#references). In simpler terms, it can be thought of as being a measure of how much your code branches.

For instance, the following test cases have a cyclomatic complexity of 1, because there are no branches.

```ts
describe('UserService', () => {
  describe('getUserById', () => {
    it('should return an active user', () => {
      const user = userFactory.build({
        id: 1,
        status: 'active',
      });
      repository.getActiveUserById.mockResolvedValue(user);

      const result = service.getUserById(1);

      expect(result).toEqual(user);
    });

    it('should return null if users status is not active', () => {
      const user = userFactory.build({
        id: 1,
        status: 'suspended',
      });
      repository.getActiveUserById.mockResolvedValue(user);

      const result = service.getUserById(1);

      expect(result).toEqual(null);
    });
  });
});
```

In my experience, a common mistake made by developers who are new to unit testing, is to introduce branches into their test cases. This is generally for one of two reasons:

1. Trying to cover multiple scenarios with a single test
2. Trying to perform type narrowing (common in TypeScript projects)

The following test attempts to cover both of the scenarios from the above example, and has a cyclomatic complexity of 2.

```ts
describe('UserService', () => {
  describe('getUserById', () => {
    it('should return an active user or null', () => {
      const user = userFactory.build({ id: 1 });
      repository.getActiveUserById.mockResolvedValue(user);

      const result = service.getUserById(1);

      if (user.status === 'active') {
        expect(result).toEqual(user);
      }
      else {
        expect(result).toEqual(null);
      }
    });
  });
});
```

Because `userFactory` is deterministic (it will produce the same value on consecutive runs), our test will only ever cover a single branch. At a glance, it is impossible to tell which scenario is covered by the test, and worse still, it implies that both code paths are covered, giving us a false sense of security.

Analyzing the cyclomatic complexity our unit tests is a quick and easy way to highlight problematic test cases. As a metric, it works nicely in concert with code coverage, which highlights all untested branches, but does not highlight specific test cases.

In certain situations, conditional logic may not be an indication of a faulty test case. For instance, [Jest](https://jestjs.io/) provides functionality for reducing repetition in similar test cases via `it.each`.

```ts
describe('UserService', () => {
  describe('getUserById', () => {
    it.each`
      status           shouldReturnNull
      ${'active'}      ${false}
      ${'suspended'}   ${true}
      ${'banned'}      ${true}
      ${'onboarding'}  ${true}
    `('should return an active user or null', (status, shouldReturnNull) => {
      const user = userFactory.build({
        id: 1,
        status,
      });
      repository.getActiveUserById.mockResolvedValue(user);

      const result = service.getUserById(1);

      if (shouldReturnNull) {
        expect(result).toEqual(null);
      }
      else {
        expect(result).toEqual(user);
      }
    });
  });
});
```

The above test works well and covers our code much more thoroughly than the original implementation. However, it's still harder to read and understand due to the conditional logic. We can refactor it as follows to improve readability, while still maintaining the tests coverage.

```ts
describe('UserService', () => {
  it('should return an active user', () => {
    const user = userFactory.build({
      id: 1,
      status: 'active',
    });
    repository.getActiveUserById.mockResolvedValue(user);

    const result = service.getUserById(1);

    expect(result).toEqual(user);
  });

  describe('getUserById', () => {
    it.each`
      status
      ${'suspended'}
      ${'banned'}
      ${'onboarding'}
    `('should return null if users status is not active', (status) => {
      const user = userFactory.build({
        id: 1,
        status,
      });
      repository.getActiveUserById.mockResolvedValue(user);

      const result = service.getUserById(1);

      expect(result).toEqual(null);
    });
  });
});
```

### Test Execution Time

Unlike the previously discussed metrics, test execution time is not necessarily a code quality metric. However, it is centrally important to enabling good testing practice, since it directly impacts the developer experience when writing tests.

Depending on your view of unit testing, you may prefer to practice test driven development, or you may prefer to write your tests after the fact. Regardless of when you write your tests, it is important that unit tests run quickly. A long-running test suite can (and should) be mitigated somewhat by good tooling, allowing developers to run only a subset of tests (ideally "affected" tests). However, at some point, the whole test suite needs to be run. When this takes an especially long time, it can discourage iteration and makes the process of writing tests slow and cumbersome. This not only impacts test quality, but also developer morale.

A long running test suite can also increase the risk of breaching service-level agreements (SLAs). Often, companies will have SLAs which require them to fix critical production issues within a pre-defined time period. If, for example, the SLA requires a critical production issue to be fixed within 1 hour, it makes a huge difference if the test suite takes 30 minutes or 5 minutes to run in CI.

What constitutes a long running test suite varies significantly depending on the scope and complexity of the project. For example, the main test suite for [v8](https://v8.dev/), which tests compliance to the ECMA262 specification [[7]](#references) takes around 37 minutes to run on a 2015 MacBook Pro [[8]](#references). This is pretty reasonable, considering there are over 74 thousand test cases being executed [[7]](#references). However, the test suite for smaller applications would be expected to run significantly faster.

From personal experience, I find that the point at which I find a test suite to be frustratingly long-running varies depending on the context. However, for large and well tested TypeScript applications, I think less than 5 minutes is a good benchmark.

There are many tricks that can be employed to speed up a test suite, but these largely depend on the language and tools being used, and are beyond the scope of this article.

### Sensitivity - Mutation Survival Rate

Code mutation testing is a way of testing the sensitivity of tests to code changes. It was originally proposed and developed in the 1970s and 80s [[9]](#references). In mutation testing, semantic changes (mutations) are systematically applied to a codebase. After each mutation, the test suite is run. If no tests fail, the mutation is said to have "survived". The proportions of surviving mutations (mutation survival rate) is tracked over many repetitions.

Take the following function and test case:

```ts
function add(a: number, b: number) {
  return a + b;
}

describe('add', () => {
  it('should add two numbers', () => {
    const result = add(0, 0);

    expect(result).toEqual(0);
  });
});
```

This test does well in many of the metrics we've discussed above, such as cyclomatic complexity, and code coverage. However, it's not a very effective test. To understand why, lets look at some mutations which might be applied during a code mutation test.

```ts
function add(a: number, b: number) {
  return a;
}

function add(a: number, b: number) {
  return b;
}

function add(a: number, b: number) {
  return 0;
}

function add(a: number, b: number) {
  return "0";
}
```

Our test would only catch the last mutation, giving it a mutation survival rate of 75%. Given we have identified this test as having a high mutation survival rate, we can re-write it to be more sensitive to regressions.

```ts
describe('add', () => {
  it('should add two numbers', () => {
    const result = add(30, 12);

    expect(result).toEqual(42);
  });
});
```

Our modified test now has a mutation survival rate of 0% - at least for the mutations we've chosen to apply.

Of course, this is a fairly contrived example, but these kind of issues can be far more subtle and hard to pick up on code review. If you've ever thought to yourself, that you're surprised that a bug wasn't caught by a test, then there's a good chance that code mutation testing could have helped in that situation.

The primary goals of mutation testing are as follows.

1. Identify parts of a codebase where mutations are not caught by the test suite
2. Identify redundant and low quality test cases which fail to catch mutations

Mutation testing allows us to quantify the sensitivity of our test suite, and individual test cases, to random code artifacts. Although not a perfect metric, it does at least allow us to track some kind of objective metric for sensitivity. We will discuss the difficulties of measuring test sensitivity and specificity further in the [following section](#specificity---redundant-failure-rate-rfr) on measuring specificity, as this is significantly less trivial to measure.

### Specificity - Redundant Failure Rate (RFR)

Earlier in this article, we discussed the concepts of [sensitivity and specificity](#sensitivity--specificity) as they relate to software testing. Unfortunately, these are fairly difficult to directly measure in an automated way. To understand why, lets remind ourselves of the variables we need to measure to calculate these metrics:

1. True positives - the number of times a test failed because it detected a bug
2. False positives - the number of times a test failed because and there was no bug
3. True negatives - the number of times a test passed because there was no bug
4. False negatives - the number of times a test passed and there was a bug

The first part of each is fairly trivial to detect - did the test pass or fail on a given run? However, determining if there was or wasn't a bug is much harder because we don't have a formal definition of what is and isn't a bug. As discussed in the [previous section](#sensitivity---mutation-survival-rate) on mutation testing, it is fairly trivial to deliberately introduce random bugs into our code and measure our test suites sensitivity to these artificial bugs. However, testing specificity is far less trivial. In fact, I'm not aware of any established techniques for measuring test specificity in the context of software testing.

However, I personally think that high test specificity is a really important feature of an effective and maintainable test suite. So take a step off the beaten path and look at how we _might_ try to measure it.

There are two ways we could approach measuring sensitivity. Firstly, we could do something similar to mutation testing, but preserve the business logic of our application. Most applications don't have a rigorous definition of their business logic, aside from the code itself. The obvious drawback here is that the code doesn't _just_ define the business logic. There's a whole bunch of implementation details, and it's a non-trivial problem to distinguish between implementation details and business logic. At some basic level, it's probably possible to automate random refactors (e.g. randomly renaming private methods). This may be something I look into in more detail down the road, but for the purposes of this article, lets focus on the second approach, empirical observation of existing data.

Fortunately, the data we want to observe here is all tracked in our git commit history. To estimate specificity from our git commits, we need to make some assumptions:

1. Developers don't update existing test cases unless they fail
2. It is much more common for a test case to be updated due to a false positive than due to a deliberate change to the behavior of a unit of code

The first assumption is generally true, unless we are refactoring or applying new linting rules. For the second assumption to be (mostly) true, we need our test cases to be narrow (i.e. they test only one thing). Note that even if this is the case, our assumptions are not foolproof. For example, a test case might change because the function signature changes, in a way which is immaterial to the test case. We don't have a good way to distinguish these changes, so we'll assume that these kinds of changes are uncommon.

Given we accept these assumptions as "good enough to be useful", we can derive the following equation:

```
           changed_cases(n, N) - cases_removed(n, N)
RFR(n,N) = -----------------------------------------
                cases(n) - cases_removed(n, N)
```

Where: `n` is a given commit, `N` is a commit after `n`, `changed_cases(n, N)` is the number of test cases which existed in commit `n` and were changed (modified or removed) by a commit between `n` and `N` (exclusive on the lower bound, inclusive on the upper bound), `cases_removed(n, N)` is the number of test cases which existed in `n` and were removed by a commit between `n` and `N` (exclusive on the lower bound, inclusive on the upper bound), and `cases(n)` is the total number of test cases in commit `n`.

In layman's terms, `RFR` is the proportion of test cases from a given commit, which ended up being modified in later commits. Given the assumptions we made above, `RFR` will vary with the inverse of our test suites specificity - i.e. a low `RFR` is good.

However, this metric is difficult to compare between ranges of commits because the number of lines changed on each commit may vary significantly. As such, the `RFR` should be adjusted as follows:

```
aRFR(n, N) = RFR(n, N) * lines(n) / lines_modified(n, N)
```

Where, `lines(n)` is the number of lines of code (excluding test cases) which existed in commit `n`, and `lines_modified(n, N)` is the number of lines of code (excluding test cases) which existed in commits `n`, and were changed (modified or removed) between `n` and `N` (exclusive on the lower bound, inclusive on the upper bound).

#### Limitations

The adjusted redundant failure rate (aRFR) attempts to adjust for variations between commits, but it does not take into account that parts of a large test suite may vary significantly in their underlying RFR.

Another limitation, is that it does not account for "refactor commits", in which test cases were refactored for a reason other than to change their semantic meaning. This is problematic, because it is not possible to simply "skip" a commit for this analysis, rendering this metric ineffective when analyzing commits on either side of a test refactor. It is worth noting that this is not only relevant to intentional refactors, but could also be due to changes in linter configurations.

At a basic level, it might be possible to adjust for purely syntactic linting changes by comparing the abstract syntax tree of the test case, rather than comparing the raw bytes. Going further than this, a good metric for syntactic similarity might allow us to adjust our equations to take this into account in a meaningful and useful way.

#### Is RFR a Useful Metric?

In short, I don't know!

To the best of my knowledge, redundant failure rate (RFR) is a novel concept and I'm not aware of any examples of it being measured in production applications.

That being said, I am currently working on a tool to automatically measure this in JavaScript/TypeScript applications with a Jest(like) test suite. I intend to apply this to some large production code-bases and intend to share the high level findings in a follow-up article.

---

## Conclusion

Although the "right" way to write unit tests varies significantly depending on the nature of the project and the team, this article has presented some key principles underpinning unit testing and software development more generally. My hope is that this article will help readers to improve the quality and usefulness of the tests they write, and encourage more thought and attention to be dedicated to the design of test suites.

---

## Further Reading

- [What is code coverage](https://www.atlassian.com/continuous-delivery/software-testing/code-coverage)
- [Test Driven Development: That's Not What We Meant](https://www.youtube.com/watch?v=yuEbZYKgZas)

## References

1. [Klasky, H.B. (2003) A study of software metrics](https://citeseerx.ist.psu.edu/pdf/05ffd5c4441efca1ebf3f4c587c5acbeb4195ea5)
2. [Fenton, N.E. and Neil, M (2000) Software Metrics: Roadmap](https://citeseerx.ist.psu.edu/pdf/055efdf2d425de330551779a1af5438509f41506)
3. [Williams, T.W. et al. (2001) Code coverage, what does it mean in terms of quality?](https://ieeexplore.ieee.org/abstract/document/902502)
4. [Goodhart, Charles (1975) Problems of Monetary Management: The U.K. Experience](https://link.springer.com/chapter/10.1007/978-1-349-17295-5_4)
5. [Pittet, S. What is code coverage?](https://www.atlassian.com/continuous-delivery/software-testing/code-coverage)
6. [IBM (2021) Cyclomatic Complexity](https://www.ibm.com/docs/en/raa/6.1?topic=metrics-cyclomatic-complexity)
7. [ECMAScript® 2022 language specification](https://www.ecma-international.org/publications-and-standards/standards/ecma-262/)
8. [Smith, P. (2020) Testing the V8 JavaScript Engine](https://medium.com/compilers/testing-the-v8-javascript-engine-cbda7d9272e6)
9. [Offutt, A.J. and Untch, R.H. (2001) Mutation 2000: Uniting the Orthogonal](https://cs.gmu.edu/~offutt/rsrch/papers/mut00.pdf)
