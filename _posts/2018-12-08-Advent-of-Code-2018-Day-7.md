---
title: "AoC 2018 - Day 7"
tags: adventofcode
excerpt: "My understanding of Day 7’s first problem is this: given a  series of instructions like the one below, return a String which is the correct order in which the instructions will be executed."
---
## [Problem 1](https://adventofcode.com/2018/day/7)
My understanding of Day 7’s first problem is this: given a  series of instructions like the one below, return a String which is the correct order in which the instructions will be executed. The sample data returns `"CABDFE"`.

```
Step C must be finished before step A can begin.
Step C must be finished before step F can begin.
Step A must be finished before step B can begin.
Step A must be finished before step D can begin.
Step B must be finished before step E can begin.
Step D must be finished before step E can begin.
Step F must be finished before step E can begin.
```

### Method
My method for solving this problem looks like this:
- Parse the input into a dictionary which holds steps that are dependent on others as the key and an array of the steps they are dependent as the value, and an array of all the steps that need to be executed, sorted alphabetically.
- Make a set to hold the steps which are completed, and an array to hold the order all the steps in the order they are completed.
- Using a similar method to [the collapsing polymer problem]({{"/Advent-of-Code-2018-Day-5" | absolute_url}}), loop through a while loop until the letter array is empty.
	- Inside of that outer loop, loop through the array until a step that can be executed is found.
	- Execute that step. Insert it into the set of completed steps, append it to the results array, and remove it from the letter array.
- After the while loop, return the results array joined into a string.

### Implementation
My `parseInput()` function looks like this:
```swift
func parseInput(_ string: String) -> ([String: [String]], [String]) {
    let array = string.components(separatedBy: .newlines))
    var dependencyDict: [String: [String]] = [:]
    var letterSet: Set<String> = Set()
    for line in array {
        let letters = line.components(separatedBy: .whitespaces).filter() { $0.count == 1 }
        dependencyDict[letters[1], default: []].append(letters[0])
        letterSet.insert(letters[0])
        letterSet.insert(letters[1])
    }
    let letterArray = Array(letterSet).sorted()
    return (dependencyDict, letterArray)
}
```
First, I break the string into an array split on newlines.  Then I set up a dictionary to hold the steps that are dependent and an array of the steps they are dependent on and a set to hold all of the steps that need to be executed. Then I loop through each line in the array, break it up by splitting on whitespaces and filtering for single character elements. I add the dependency to the dependency dict and make sure both letters are in the letter set. At the end I make an array from the letter set, sort it, and then return a tuple of the dependency dictionary and a letter array.

My main function for part 1 looks like this:
```swift
func figureOutOrder(_ string: String) -> String {
    var (dependencyDict, letterArray) = parseInput(string)

    var completedSet: Set<String> = Set()
    var results: [String] = []
    while letterArray.count > 0 {
        for index in 0..<letterArray.count {
            let letter = letterArray[index]
            let dependencies = dependencyDict[letter]?.filter() { !completedSet.contains($0) }
            if dependencies == nil || dependencies == [] {
                completedSet.insert(letter)
                results.append(letter)
                letterArray.remove(at: index)
                break
            }
        }
    }
    return results.joined()
}
```
First, I get my dependency dictionary and letter array by calling `parseInput()` and set up variables to hold the completed steps and the resulting order. Then, inside a while loop that will continue until the letter array is empty, loop through the letter array until a letter is found that has no dependencies or, all of whose dependencies have been completed. Once one is found, I insert it into the completed set, add it to the results array, remove it from the letter array and break out of the current loop. Restarting from the beginning of the array every time a step is completed insures that if two steps could be completed at the same time, they will be completed in alphabetical order, as stipulated in the directions. Once the while loop completes, return the results array `.joined()`

The answer for my input was `"CHILFNMORYKGAQXUVBZPSJWDET"` and it takes about 2 milliseconds to find using the command line method described in [my AoC setup post]({{"/Advent-of-Code-2018-Setup" | absolute_url}}).

## [Problem 2](https://adventofcode.com/2018/day/7#part2)
I would describe the second problem like this: given the same set of instructions before, assuming that each step takes 60 seconds plus the letter’s index (e.g. A=61, B=62, Z=86) and assuming that there are up to five workers working on concurrent steps, return an Int which is how many seconds it will take to complete all the instructions.

### Method
How I solved this problem:
- Write a helper function to build a dictionary of how long each step will take.
- Add a custom Queue struct to represent a worker.
	- Has a private array to hold its work
	- A computed property which is the step it is currently working on (optional)
	- A function to push a new step onto the queue, a given number of times
	- A function to pop an element off of the queue
- Use the same `parseInput()` method to get a dependency dictionary and letter array, use the helper function to get a dictionary of letter values, set up an array of Queues to hold the workers based on the number passed into the function, make a set to hold completed steps, a set to hold steps being worked on, and an Int variable to hold the resulting number of seconds.
- Similar to part 1, make a while loop that will continue until the letter array is empty.
- The while loop will have two sections, the first will cycle through the queues and complete one “second” of each of their work and update the letter array if any of them finish. The second part will loop through all the remaining letters, try to find one with no dependencies, and if there are any open queues, add it to that queue.
- Each cycle through the while loop will add one second to the result “timer”

### Implementation
My helper function to set up the dictionary of letter values looks like this:
```swift
func setupLetterValueDictionary(_ offset: Int) -> [String: Int] {
    let alphabet = "abcdefghijklmnopqrstuvwxyz".uppercased()
    var value = 1 + offset
    var result: [String: Int] = [:]
    for letter in alphabet {
        result[String(letter)] = value
        value += 1
    }
    return result
}
```
Nothing very interesting, it just loops through the alphabet and adds the associated value to a dictionary at the key for each letter.

My custom Queue looks like this:
```swift
class Queue<T> {
    private var queue: [T] = []

    var currentlyWorkingOn: T? {
        return queue.first
    }

    func push(_ string: T, times: Int) {
        for _ in 0..<times {
            queue.append(string)
        }
    }

    func pop() {
        if queue.count > 0 {
            queue.remove(at: 0)
        }
    }
}
```
It is generic, mostly in case I need to reuse it in future code challenges. It has a private queue which is held in an array. It has a computed property which returns an optional element that is the first element of the queue or nil if it is empty. It has a function to push a new element onto the queue a given number of times, and it has a function to pop the first element off the queue. I did not return the element that is popped, because it isn’t necessary here.

The setup section of my main function looks like this:
```swift
var (dependencyDict, letterArray) = parseInput(string)
let letterValueDict = setupLetterValueDictionary(timeAddition)
var queues: [Queue<String>] = []
for _ in 0..<numOfWorkers {
    queues.append(Queue())
}
var completedSet: Set<String> = Set()
var currentlyWorkingOn: Set<String> = Set()
var result: Int = 0
```
I get the dependency dictionary and letter array by calling `parseInput()`, I get the letter values by calling `setupLetterValueDictionary()`, and I set up an array of queues to which I add as many queues as are called for when the function is called. I also make a set to hold the completed steps, a set to hold the steps that are being worked on, and a variable to hold the resulting number of seconds.

The first part of the while loop in the main function looks like this:
```swift
while letterArray.count > 0 {
        for queue in queues {
            let previouslyWorking = queue.currentlyWorkingOn
            queue.pop()
            if previouslyWorking != queue.currentlyWorkingOn {
                if let previouslyWorking = previouslyWorking {
                    completedSet.insert(previouslyWorking)
                }
            }
        }
        letterArray = letterArray.filter() { !completedSet.contains( $0 ) }
        if letterArray.count == 0 { break }
```
I loop through each queue, grab a reference to what it was working on, and then pop the first element. Then I check to see if the previous element is not the same as the current element. Because we are only adding one step to each queue at a time, we know that the only time they would be different is if we had some step in the queue and now we don’t. That means we’ve finished that step, so we’ll insert it into the set of completed letters. Then we filter the letter array to only have the letters that haven’t been completed and check to see if it is empty. If it is, that means we’re done and we break out of the loop.

The second half of the while loop looks like this:
```swift
    for letter in letterArray {
            let dependencies = dependencyDict[letter]?.filter() { !completedSet.contains( $0 ) }
            if (dependencies == nil || dependencies == []) && !currentlyWorkingOn.contains(letter) {
                for queue in queues {
                    if queue.currentlyWorkingOn == nil {
                        let times = letterValueDict[letter]!
                        queue.push(letter, times: times)
                        currentlyWorkingOn.insert(letter)
                        break
                    }
                }
            }
        }
        result += 1
    }
```
Here we first loop through the letters left in the letterArray, try to find one that doesn’t have any dependencies or all of whose dependencies have been completed, and that isn’t already being worked on by some other queue. If we find one, we’ll loop through all the queues and try to find one that isn’t already working on something. If we find one, we get the number of seconds that step will take to complete from the dictionary, push it onto the queue, insert it into the set of steps currently being worked on, and break out of this loop through the queues. At the end of each pass through the while loop, we add one to the resulting seconds. And after the while loop completes, return the result

The answer for my input is `891` with 5 workers and a 60 second offset. It takes about 45 milliseconds to calculate on the command line.

## Reflections
- This problem gave me some good practice with using Sets and Dictionaries.
- It also forced me to figure out a way to work out queues/concurrency with a possible different number of tasks being worked on at the same time.
- It also gave me the opportunity to figure out how to simulate the passage of time.

You can find all the code for my attempts so far in [my advent of code 2018 repository on GitHub](https://github.com/dillon-mce/advent-of-code-2018).
