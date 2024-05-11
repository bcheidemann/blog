---
title: Tailwind Turns the Web into the Linux Kernel
categories:
  - CSS
  - Tailwind
  - Web Platform
articleId: 6
---

An exploration of the implications of Tailwind CSS's widespread adoption when considered in the wider context of software development history. The article is based on a conversation with two former colleagues and friends, [Luke Sterry](https://lukes-code.vercel.app/) and [Chris Jenkins](https://jenscript.com/).

---

## Acknowledgements

This article wouldn't be possible without the input of Luke Sterry and Chris Jenkins, who helped develop, refine, and challenge the ideas presented in it. Please check out their websites at [lukes-code.vercel.app](https://lukes-code.vercel.app/) and [jenscript.com](https://jenscript.com/). Although our views on Tailwind and other technologies may differ at times, they are both knowledgeable and talented developers, who's views I hold in the highest regard.

## Introduction

It’s no secret that the rise of Tailwind has sparked controversy and debate amongst the web development community. Recently, I discussed an article by Daniel Nagy, titled [Are Inline Styles Faster than CSS?](https://danielnagy.me/posts/Post_tsr8q6sx37pl), with two esteemed former colleagues of mine, [Luke Sterry](https://lukes-code.vercel.app/) and [Chris Jenkins](https://jenscript.com/). Although the article by Daniel Nagy doesn’t cover Tailwind, it sparked the question of whether Tailwind would perform better than [BEM](https://getbem.com/) on his benchmark.

This article is not about that though (I may follow up with some benchmarks in future). This article covers the lively debate which inevitably ensued when the topic of Tailwind was raised in conversation between a Tailwind evangelists and a Tailwind sceptics. Specific, we'll be discussing the historical perspective, and related conserns, which were raised in my conversation with Luke and Chris.

For the avoidance of doubt, this article does not seek to persuade the reader one way or the other on the use of Tailwind CSS. It represents only a narrow slice of the trade offs involved when making such technical decisions, and other factors should be considered when deciding whether Tailwind CSS is right for your team.

With that in mind, this article presentas a macro-view of the state of CSS, and the possible implications of widespread Tailwind adoption for the web development community.

## Hungarian Notation, WinUI & The Linux Kernel

*Please bear with me... we'll get back to Tailwind soon!*

Throughout the history of programming, we've seen many conventions come and go. Variable naming is one such example, where we've seen conventions such as hungarian notation, and the widespread use of accepted abbreviations. These conventions were largely justified due to the limitations around variable length in many ancient programming languages. These restrictions no longer apply, but the conventions remain in codebases which have stood the test of time.

One such codebase is the Linux Kernel. The Linux Kernal codebase makes prolific use of abbreviations, many of which are a practical necessity. Others, seem to exist mostly to save keystrokes. We don't have to look deep into the kernel code to see some examples of this.

```c
static inline bool rb_sched_core_less(struct rb_node *a, const struct rb_node *b)
{
	return __sched_core_less(__node_2_sc(a), __node_2_sc(b));
}
```

This code is taken directly from the [`torvalds/linux`](https://github.com/torvalds/linux) repository. Specifically, you can find it in [`/kernel/sched/core.c`](https://github.com/torvalds/linux/blob/cf87f46fd34d6c19283d9625a7822f20d90b64a4/kernel/sched/core.c#L221-L224). It may be surprising to see how simple this code. And indeed, this simplicity is common to a majority of code in the Linux Kernel. Often, the Kernel is viewed as a kind of black box which is out of reach for mere mortals. This is not true of the Kernel, or any other open source codebase. After all, complex code rarely stands the test of time, since it is inaccessible to new contributors.

> Often, the Kernel is viewed as a kind of black box which is out of reach for mere mortals

That said, despite it's simplicity, I would argue this code is completely inscrutible to developers who are new to the kernel. This is largely due to the use of abbreviations. Let's say we want to make sense of this code, not just parse its structure. We'll need to resolve a few abbreviations:

- `rb`
- `sched`
- `sc`

These are not easy to Google. Googling the term "rb" returns results for Football teams, F1, and a colleage I've never heard of. The first page of Google does not include any results for red-black trees, which are what the "rb" in "rb_sched_core_less" is referring to.

If you're an experienced kernel developer, you may be of the persuasion that developers who don't even know about red-black trees may not be the kinds of developers which are suited for contributing to the Linux Kernel. Maybe that's true, or maybe not. But, I certainly find it hard to believe that the Linux Kernel would actively suffer from expanding a few of these acronyms to something more readable.

The Linux Kernel is not the only codebase plagued by inscrutible accronyms. For instance, the [WinUI documentation](https://learn.microsoft.com/en-us/windows/apps/develop/ui-input/retrieve-hwnd) makes frequent references to acronyms like `HWND` and `WinRT`, which are similarly opaque to new developers.

Whether or not you believe the acronyms covered above are justified in their respective contexts, I wonder how you would answer if I asked you whether web development would benefit from similarly widespread use of acronyms?

## Tailwind CSS

*Phew! That wasn't too bad... let's get back to Tailwind!*

So far, this article has discussed very little about Tailwind CSS and web development. But I think we (web developers) can learn something from the examples discussed above. After all, Tailwind is fast becoming ubiquitous in webdev. And, unlike the Linux Kernel, the webdev community has historically prided itself on being accessible to beginners. For many of us, even those who no longer work with the web platform, this was our entrypoint to programming.

> the webdev community has historically prided itself on being accessible to beginners

What happens when Tailwind becomes a de-facto standard, an accepted alternative to CSS. When I discussed this with Luke, he rightly made the point that Tailwind calsses are quite guessable when you already know CSS. For example, he points out that it's pretty obvious that `p-1` stands for "padding 1".

This is very true, particularly as there's not many common CSS properties which start with "P". Most of the ones that do are some variation on "padding-[direction]". Although I agree that many of Tailwinds properties are very guessable *if* you already know CSS, I hypothesised that new developers will find Tailwind much less intuitive than vanilla CSS.

## Testing the Hypothesis

To test my hypotheses, I asked my partner, a simple question.

> Given a webpage with a box in it, if that box has the style p-1 applied to it, how would that influence the way in which it’s displayed by the browser?

For context, my partner is not a developer, but she does know the basics of HTML and CSS. After a thoughtful pause, her response was:

> I don’t know

I then asked her a very similar question. Taking care to preserve the exact wording, I substitued the term “p-1” for “padding 4 pixels”. Without hesitation, she responded:

> Well, it would add 4 pixels of space around the box

Of course, this isn’t fully correct; as the astute reader will no doubt have noticed, she’s describing the margin CSS property. None the less, I believe this simple example shows how much more accessible CSS is than short-hand Tailwind classes.

And don't forget, this example was chosen by a Tailwind evangelist to demonstrate how intuitive Tailwind CSS is.

## Conclusion

If historic conventions for variable names have taught us anything, it’s that codebases which prioritise saving keystrokes at the cost of clarity also tend to be those which are most widely viewed as black boxes, inaccessible to all but the upper echelons of software engineering.

Should the web development community prioritise convenience for the current generation of developers, over the accessibility of the web platform to the next generation? Or, in the words of Professor Scott Galloway, ["Do we love our children?"](https://www.youtube.com/watch?v=qEJ4hkpQW8E).
