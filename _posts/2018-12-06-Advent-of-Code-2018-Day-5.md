---
title: "AoC 2018 - Day 5"
tags: adventofcode
excerpt: "My understanding of Day 5’s first problem is this: given a String like `\"dabAcCaCBAcCcaDA\"`, return an Int which is the number of characters remaining after being reduced by removing pairs of letters which are adjacent, the same letter, and opposite cases."
---
## [Problem 1](https://adventofcode.com/2018/day/5)
My understanding of Day 5’s first problem is this: given a String like `"dabAcCaCBAcCcaDA"`, return an Int which is the number of characters remaining after being reduced by removing pairs of letters which are adjacent, the same letter, and opposite cases. For example `"cC"` and `"Aa"`. The sample returns 10.

### Method
My method for solving this problem looks like this:
- Make an outer while loop that will continue until you make it all the way through the String without finding any pairs that can be removed.
- Inside of the while loop, loop through all the letters in the String except the last, and check the current character against the next one.
- If the current letter doesn’t equal the next letter, but if the lowercased versions of each of them are equal, it means they meet the criteria for removal.
	- Remove them
	- Update our start index for the next time through the for loop
	- Break out of the current loop.
- We need to check the letter before the ones we just removed, because it may have been affected by the removal. But we only need to go back one letter, we don’t need to check the whole preceding String again.
- If they don’t match and the current index equals the second to last element, that means we have made it all the way through the String successfully, so update that Bool and end the while loop.
- Finally, return the resulting String.

### Implementation
My `collapsePolymer()` function for the first part looks like this:
```swift
func collapsePolymer(_ string: String) -> String {
    var array = Array(string)
    var madeFullPass = false
    var madeToIndex = 0

    while !madeFullPass {
        for i in madeToIndex..<array.count - 1 {
            let firstLetter = String(array[i])
            let secondLetter = String(array[i+1])
            if firstLetter.lowercased() == secondLetter.lowercased() && firstLetter != secondLetter {
                array.remove(at: i)
                array.remove(at: i)
                madeToIndex = i > 1 ? i - 2 : 0
                break
            } else if i == array.count-2 {
                madeFullPass = true
            }
        }
    }
    return String(array)
}
```
First, I get an array of the string, so I can loop through it easily and set up helper variables to keep track of whether I have made a full pass yet, and what index I have checked up to so far. Then I make a while loop that will continue until I’ve made a full pass. In side of that, I make a for loop that starts at the index I’ve made it too that I’m sure doesn’t need to be checked again and goes up to the array’s count minus one. Since we’re removing elements, that count is constantly changing, which is why it is necessary to break out of the loop every time we remove elements.

I get references to the letters I want to check, I make them strings so I can call `.lowercased()` on them, and then I have a two part test. If they are equal when they are both lowercased, but not equal when they are not, that means they are the same letter, but different cases, which is the criteria we’re looking for. If they meet that criteria, I remove both of them from the array, update the index I’ve made it to so far, and then break out of the current loop. If they don’t meet that criteria, but the index does equal the second to last index in the array, we’ve made a full pass.

I didn’t originally keep track of the index I have made it to, but it was taking too long, so I added that as a limit to how many times we’d need to run through all of the for loop. It is necessary because of the length of the data and the fact that I am restarting the for loop every time I remove a letter (which happens a lot). Part 1 takes about 0.2 seconds when keeping track of the index, and about 30 seconds when starting the for loop over at 0 every time, meaning it goes about 150 times faster when you make those few simple changes.

I also made the function return a String because it was easier to troubleshoot while I was writing the code. Then I got the real answer by calling `.count` on the String given back to me.

The answer for my input was `11720` and it takes about 0.2 seconds to calculate when running on the command line as described in [my AoC Setup post]({{"/Advent-of-Code-2018-Setup" | absolute_url}}).

## [Problem 2](https://adventofcode.com/2018/day/5#part2)
I would describe the second problem like this: given the same String as before, return an Int which is the length of the String after all reductions, after removing all of one letter (both cases). The sample data returns 4, which is produced after removing all the `c/C's`

### Method
This is how I solved this problem:
- Get a set of all the letters contained in the String (hint: it was all 26 of them.)
- Make a dictionary to hold the result after the removal of each letter.
- For each letter in the set, filter out that letter and then call `collapsePolymer()` on it.
- Store the result in the dictionary
- After all the letters have been checked, find the smallest resulting value and return it.

### Implementation
Here is my function to solve part 2:
```swift
func testWithoutCertainLetters(_ string: String) -> Int {
    let letters = Set(Array(string.lowercased()))
    var resultsDict: [Character: String] = [:]

    for letter in letters {
        let array = Array(string).filter() { String($0).lowercased() != String(letter) }
        let result = collapsePolymer(String(array))
        resultsDict[letter] = result
    }

    let answer = resultsDict.min(by: { $0.value.count < $1.value.count })
    return answer?.value.count ?? -1
}
```
First, I make a Set out of an Array that is made from the input String that has been lowercased. This gives me a set of each letter that is used in the String to check against. Then I make a dictionary of Characters and Strings, to hold the results after removing each letter.

Then I cycle through the letters, remove them from the String and call `collapsePolymer()` on it. I store the result in the dictionary with that letter as the key. At the end, I return the smallest result’s count.

It was only really necessary to keep them in a dictionary because I was curious what the spread looked like with removing each letter and that the winning letter was. Turns out, there wasn’t a lot of variance except with the winning letter. With my input, removing every letter except `"w"` resulted in a polymer between 11,000 and 11,999 characters long. Removing `"w"` resulted in the answer of `4956`. The whole thing takes about 4 seconds to calculate, running on the command line.

## Reflections
- This problem forced me to think of creative ways to limit the scope of the function to only what we cared about.
- It also gave me the opportunity to discover this double-loop technique where you keep looping through the outer loop until you make it all the way through the inner loop, and the inner loop keeps restarting every time an element is removed. That technique, paired with keeping track of where you need to restart the inner loop, becomes a fairly efficient way to cycle through the data and compress it. I could imagine a similar technique being used to compress an image or audio file.

You can find all the code for my attempts so far in [my advent of code 2018 repository on GitHub](https://github.com/dillon-mce/advent-of-code-2018).
