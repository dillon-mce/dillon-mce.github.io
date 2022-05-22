---
title: "Fixing an Unfixable Bug"
date: 2022-03-06
tags: ["iOS", "Xcode"]
---

If you have been an iOS developer for any length of time, you have certainly come across bugs that are hard to understand and hard to track down. And if you have an app out in the world that real human beings use you have probably encountered a bug or crash that is happening for some subset of your users, but that you cannot for the life of you reproduce on your device. This is the story of one of those bugs. It was one where I went into it not having a clue what the problem was and where I fully expected to bang my head against the wall for a day without actually solving anything.

*If you don't already know, my day job is working on an app called [Hallow](https://hallow.app). It is a prayer and meditation app (in the vein of Calm or Headspace) with Christian-themed content, but this is a crash that you could run into in practically any iOS app. And a lot of the process is relevant to finding/fixing bugs in any environment, not just iOS apps.*

## The Report
A few weeks ago I was looking through our bug reports (we use [bugsnag](https://www.bugsnag.com)) and I noticed a crash in our app. It was reported as having been introduce in the latest version of the app. It was listed as "unhandled". And every instance had a very short stacktrace which ended in a view that very few users ever see. The error message was `EXC_BAD_ACCESS`, with this detail: `Attempted to dereference garbage pointer 0x10.` The address of the garbage pointer was different in various instances (though many of them were `0x10` and `0x20`) but everything else was the same.

## My Thoughts
Here are some of my initial thoughts on this crash, as I remember them now and from looking back through Slack messages to my coworkers.
- the view in the stacktrace was probably irrelevant. I'm not sure why it is showing up consistently, but there were way more of these crashes than users who ever get to that view. Probably a red herring
- the crash was grouped in the reports by stacktrace, but this was a memory bug, having to do with a bad pointer. That meant a couple of things. There were probably other crashes with different stacktraces but with the same root cause. This bug may not have been introduced in the latest version, but bugsnag may just be unable to link the crashes in previous versions with the same cause
- `EXC_BAD_ACCESS` is not a particularly helpful error message, but knowing that we are trying to deference a garbage pointer gives me something to go off of. Instruments has a tool for profiling zombies which is helpful for tracking down exactly that sort bug.

## Discovery
I confirmed the first two thoughts by doing a cursory glance of the other crashes in our logs. There were several other ones with the same `Attempted to dereference garbage pointer 0x10.` but with different stacktraces in the same version of the app. I took that to mean that the view bugsnag claimed was the issue was not actually the issue. There were also similar numbers of crashes in the last several versions of the app with that same message, which I took to mean that the version of the app that bugsnag claimed introduced it actually did not. That helped me broaden my scope as I looked through recent code changes. In retrospect, I should have dug back through versions and tried to find the version where this bug actually was introduced, but I didn't think of it at the time.

I also spent twenty minutes or so digging through the code looking into things that I thought _might_ be the issue. In the grand scheme of things, this was not a particularly good use of time because I didn't really have enough information to go off of and I didn't find anything helpful. I should have gone straight to the zombie profiler.

## Profiling The App
I knew Instruments has a tool called "Zombies" which is used to track down over-released objects (which is what leads to garbage pointers), but I had only used it once several years ago, so I did some googling and found [this article about how to use it](https://pratheeshbennet.medium.com/xcode-instruments-zombies-8b262b1ae9d8).  I skimmed it and felt equipped enough to dive in and see what I could find.

Fortunately for me, it was incredibly easy. I started the app in Instruments, tapped on a piece of content and it immediately crashed and told me I had over-released an object. I clicked into the flag, as described in that article, and found that the object was a view controller representing a certain block of content in the app. I tried profiling several other times and found the same object every time, so I felt pretty confident that I knew where to dig in on the code.

## The Culprit?
The specifics of the bug in the case are not terribly important. I am trying to document the process I went through to find it and fix it more than trying to document this specific fix, but for those of you who are curious, it basically boiled down to this:
- We have a collection view cell that we use to wrap views.
- We were sticking the aforementioned view controller's view into this cell in `collectionView(_:cellForItemAt:)`
- Nothing was holding on to that view controller, just a local reference. It was never added as a child view controller to any other view controller.

The fix? Our first attempt was to just keep a reference to that view controller in the collection view cell, and remove it on reuse.

## Validation
I re-ran the zombie profiler after implementing this fix and found that I could navigate all around the app without it crashing, which I took to mean that the issue had been resolved. But I knew that we wouldn't know if it really fixed the issue until we got it out to users and saw how it acted in the real world. So that's what we did. We merged the fix in for the next release and then closely monitored how it acted.

Here's what we saw:
{{< figure src="stability.png" alt="Line graph showing that the stability of the version with the fix is higher than the version without it." >}}

As you can see, the stability for the version with the fix consistently stays above the stability for the version before it. It's not a huge percentage difference (because we have quite a few users and only a small number of them ever encountered this crash), but it got rid of a few hundred crashes per day, so it probably made a few people's day better, even if they didn't realize it. 🙌🏼

## Wrap up
So what can we learn from this experience? I have a couple of takeaways. First, **take a breath**. It always pays to spend some time thinking through, reasoning about, and talking out bugs that you don't understand. If I had just taken the stack trace and tried to dig into that specific view I would have wasted a bunch of time. Second, it is important to **do your homework thoroughly in cases where there is not a clear line between the crash report and the bug causing it**. If I had spent a little more time investigating crash reports, I may have been able to narrow down the version that introduced the bug, which may have made the fix more obvious to me. Third, **get familiar with the tools available**. Most iOS developers I know are pretty comfortable with `print` statements and breakpoints, but way fewer of them are comfortable with Instruments (or have ever even opened it). It probably doesn't make sense to become an expert in of all the various tools before you actually need them, but you should at least learn about what is available to you so when the time comes that you need one, you'll know that it exists. [This should get you started.](https://www.raywenderlich.com/16126261-instruments-tutorial-with-swift-getting-started)

And finally, **sometimes you can actually fix bugs that are beyond your ability**! I know it can feel hopeless when you're looking at a cryptic crash log and trying to reason about something that is theoretical. And sometimes you won't find the answer. But if you stick with it, keep trying, and seek out the help you need, you can actually fix some of these bugs and grow as a developer in the process. When it happens, it feels amazing, but even when it doesn't you can still learn from the experience.

Let me know about a time you tried to tackle a bug that you felt was too hard for you. Did you find the fix? What did you learn?
