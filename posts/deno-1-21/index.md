---
title: Deno 1.21 – One Giant Leap for Testing
categories:
  - Testing
  - Deno
articleId: 2
---

The release of Deno 1.21 marks a huge step forward for testing. It includes features such as snapshot testing (which I worked on personally), a BDD style test runner, and mocking utilities. Let's do a deep dive into why these features are so important for Deno!

---

## Introduction

Since its version 1.0 release in 2020, Deno has challenged many people’s assumptions about what a JavaScript runtime can and should be. One example is its approach to testing. Deno asserts (pun intended) that testing is fundamental part of modern software development and should be treated as such. What this means, is that testing in Deno is a language feature – not a third-party library. This approach is heavily inspired by Rust, which is the language used to implement Deno, but it is new to the JavaScript ecosystem.

However, until the release of version 1.21, Deno has rarely strayed from the basics – offering a test runner and some basic assertions, courtesy of the standard library. The implementation of more powerful test features has historically been left to third party libraries.

This is problematic because, although Deno has attracted much attention over the last two years, the ecosystem is still quite immature, and few projects have made it to production. This is a bit of a chicken and egg problem; a lack of production projects means a lack of incentive to create and maintain libraries, and a lack of libraries can contribute to a reluctance to use Deno in production.

In my opinion, the most important question that Denos community and maintainers need to ask themselves is how Deno can break this cycle. One piece of the puzzle arrives in the form of high-profile announcements like [Netlify partnering with Deno to deliver edge functions](https://deno.com/blog/netlify-edge-functions-on-deno-deploy). When high profile companies like Netlify put their trust in Deno, this builds confidence in the ecosystem and drives more users to Deno.

But Deno hasn’t rested on its laurels. Version 1.21 tackles this problem from another angle. If the third-party ecosystem lacks important features, why not implement those features as in the standard library.

Denos standard library has not yet reached its first major version but has built a lot of trust with the community by offering a well thought out, and robust feature set. Importantly, if the feature you need is in the standard library, you can be confident that it’s here to stay and will be properly maintained.

So why is Deno 1.21 such a big deal? Put simply, it marks the release of three new standard library features which put Denos test capabilities on a par with Jest. If that doesn’t mean anything to you, Jest is a testing framework maintained by Facebook (now Meta) and is considered the gold standard in JavaScript testing.

Let’s take do a deeper dive into these features!

---

## Snapshot Testing

Snapshot testing is a relatively recent addition to the JavaScript ecosystem, being introduced to Jest with version 13 in 2016 (as per the changelog). Since then, it’s exploded in popularity, driven largely by the success of React (another Facebook project) and the unprecedented simplicity of writing and maintaining tests that is afforded by the feature.

It offers a solution to many common problems in software testing:

1. Complexity
2. Maintainability
3. Readability

To understand how, we need to touch on the basics of how traditional unit tests work. A good practice in unit testing is to follow the “arrange, act, assert” pattern. The “arrange” step involves setting out some input data, then this input is fed into a unit of code in the “act” step, and finally, assertions are made about the output in the “assert” step.

For example, if we have a function called “addOne”, we might write a which looks something like this:

```ts
// Arrange
const input = 1

// Act
const result = addOne(input)

// Assert
assertEquals(result, 2)
```

This test will fail if the result is of “addOne” is anything other than 2, alerting us to a problem in our code.

This works well with simple functions like “addOne”. But what happens when your expected output is, for example, the statically rendered HTML for a webpage? As a rule of thumb, a unit test will always be at least as many lines long as the number of things you want to assert about the output. This means that to fully cover our HTML output, our test could easily be hundreds of lines long, and we might require dozens of tests like this to test all the possible input cases.

Adding that much code to test a single unit of code is inadvisable. Longer tests are harder to read, harder to maintain, take longer to write, and cost more money. For this reason, many companies have historically elected not to test their frontend code. This has resulted in many bugs making it into production which could have been avoided by having concise, maintainable, and effective unit tests.

The philosophy behind snapshot testing is that writing code that reliably determines the correctness of some complex output is hard, but that developers are good at determining the correctness of the output by manual inspection.

As such, when testing complex outputs, it can be better for a developer to manually inspect the output the first time the test is run. This output is then saved to disk as a “snapshot”, and whenever the test is run in future, the output will be compared to the existing snapshot. Whenever the output differs from the snapshot, the test will fail, and the developer will be given a summary of what has changed – this is known as a “diff”.

This diff (developer slang for difference) is essential as it enables developers to identify more easily if a) the change is intentional and b) if the change is unexpected, what part of the code is likely to be causing it.

In the former case, the developer can then choose to update the snapshot and the updated snapshot will be used in future test runs. In the latter case, the developer can more easily identify the cause of the bug and fix it sooner.

Because the snapshot file is committed to version control, along with code changes, it is easy for a code reviewer to validate that the changes are correct. Changes to a snapshot can also help code reviewers to better understand code changes.

As of version 1.21 of Deno, snapshot testing is now available via the standard library and can be used like this:

```ts
// example_test.ts
import { assertSnapshot } from 'https://deno.land/std@0.138.0/testing/snapshot.ts'

Deno.test('isSnapshotMatch', async (t) => {
  // arrange
  const title = 'Article Title'
  const summary = 'This is a short summary of the article.'

  // act
  const html = renderArticleCard({
    title,
    summary
  })

  // assert
  await assertSnapshot(t, html)
})
```

```js
// __snapshots__/example_test.ts.snap
export const snapshot = {}

snapshot[`isSnapshotMatch 1`] = `
"<article>
  <h3>Article Title</h3>
  <p>This is a short summary of the article.</p>
</article>"
`
```

For more information on snapshot testing see the [Deno manual](https://deno.land/manual/testing/snapshot_testing).

---

## BDD Style Testing

Like me, you may not have been familiar with the term "BDD style testing" before reading it in the context of the Deno 1.21 release. However, it's something that the majority of JavaScript developers will be familiar with, due to the prevalence of BDD style test frameworks like Jest, Mocha and Jasmine (to mention a few).

BDD (Behaviour Driven Development) style testing focuses on grouping tests into “describe” blocks. Each “describe” block should focus on a single unit of code and typically contains many “it” blocks. Each “it” block makes an assertion about the code being tested. Note that this approach is distinct from the concept of behaviour driven development itself.

In case you are unfamiliar with the "describe/it" format, it would typically look something like the following:

```js
describe('addOne', () => {
  it('should add one to a positive integer', () => {
    // Arrange
    const input = 1

    // Act
    const result = addOne(input)

    // Assert
    assertEquals(result, 2)
  })

  it('should add one to a negative integer', () => {
    // Arrange
    const input = -1

    // Act
    const result = addOne(input)

    // Assert
    assertEquals(result, 0)
  })

  it('should fail when given a string as an input', () => {
    // Arrange
    const input = 'some string'

    // Act
    const action = () => addOne(input)

    // Assert
    assertThrows(action)
  })
})
```

This style allows developers to more clearly and easily organise their tests and is a common choice for unit tests and automation tests.

In addition, BDD style frameworks typically offer “hooks” to reduce the amount of duplicate code needed for setting up a test and cleaning up after a test has run. For example:

```js
describe('SingletonClass', () => {
  beforeEach(() => {
    SingletonClass.init()
  })

  afterEach(() => {
    SingletonClass.reset()
  })

  // --snip--
})
```

As of version 1.21 of Deno, a BDD style test runner is now available via the standard library. This is an important contribution as it is a style of test which many developers have come to expect due to its prevalence. To learn more about it, see the the [Deno manual](https://deno.land/manual/testing/behavior_driven_development).

---

## Mocking

Mocks spies and stubs - collectively mocking utilities - can enable developers to test units of code which would ordinarily be hard to test. Common use cases are in testing code which interacts with the filesystem or an API.

To take an extreme example, you may want to test a unit of code which deletes every file on the users computer. In this case, it would be undesireable to run this code against your actual filesystem. So how do you test it?

Hypothetically, you could run your code in some kind of contained environment like a virtual machine or a docker container. But this would be a lot of work. Instead, we could simply mock the operations which read from the filesystem and stub the operations which write to the filesystem.

Lets say our code looks like this:

```js
async function deleteEverything() {
  const files = await FileSystem.getListOfFiles()
  await Promise.all(files.map(async (file) => await FileSystem.delete(file)))
}
```

In this case, there is no output to assert in the traditional sense. Equally there is no input. Instead, the input and output come in the form of side effects. This makes a traditional unit test hard to write.

We have a few options here. Firstly, we could refactor our code to make it easier to test. For example, by using a more functional style and/or making use of the dependency injection pattern. Alternatively, we could skip writing a test all together and rely on manual inspection to determine the correctness of this unit of code.

Either of these options may be acceptable to you. But we do have a third option. Consider the following test:

```js
describe('deleteEverything', () => {
  let getListOfFilesStub
  let deleteSpy
  let deleteStub

  beforeAll('setup mocks', () => {
    getListOfFilesStub = stub(FileSystem, 'getListOfFiles', () => [
      '/first.file.txt',
      '/second.file.txt',
      '/third.file.txt'
    ])
    deleteSpy = spy()
    deleteStub = stub(FileSystem, 'delete', deleteSpy)
  })

  afterAll('restore mocks', () => {
    getListOfFilesStub.restore()
    deleteStub.restore()
  })

  it('should delete everything', async () => {
    // Act
    await deleteEverything()

    // Assert
    assertSpyCall(deleteSpy, 0, {
      args: ['/first.file.txt']
    })
    assertSpyCall(deleteSpy, 1, {
      args: ['/second.file.txt']
    })
    assertSpyCall(deleteSpy, 2, {
      args: ['/third.file.txt']
    })
    assertSpyCalls(deleteSpy, 3)
  })
})
```

The above test asserts that `FileSystem.delete` is called once for each of the files returned by `FileSystem.getListOfFiles`. It is written in the BDD style we [discussed above](#bdd-style-testing), using some hooks to setup and tidy up the mocks.

Mocking utilities were introduced in version 1.21 of Deno and are available via the standard library. For more on mocking, see the the [Deno manual](https://deno.land/manual/testing/mocking).

---
