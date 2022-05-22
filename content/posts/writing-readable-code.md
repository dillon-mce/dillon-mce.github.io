---
title: "Writing Short Code Isn't The Point"
date: 2022-03-13
tags: ["Swift"]
---

I don't know about you, but I have spent a lot of time trying to write code in the shortest, most elegant way possible. I have put a lot of thought and time and effort into it over the years. And I see it a lot in other people's code as well. I have seen it from both junior and senior developers. I have seen it in new code bases and mature code bases, both large and small. I think to some extent it comes from the code challenge and technical interview culture where accomplishing the goal with fewer characters is seen as better. Maybe it comes from our desire to prove ourselves and line count is a metric that is simply easy to gather. Maybe we just like the challenge, or we see it as a status symbol.

Whatever the reasons, I think we should stop it. We should strive to write understandable code instead. In this article we will take a look at how it is saves time, it is easier to share with other people, it is easier to fix and it is easier to modify. And then I'll share a few simple tactics for writing more readable code.

## Why
Almost all the reasons I could come up with boil down to the fact that it saves time to write longer, more understandable code. It saves you time. It saves time for everyone you work with. It saves time for your users. And time is our most valuable resource. It is one of the few things we cannot get more of, so let's try to make the most of it.

You read code more than you write it. Be mindful during almost any work day and you'll find that this is true. You spend time looking up existing functionality to answer your colleague's question. You spend a bunch of time trying to find the right spot to fix a bug, and then write the one line to fix it. You spend time planning where and how to add a new feature, and then some time writing it. But writing it isn't just a straight shot where you start at the first line and move on to each subsequent one. You reference docs or example code. You check against other parts of the code base to find reusable elements, and to make sure you're not breaking something. You read back over the code you've just written to make sure it does what you think it does. And so on.

So if you spend so much more time reading code than writing it, it makes sense to optimize for readability. Make it easier to reason about. Easier to load into your RAM when you come back to it in six months. Not only will this save you time, but it will also scale better. Other people have to understand your code. You have to understand other people's code. Would you rather spend six hours trying to tease out what a dense block of code actually does or be able to scan it, get the gist, and move on to adding your feature? Or even worse, maybe you didn't take the time to fully understand that dense block of code and you introduce a hard-to-find bug that will be the bane of your existence for the next year. (Not that this has ever happened to me 😁)

The more readable code is, the easier it is to debug. You are less likely to introduce bugs in the first place because they aren't obscured by the complexity. But with the ones that do get through, they are easier to track down because the flow of the logic is obvious. There is nowhere for them to hide.

Finally, the only constant is change. Your code base will change. The developers working on it will change. You will need to onboard new developers when they join the team and you will need to take ownership over code you didn't write when people leave. You will need to change functionality over time. Having readable code makes all of those things easier.

On top of that the language/libraries you are using will probably change over time and there is a high correlation between short/clever code and the most sugary of syntax. That cutting-edge syntax is way more likely to change over time than the fundamentals. This isn't a huge problem, but it is one more way that you are likely to waste some time if you insist on writing the shortest code.

## Do This Instead
So how do we write this mythical "readable" code? There is no single thing you can do that will magically make your code readable. But there are many things you can do which will make it _more_ readable. It starts with your mindset. Right now, when you sit down to write some code you may be in the habit of asking yourself something like this "how will I write this in the best/most efficient/most elegant/shortest way possible?" I know that was my mindset for a long time. But now I would advocate that we should instead ask ourselves "how can I write this in such a way that it will make the logic obvious and understandable?" Simply re-framing things in this way is a huge step in the right direction, but I've also got a few practical tips that I try to keep in mind along the way:

**Name variables for their role, not their type.** Especially in strongly-typed languages like Swift, it isn't necessary to add type information to a variable's name. That information already lives somewhere. But the compiler can't really know what that variable is _for_. Why is it there? What does it contribute to this piece of code? Give future readers a huge clue by naming that variable in a way that communicates this information.

```swift
// don't do this
let redColor = UIColor.red
let array = [24, 31, 27, 45, 37]

// do this instead
let error = UIColor.red
let userAges = [24, 31, 27, 45, 37]
```

**Do one thing at a time.** Break code up into discrete pieces of logic. I often come across code where someone is trying to save time or improve performance by mashing multiple pieces of logic into one loop or something. There may be places where that becomes necessary for performance reasons, but it is very difficult to estimate performance of software ahead of time and in my experience developers are pretty bad at estimating the actual causes of performance bugs. It is much better to write it the clear and slow way first, and then test to look for places to improve the performance if necessary. Most of the time the "slow" way will be fast enough.

```swift
// don't do this
var ages: [Int] = []
var names: [String] = []

for user in users {
    ages.append(user.age)
	names.append(user.name)
}

// do this instead
let ages = users.map { user in user.age }
let names = users.map { user in user.name }
```

**Prefer clarity over brevity.** That is just a fancy way of saying that it is more important to write clear code than short code. Use as longer names for functions if it helps. Name anonymous closer arguments. Name a variable even if it is only used once. Break down large functions into smaller parts. Etc.

```swift
// both versions assume this extension
extension String {
    func orIfEmpty(_ replacement: String) -> String { isEmpty ? replacement : self }
}

// don't do this, real solution I found online by googling "Fizz Buzz solutions"
(1...15).map { (($0 % 3 == 0 ? "Fizz" : "") + ($0 % 5 == 0 ? "Buzz" : "")).orIfEmpty("\($0)") }.forEach { print($0) }

// do this instead
// name the function so that it is clear what it does at the call site
func playFizzBuzz(from start: Int = 1, to end: Int) {
    playFizzBuzz(start...end)
}

// allow users to user whichever interface makes sense at call site
func playFizzBuzz(_ range: ClosedRange<Int>) {
    let answers = range.map { number in // name closure arguments
        singleFizzBuzz(for: number)
    }

    answers.forEach { answer in
        print(answer)
    }
}

// pull out discrete pieces of logic to separate functions
private func singleFizzBuzz(for number: Int) -> String {
	// use language-provided functions where you can
    var wordReplacement = number.isMultiple(of: 3) ? "Fizz" : ""
    wordReplacement += number.isMultiple(of: 5) ? "Buzz" : ""
    return wordReplacement.orIfEmpty("\(number)")
}

// maybe a bit overkill for fizz buzz, but it illustrates the principle

```

**Follow the conventions of the language and codebase.** If you're an iOS developer writing Swift like I am, a good start is to read through [the API design guidelines](https://www.swift.org/documentation/api-design-guidelines/) (where you'll notice some similarities with my advice here). If the code base you're working in has agreed-upon conventions, follow them! Even if it goes against your personal preference. Clarity is more important that your opinions. If you really care about something, get the team to agree upon a new convention and offer to migrate the existing code as much as you are able.

Finally, **always be refactoring.** If you don't know how, read [Martin Fowler's book on the topic](https://martinfowler.com/books/refactoring.html). (Even if you do know how, you should probably read it anyways.) Make a lot of small changes in the direction of cleaner/simpler/easier to understand code over time and it will add up. Any time you make changes to some part of the code stop and ask yourself this question: "What could I do in five minutes that would make it easier for the next person to understand?" Don't waste a ton of time on it, that is the opposite of what we're trying to do. But give it a couple minutes and you'll find something. Rename a variable. Break up the function. Add a signpost comment. Follow the boy scout rule. Always leave the code cleaner than you found it.

## Wrap Up
That's all I've got for today. Spend less time making your code short and clever and spend more time making it readable. Shift your mindset. Make lots of small changes to improve readability. They will compound into massive improvements over time.  And they will save you time, they will make your code easier to change, easier to debug and easier to communicate.

Let me know what you think. How do you go about making your code more readable? What are the cases where short and clever code is better?
