---
title: "AoC 2018 - Day 1"
tags: adventofcode
excerpt: "My understanding of day 1’s first problem was this: given a string like `\"+1, -2, +3, +1\"` , return the Int it reduces to if you add and subtract all the numbers, starting at 0."
---
## [Problem 1](https://adventofcode.com/2018/day/1)
My understanding of day 1’s first problem was this: given a string like `"+1, -2, +3, +1"` , return the Int it reduces to if you add and subtract all the numbers, starting at 0. The actual input was much longer and the elements were separated by newlines instead of spaces, but I accounted for that in my function.

### Method
Here’s how I broke down the problem:
- Break up the string into an array
- Define a variable to hold the result
- Loop through that array and if the element starts with “+”, add it to result and if it starts with “-“ subtract it from result. (I realized later that it was probably a little cleaner to filter them into two separate arrays based on the first character, and then run reduce on each of those arrays, so I switched to that method.)

### Implementation
Swift has a great method for splitting strings into an array, separated by built in character sets (or one’s you define yourself) called `.components(separatedBy: )` and one of the built in sets is called `.whitespacesAndNewlines` so breaking up the string is fairly simple.

The result just needed to start at 0, so I set that. I also defined a custom CharacterSet for myself so I could use that to get the number without the sign in front of it. `let characterSet = CharacterSet(charactersIn: "+-"`

Here is my original pass at the calculations, using a for loop:
```swift
for element in array {
        guard let number = Int(element.components(separatedBy: characterSet).joined()) else { continue }
        if element.first == "+" {
            result += number
        } else if element.first == "-" {
            result -= number
        }
    }
```
I make sure I can get the number, and then if the element starts with `"+"` I add it to the result, and if it starts with `"-"` I subtract it.

Here is the method with filter and reduce:
```swift
let additions = array.filter({ $0.hasPrefix("+") }).compactMap() {Int($0.components(separatedBy: characterSet).joined())}
    let subtractions = array.filter({ $0.hasPrefix("-") }).compactMap() {Int($0.components(separatedBy: characterSet).joined())}

    result = additions.reduce(result, +)
    result = subtractions.reduce(result, -)
```
I filter by the plus or minus sign and then compact map the results to integers of the string without the plus and minus signs. Then I can just set the result to be the reductions of the two arrays.

At the very end I print and return the result. Problem solved. The answer for my input was 592.

## [Problem 2](https://adventofcode.com/2018/day/1#part2)
I would describe the second problem like this: given a string like `"+1, -2, +3, +1"`, return the first integer that is achieved twice in the sequence. If none is found the first time through, loop through it until you find one.

### Method
- Break up the string into an array
- Make a set to hold the frequencies hit, a variable to hold the current frequency and a variable to hold the result
- Make an outer while loop that will continue until a result is achieved
- Inside of the while loop, use the loop from part 1 to loop through and add/subtract each element to the current frequency.
- Check if the frequency is contained in the set of frequencies hit and if it is, set the result to the current frequency and break out of the loop.
- If it isn’t, insert it into the set.

### Implementation
My function definition is the same, it takes in a string and returns an Int, but I renamed it to `calculateFirstReusedFreq()`.

I used the same method as in part 1 to split the array and many of the variables I used were the same. The ones that are different are:
- I set the result to be an optional Int `Int?` so that I could set it to nil for the while loop.
- I added a frequency variable to hold the current frequency (same as the result variable in part 1)
- I added a `frequenciesHit: Set<Int>` variable to hold all the frequencies that had been hit so far. I chose a Set because the complexity of a look-up in a Set in Swift is O(1), so it wouldn’t take longer to check the current frequency against it as it grew.
- I also added a count variable that just kept track of how many times I needed to loop through the while loop to arrive at an answer (mostly because I was curious).

My while loop looks like this:
```swift
while result == nil {
        count += 1
        for element in array {
            guard let number = Int(element.components(separatedBy: characterSet).joined()) else { continue }
            if element.first == "+" {
                frequency += number
            } else if element.first == "-" {
                frequency -= number
            }
            if frequenciesHit.contains(frequency) {
                result = frequency
                break
            }
            frequenciesHit.insert(frequency)
        }
    }
```
The logic in the for loop is largely the same as part 1, except that I had to switch back to the for loop method so I could get access to each frequency as we went. Then I just check to see if the frequency has been hit before, set it as the result and break if it has, and add it to the set of frequencies hit if it hasn’t. And I just keep looping that loop until I get a result.

Once I get out of the while loop I print and return the result. Problem solved. The answer for my input was 241 (found on the 137th time through the while loop) and it took a little over 8 seconds to arrive at that answer running in a playground on my 2018 MacBook.

You can find all the code for my attempts so far in [my advent of code 2018 repository on GitHub](https://github.com/dillon-mce/advent-of-code-2018).

**Update:** After I switched over to the command line script method described in [my AoC 2018 Setup post]({{"/Advent-of-Code-2018-Setup" | absolute_url}}), part 1 takes 5 milliseconds, and part 2 takes 0.5 seconds on the same MacBook.
