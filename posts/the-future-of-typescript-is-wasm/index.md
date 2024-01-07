---
title: The Future of TypeScript is WASM
categories:
  - TypeScript
  - WebAssembly
  - Hermes
articleId: 5
---

In this article, we take an in depth look at static Hermes and consider why a future where TypeScript compiles to WebAssembly, not JavaScript, might not be as unlikely as it sounds.

---

## Introduction

Since it's creation in 2012 [\[1\]](#references), TypeScript has become one of the most successful programming languages ever [\[2\]](#references). By offering static analysis and developer tooling like LSP, TypeScript has helped scale JavaScript codebases to millions of lines of code, and teams of hundreds of developers [\[3\]\[4\]](#references). This is something which arguably would have been near impossible otherwise [\[5\]](#references).

TypeScript's superpower - the reason for it's success - has always been that it nothing more than JavaScript with types. In fact, it is a strict superset of JavaScript [\[6\]](#references), and compilation of TypeScript amounts to the removal of type annotations (with only a handful of exceptions). So with JavaScript being the defacto language of the web [\[7\]](#references), why would you ever consider compiling TypeScript to anything else?

## About me

I think it's worth covering a little about myself before we get started. I hope that this will help you, the reader, to understand how my own perspectives color my view of the topics we are about to discuss, and to make my personal biases explicitly clear.

My journey in software development started 18 years ago, when I learned to build 2D games with Flash. At the time, I was struggling in school, and programming helped me find the passion for learning that I was unable to find in the pages upon pages of long addition my teachers had us practice day in and day out. I have been fortunate to retain my passion for programming and learning to this day, and have since had the chance to dabble with many programming languages, frameworks and tools. I do not profess to be an expert in most of these, but I have loved learning them, and the experience of doing so has helped me to broaden my view of software development immensely. It's also taught me that **some tools are better for some jobs than others**, which is an idea somewhat salient to this article.

When it comes to my commercial experience, I have primarily worked in small teams; start-ups and scale-ups (though I've also worked at some larger companies). Because of this, I've had the pleasure (and occasionally the pain) of working on several bleeding-edge projects. These projects helped me to appreciate the value of the web as a technology beyond the browser (think Electron or Tauri), as they never would have been possible otherwise. They also helped me to appreciate the need to make the web faster, more capable, and more diverse. If we want to empower the next 20 years of tech startups to innovate at the same rate as the last 20 years of tech startups, then this isn't optional.

## Background

To avoid cluttering the rest of the article, we'll go over some relevant terminology up front. I recommend [skipping this section](#in-defense-of-webassembly) and coming back to it if you get to a part of the article that you don't understand. Of course, you're more than welcome to read it if you like (I probably would!), just don't expect it to be engaging, or have any semblance of narrative structure.

### JavaScript Runtimes

A JavaScript runtime is an application which runs JavaScript code. JavaScript runtimes are typically shipped as a component of browsers, but also exist independently, as server-side runtimes like Node.js, Deno, or Bun. These consist of several components, one of which is the engine. [\[8\]](#references)

### JavaScript Engines

You may have heard of some of the more popular JavaScript engines like v8 (used in Chromium based browsers and Node.js) or JavaScript Core (used in Safari and Bun). JavaScript engines are responsible for executing JavaScript code [\[8\]\[9\]](#references). To do this, they need to parse raw JavaScript code, and then execute the parsed code, managing system resources appropriately. Modern JavaScript engines have become quite complex in an effort to squeeze as much performance out of your code as possible, and the details of how they work are beyond the scope of this article.

In isolation, the engine itself is not very useful. This is because it does not provide system access [\[9\]](#references), and as a result, it cannot load modules from disk, interact with system hardware like the GPU, process user input, or output anything. Even things as basic as `console.log` are not implemented in the engine because they need to interact with the environment in which the code is running.

Even in cases where the engine could provide the implementation for an API, it often doesn't. In some cases this is because the API in question is not part of the [ECMAScript® Language Specification](https://html.spec.whatwg.org) (perhaps it's part of the [WHATWG HTML Living Standard](https://html.spec.whatwg.org) or unique to the runtime). Examples of this include the likes of `atob` and `btoa`, which do not require system access, but are defined in the HTML standard [\[10\]](#references) rather than the ECMAScript language specification.

### Native Code

If you've ever tried to run something like `eval.toString()`, you will have seen:

```js
function eval() { [native code] }
```

The presence of `[native code]` here tells us this is not a "normal" JavaScript function. Instead, the function is implemented at the native layer, either by the runtime or the engine. The `eval` function is implemented by the JavaScript engine, but other functions (e.g. `console.log`) are implemented by the runtime.

Functions tend to be implemented natively when they cannot be implemented in pure JavaScript (e.g. `console.log`), or superior performance is required (e.g. `atob` and `btoa`). Generally, where the function is part of the [ECMAScript® Language Specification](https://html.spec.whatwg.org), they will be implemented by the engine. Other functions which require a native implementation will be implemented by the runtime.

### Runtime APIs

As discussed in [Native Code](#native-code), runtimes may extend the functionality of the engine. This can be done by providing native implementations, as already discussed, or by injecting custom JavaScript code into the engine. Whether or not a runtime chooses to provide an API natively or in JavaScript (or a hybrid of both) depends on several factors, including performance and requirements on system access.

### Type Systems (Sound Types)

A type system is a set of rules about a typed program which offers certain guarantees about how the program will behave at runtime. For example, in a typed language, the type system may guarantee that if the program follows a set of defined rules, then a value annotated as being of one type will never be of a different type at any point during the execution of the program. The rules of the type system can be validated statically. [\[11\]](#references)

Different type systems may impose different rules in return for different guarantees. Most people will be familiar with type checkers which enforce rules like "the type of function argument must match the type of the function parameter". This rule will allow the following program:

```ts
function doSomething(param: string) {}

doSomething("Hello World!");
```

But will disallow the following program:

```ts
function doSomething(param: string) {}

doSomething(42);
//          ^ ERROR: `42` is of type number, expected `string`
```

However, other type systems may imposes additional rules in return for further guarantees. For instance, the Rust type system is able to provide guarantees about variable access which other languages cannot. The following program is disallowed in Rust, while being allowed in other languages:

```rs
struct Address {
  street: String,
  city: String,
  country: String,
}

struct User<'a> {
  address: &'a Address,
}

impl<'a> User<'a> {
  fn new(address: &'a Address) -> Self {
    Self {
      address,
    }
  }
}

fn main() {
  let mut address = Address {
    street: String::from("1234"),
    city: String::from("Seattle"),
    country: String::from("USA"),
  };

  let user = User::new(&address);
  //                   -------- `address.street` is borrowed here

  address.street = String::from("4321");
  // ^^^^^^^^^^^ `address.street` is assigned to here but it was already borrowed

  println!("User address: {}, {}, {}", user.address.street, user.address.city, user.address.country);
  //                                   ------------------- borrow later used here
}
```

In this example, we use the `&` symbol to indicate to the rust type system that we are passing an immutable reference to the `address` variable into the `User::new` function. Because we have provided this annotation, the Rust type system is able to guarantee that the `address` will not be mutated while it is still being referenced ("borrowed") by the `user` variable. When we try to do so, the Rust type system will prevent the program from being compiled.

A type system is considered to be sound when the guarantees made are provably correct - i.e. the type system never lies. However, all non-trivial languages have properties which either cannot be determined with static analysis, or are omitted from static analysis for some reason. For instance, many statically typed languages will allow arrays to be indexed out of bounds because it is impossible to determine what the length of the array will be at runtime in all cases. However, it is understood by the programmer that indexing out of bounds will result in a runtime exception. Where the type system is provably correct, within the constraints of known exceptions, the type system is considered sound. [\[11\]](#references)

An example of an un-sound type system is TypeScript. In TypeScript, runtime values may differ from the annotated type, without causing a defined runtime exception:

```ts
function sayHello(name: string) {
  console.log(`Hello, ${name}!`);
}

const names: string = ["Bob"];
const name: string = names[2];

sayHello(name);
// Hello, undefined!
```

## In defense of WebAssembly

WebAssembly (abbreviated WASM) is a binary instruction format [\[12\]](#references), originally intended to be used as a compilation target for native languages like C, C++ and Rust to allow them to run in the browser. As JavaScript did with Node, WASM has escaped the browser and is now used as a generic compilation target to allow any compiled language to run on any server, browser or client.

WASM, in my opinion, does not get the kind of attention it deserves. Many people in the web development space are either blissfully unaware of it's existence, or see it as being useful only for certain edge cases. This is understandable. It didn't arrive with a big bang, and many features which would have garnered increased attention were missing. The following issues are, I think, responsible for WASM's initial luke warm reception:

1. WASM Binaries tended to be quite bloated, particularly when shipping GC languages
2. WASM requires the use of JS glue code to interact with the browser, and this has resulted in the misconception that it is slow to update the DOM
3. WASM didn't ship with support for threading, limiting possible performance gains
4. To use WASM, developers needed to learn "un-web-like" languages (e.g. C, C++ and Rust)
5. WASM didn't initially enjoy strong cross-browser support

These issues are largely being addressed. Proposals like [Garbage collection](https://github.com/WebAssembly/gc) and [Threads and atomics](https://github.com/WebAssembly/threads/blob/master/proposals/threads/Overview.md) have been standardized and are well on their way to general availability across browsers [\[13\]](#references). Additionally, other languages have started to emerge which make WASM more accessible to web developers - [AssemblyScript](https://www.assemblyscript.org/) being one such example.

With the progress that is being made, I believe strongly that WASM has the potential to expand beyond it's current niche and work its way into the average web developers arsenal (whether they are aware of it or not). Furthermore, I think the growth of WASM is essential to enable a wider set of applications to be built on the web platform.

My stance on WASM is likely more hard-line than most. To me, the future of the web should be WASM, and I would love nothing more than to see it replace JavaScript as the defacto way to run applications in the web. Why? Don't get me wrong, I love JavaScript (well... TypeScript), and I will continue to use it where it makes sense. But there are many project for which JavaScript is being used simply because there is no real alternative, and I'm sure there are an increasing number of cases where commercially viable projects aren't being worked on (or where startups fail) because JavaScript is not the right tool to enable them to be successful. I would love for the web to become a place where developers and businesses alike feel confident reaching for the right tool, not the default tool. This is something WASM makes possible.

## AssemblyScript (a brief aside)

[AssemblyScript](https://www.assemblyscript.org/) is an amazing exploration into making WebAssembly accessible to average web developers. It allows developers to write code with TypeScript syntax, which compiles to WASM. However, although it implements many JavaScript APIs in it's standard library [\[14\]](#references), it is not TypeScript. It does not adhere the [ECMAScript® Language Specification](https://html.spec.whatwg.org), nor can "normal" TypeScript code be compiled using AssemblyScript. This means, when we write AssemblyScript, we throw away the majority of the NPM ecosystem.

## Hermes

Hermes is a JavaScript engine developed by Meta for use with React Native. It differs from most established engines, in that hermes ingests bytecode, not raw JavaScript. Although other engines also execute bytecode behind the scenes, they must perform the compilation to bytecode at runtime. This is known as just in time (JIT) compilation and is *typically* necessary in the context of the browser, since they need to have the ability to execute JavaScript source code. Note that this is no longer the case with the introduction of Web Assembly (WASM), and we'll get into the implications of that shortly.

The ability to ingest bytecode unlocks several advantages for Hermes. Firstly, it allows for much faster startup times [\[15\]](#references). This is actually the motivation for Hermes, since the startup times of React Native apps were previously limited by the time it took to parse and compile JavaScript source code to bytecode that the engines VM could execute. Another advantage of this approach is that the hermes compiler can perform optimizations ahead of time. In comparison, a typical runtime will only optimize hot code paths, due to the cost of performing optimizations at runtime.

## Static Hermes

Static Hermes is a version of the Hermes compiler designed to produce highly optimized, native code, utilizing type information from TypeScript. For this to be possible, code run by static Hermes behaves slightly differently at runtime than it would in a conventional runtime like v8. For instance, accessing an array index out of bounds doesn't return `undefined`, instead this is undefined behavior, as it would be in C.

Although this means that *some* TypeScript code will be incompatible with static hermes, it is still executing TypeScript code, as it aims to be compatible with the [ECMAScript® Language Specification](https://html.spec.whatwg.org). It also does not require the use of non-standard types, like [AssemblyScript](https://www.assemblyscript.org/) does. Therefore, a large portion of the TypeScript ecosystem will be compatible with static hermes, or can easily be made compatible. Furthermore, as static Hermes interoperates with regular hermes, it would technically be possible to use all of the existing JavaScript ecosystem with the Hermes engine, albeit with a performance cost when we need to fallback to non-native code.

Static Hermes is still in its infancy at the time of writing, but it offers some exciting possibilities. For instance, it allows TypeScript to have comparable performance to languages like Go, while still maintaining compatibility with most of the existing ecosystem of libraries. Another exciting possibility is native system access, directly from TypeScript. Because static Hermes compiles to native code, it can make system calls and do all the other things native code can do, that JavaScript usually can't. One example of this is that both static and dynamic linking are possible, enabling extremely performant FFI.

## Compiling TypeScript to WASM

We mentioned previously that the web, until recently, did not have a bytecode interpreter. Web assembly (WASM) addresses this issue, providing a web-compatible compilation target for many languages which previously could not be easily run in the web. It may seem counter-intuitive at first, but there is no reason why static Hermes could not target WASM.

This seems like a strange idea at first. After all, TypeScript already runs in the browser, so surely there's no need to compile it to WASM. I think there's two compelling use cases for this:

- Performant interop with other languages
- Leveraging type information to optimize performance critical applications

These might seem like niche cases, and I don't disagree. In fact, I don't think this will ever become the defacto way to run TypeScript (at least not for a very long time). However, speaking from experience, there are cases where performance on the web matters... a lot. In these cases, it would be nice for web developers to be able to continue to work with TypeScript as a language, while gaining the benefits WASM provides. With the direction that WASM and static Hermes are taking, it seems to me that they are all but destined to collide.

## Conclusion

In this article, we discussed static Hermes, an experimental approach to compiling TypeScript natively. We also discussed why, in certain cases, it might make sense to use WASM over JavaScript as a compilation target for TypeScript. My hope is that I have been able to show two things in this article:

1. Compiling TypeScript to WASM makes sense in some use cases
2. We are not that far away from this becoming possible

I look forward to the possibility of compiling TypeScript to WASM, and to the continued evolution of WASM and the web platform.

## References

1. [Microsoft - DevBlogs - Announcing TypeScript 1.0 (2014)](https://devblogs.microsoft.com/typescript/announcing-typescript-1-0/)
2. [Stack Overflow - 2023 Developer Survey - Programming, scripting, and markup languages (2023)](https://survey.stackoverflow.co/2023/#most-popular-technologies-language-prof)
3. [GitHub - Most Popular TypeScript Repositories](https://github.com/search?l=TypeScript&o=desc&q=typescript&s=stars&type=Repositories&utf8=%E2%9C%93)
4. [GitHub - VSCode Source Code](https://github.com/microsoft/vscode)
5. [GitHub - Most Popular JavaScript Repositories](https://github.com/search?l=JavaScript&o=desc&q=javascript&s=stars&type=Repositories&utf8=%E2%9C%93)
6. [Wikipedia - TypeScript](https://en.wikipedia.org/wiki/TypeScript#:~:text=Because%20TypeScript%20is%20a%20superset,type%2Dcheck%20for%20safety%20reasons.)
7. [Stack Overflow - 12 Best Languages for Web Development in 2023 (2023)](https://www.browserstack.com/guide/best-language-for-web-development#toc2)
8. [Introduction to JS Engines and Runtimes](https://algodaily.com/lessons/introduction-to-js-engines-and-runtimes)
9. [A Guide to JavaScript Engines for Idiots](https://web.archive.org/web/20181208123231/http://developer.telerik.com/featured/a-guide-to-javascript-engines-for-idiots/)
10. [WHATWG HTML Living Standard - 8.3 Base64 utility methods](https://html.spec.whatwg.org/#atob)
11. [Brown University: Programming and Programming Languages by Shriram Krishnamurthi - The Central Theorem: Type Soundness](https://papl.cs.brown.edu/2014/safety-soundness.html)
12. [WebAssembly.org](https://webassembly.org/)
13. [WebAssembly Feature Extensions](https://webassembly.org/features/)
14. [AssemblyScript - From a JavaScript perspective](https://www.assemblyscript.org/introduction.html#from-a-javascript-perspective)
15. [Hermes Performance on iOS](https://www.callstack.com/blog/hermes-performance-on-ios)
