---
title: "Algorithms in Swift - Intro"
tags: algorithms swift
excerpt: ""
---
I am looking for an excuse to learn more about algorithms, so I am going to write a series of posts that cover some of the most common ones and examine at what they might look like in Swift. I know algorithms often feel like an academic topic – something for nerds in computer science classes. They can be intimidating because their explanations and examples tend to be math-heavy and there is a lot of hard-to-decipher jargon out there about them. But they are really useful for solving interesting problems, and knowing how to pick the right algorithm for the problem at hand can make you seem like a programming genius! In this post, I want to talk about algorithms in general, take a look at the binary search algorithm, and examine how to think about the complexity of an algorithm.

### Algorithms
[If you search Google](https://www.google.com/search?q=algorithm), you’ll find that an algorithm is “a process or set of rules to be followed in calculations or other problem-solving operations, especially by a computer.” You use algorithms to search and to sort. You use algorithms to find the shortest route between two points, or to determine the next best move to take in a game. They are basically a set of instructions that you give to the computer that allows it to accomplish a task. And I know what you’re thinking, that is *all code*. And as far as I know, that is technically true. At least according to the definition given above. But when we talk about algorithms, we are usually talking about solving *interesting* problems. These are problems that seem easy for a human to do (with a small number of inputs at least) but whose solutions are not straightforward to describe in a way that computers can understand.

Algorithms tend to be described as *elegant* or *clever*. They don’t list out explicitly all the steps that a computer would use to solve the problem, they are distilled down descriptions of how to solve the problem. You and I are unlikely to ever come up with one that wasn’t already described by some computer science professor or medieval Muslim. [Basically all of the good and useful ones have already been thought of](https://en.wikipedia.org/wiki/Timeline_of_algorithms), so our job when it comes to algorithms tends to be *finding* the right one and *adapting* it to solve our particular problem. Even so, it is a helpful exercise to go through trying to come up with them for yourself, as it will help you remember the algorithms when the time comes that you need to implement them in real code.

### Binary Search
Binary search is a relatively simple algorithm that can save **huge** amounts of time/calculations when you are searching against a large sorted list. Here is an example of what this algorithm might look like in real life (or at least what it would have looked like before we had digital dictionaries): Suppose you were reading an article or book and came across the word “obeliscolychny”. If you are like me, you probably wouldn’t know what that word means, and you would probably even have difficulty parsing out any meaning from the roots by just looking at it. So you, like any good reader, would pull out your dictionary and look up the word.

How would you go about doing that? Would you start at the first page and flip through them one at a time, scanning each page until you found the word? No! You would crack it open in the middle, check what letter you landed on, and move backwards if you had gone too far or forward if you had not gone far enough. You would quickly narrow down your scope until you found the word you were looking for. (It means “lighthouse” if you haven’t already Googled it.)

That’s how binary search works. It halves the number of possible elements with each step by checking the middle element. If it is too high, it can safely disregard all the elements above that element. If it is too low, it can safely disregard all the elements below that element. (This is why it only works if you are searching a sorted list.) It keeps doing that until finds the element it is looking for. Here is an example of what it might look like in Swift:
```swift
func binarySearch<T: Comparable>(_ array: [T], for element: T) -> Int? {
    var low = 0
    var high = array.count - 1

    while low <= high {
        let mid = (low + high) / 2
        let guess = array[mid]

        if guess == element { return mid }
        if guess > element {
            high = mid - 1
        } else {
            low = mid + 1
        }
    }
    return nil
}
```
First, I make variables to keep track of the low and high spots to check against. Then I make a while loop that continues as long as `low` is less than or equal to `high`  (meaning we still have elements to check). Each time through the loop, I get the midpoint between `low` and `high` and get the guess associated with that midpoint. If `guess` is the element we’re looking for, I return it. Otherwise, if `guess` is greater than the element we’re searching for (meaning it is too high) we set `high` equal to the current midpoint minus 1. If `guess` is less than the element (meaning it is too low) we set `low` equal to the current midpoint plus 1. If we get to the point were `low` is no longer less than or equal to `high`, it means the element doesn’t exist in this array, so I return nil.

In comparison, a “simple search” might look like this:
```swift
func simpleSearch<T: Equatable>(_ array: [T], for element: T) -> Int? {    
    for (index, item) in array.enumerated() {
        if item == element { return index }
    }
    return nil
}
```
As you can see, it is much less code. Fewer places for errors. It just starts at the beginning and moves forward until it finds the element it is looking for. If it gets all the way through and doesn’t find it, it returns `nil`. It may be a better option, or at least it may not make a significant difference, if you know for certain that there will be a relatively small number of elements you are searching against.

To test these functions (and illustrate how many steps they take) I made a little list and searched for an element on it using both sorts:
```swift
var list: [Int] = []
for i in stride(from: 1, to: 100, by: 2) {
    list.append(i)
}

binarySearch(list, for: 99)
simpleSearch(list, for: 99)
```
Then I added a couple helper variables in the and printed these results out to the console:
```
Binary search took 6 steps and completed in 3.898143768310547e-05 seconds.
Simple search took 50 steps and completed in 7.200241088867188e-05 seconds.
```
Binary search takes about 0.000039 seconds and simple search takes about 0.000072. With so few elements, binary search is only slightly faster in terms of time (and basically imperceptibly), but you can see that the number of steps is way lower with binary search. We’ll look at what a difference this can mean when you have way more inputs in the next section.

### Complexity
If you were only looking at the times above, you might guess that simple search takes twice as long as binary search. So if we were to search a list of 500,000 elements instead of 50, however long binary search took, simple search would take twice as long. Let’s look at how long it actually takes:
```
Binary search took 19 steps and completed in 6.508827209472656e-05 seconds.
Simple search took 500000 steps and completed in 0.1531449556350708 seconds.
```
Binary search didn’t even double in the amount of time it takes (although the number of steps more than doubled, I’m assuming that a chunk of the time it takes has to do with setting up the objects in memory, not the actual calculations), but simple search takes more than 2000 times as long as it did the first time. You can also see that the number of steps necessary for simple search is *way* higher than binary. So it is not just that simple search takes longer than binary, but that the number of calculations it takes to complete (in the worst case) grows at a different rate as a function of the number of inputs.

Let’s break that down. In the first example above, simple search took 50 steps to find the last element in a list of 50 items. In the second example, it took 500,000 steps to find the last element in a list of 500,000 items. This means that the number of calculations required in the worst case is the same as the number of inputs given. The way you’ll see this described when talking about algorithms is its complexity or its speed or its running time and it is communicated with [Big O notation](https://en.wikipedia.org/wiki/Big_O_notation) (insert sex joke here). The big O notation for simple search is `O(n)` because the number of calculations required is the same as the number of inputs (`n` stands for the number of inputs).

So what about Binary search? You don’t necessarily have to understand the math behind it, but since we are cutting the elements in half with each step, the number of calculations required in the worst case is log base2 of the number of inputs (rounded up to the nearest integer because you can’t have half a calculation). For instance log(50) is approximately 5.6 and so it took 6 steps when we had 50 items. log(500000) is about 18.9 and it took 19 steps when we had 500,000 items. So the big O notation for binary search is `O(log(n))`.

It is important to understand the complexity of the algorithm you are using if you are going to be calling it with more than a few inputs. Even algorithms with the worst complexity work ok with a small number of inputs, but many of them grow at a rate much faster than you might think. It is easy to write code that works with your sample data, but is totally unusable when it comes into contact with real-world data that is often a much larger set.

There are a few other common big O notations that you’ll see associated with algorithms:`O(1)`, `O(n * log(n))`, `O(n^2)`, `O(2^n)` and  `O(n!)`. The best is `O(1)` or *constant time* which means that it takes the same amount of time no matter how many inputs there are. One example of this in Swift is checking whether an element is contained in a `Set`. The next is `O(log(n))` or *log time* which grows very slowly. Binary search is a good example of this. Next is `O(n)` or *linear time* which grows just as fast as the number of inputs. We saw this with simple search. Next is `O(n * log(n))`  or *log-linear time*. A good example of this is a the quicksort algorithm (which I’ll cover in another post and link to here). Next is `O(n^2)` or *quadratic time*. An example of this is selection sort (again, I’ll link to this after I write about it). After that is `O(2^n)` or *exponential time*. One example of this is bogo sort (also known as permutation sort). Finally we have `O(n!)` or *factorial time*. An example of this is the algorithm to solve the [traveling salesman problem](https://en.wikipedia.org/wiki/Travelling_salesman_problem).

{% include figure image_path="/assets/images/algorithms/Big-O-Graph.png" alt="Graph showing the growth rates of different complexities." caption="As you can see, algorithms at the beginning of our list grow at a very slow rate and the ones at the end grow incredibly quickly."%}

### Wrap Up
I hope this has been helpful. I know I learned a lot in writing it. Quick recap:
- Algorithms are condensed descriptions of how to solve a particular problem, especially for computers.
- Binary search is faster than simple search, but requires a sorted list and a little more code.
- The complexity, or the rate of the number of necessary calculations as a function of the number of inputs, of a function can be described with big O notation.
