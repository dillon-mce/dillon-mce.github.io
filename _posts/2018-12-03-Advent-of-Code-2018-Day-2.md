---
title: "AoC 2018 - Day 2"
tags: adventofcode
---
## [Problem 1](https://adventofcode.com/2018/day/2)
My understanding of the second day’s first problem was this: given a list of IDs like `"abcdef bababc abbcde abcccd aabcdd abcdee ababab"`, return a checksum which is the number of IDs that contain exactly two of the same character *times* the number of IDs that contain exactly three of the same character. IDs can count towards both numbers. The sample returns 12.

### Method
- Break up the input into an array of the individual IDs
- Write a function that takes a string and a count (int) and returns a bool that is whether that string contains exactly “count” of any letter
- Filter the array using that function for count 2
- Filter the array using that function for count 3
- Multiply the counts of the filtered arrays together

### Implementation
Again, I start my function by saying `let array = string.components(separatedBy: .whitespacesAndNewlines)`. That gets me an array of strings which are the IDs.

Then I write my helper function.
{% highlight swift %}
func containsMultiples(_ string: String, count: Int) -> Bool {
    let letters = Set(string)
    for letter in letters {
        let filtered = string.filter() { $0 == letter }
        if filtered.count == count { return true }
    }
    return false
}
{% endhighlight %}
This loops through each letter in the string, creates a filtered array of letters that are the same as the current letter, and checks if the filtered array’s count is the same as the count parameter given when the function is called. It returns true for the first one it finds, and if it doesn’t find any it returns false.

After that, the rest of the `produceCheckSum` function is pretty straightforward:
{% highlight swift %}
let contains2 = array.filter() { containsMultiples($0, count: 2) }.count
let contains3 = array.filter() { containsMultiples($0, count: 3) }.count

let result = contains2 * contains3
{% endhighlight %}
I get one array which is all the IDs that have at least one set of two identical characters, and then I get an array which is all the IDs that have at least one set of three identical characters, and then I multiply their counts together.

The answer for my input was 5000, and with the command line method described in [my AoC Setup post]({{"/Advent-of-Code-2018-Setup" | absolute_url}}), it takes about 30 milliseconds to find it.

## [Problem 2](https://adventofcode.com/2018/day/2#part2)
I would describe the problem for part 2 like this: given a list of IDs like `"abcde fghij klmno pqrst fguij axcye wvxyz"`, return a string that is the common letters between the two IDs that only differ by one letter. The sample returns “fgij”.

### Method
- Break up the array into individual IDs
- Write a helper function that takes two strings, checks for mismatches between them, and returns a bool that is whether the mismatches is less than 2.
- Loop through all the IDs and get an array that is filtered by that helper function. If that filtered array is more than 1 element long, break out of the loop.
- Loop through the two ID’s characters and, if they are the same, add them to a result string.

### Implementation
I break up the array in the same way described above. Then, my helper function looks like this:
{% highlight swift %}
func filterStrings(_ string1: String, _ string2: String) -> Bool {
    var mismatches = 0
    let array1 = Array(string1)
    let array2 = Array(string2)
    for i in 0..<string1.count {
        if array1[i] != array2[i] { mismatches += 1 }
    }
    return mismatches < 2
}
{% endhighlight %}
I start with the mismatches at 0, loop through the indexes in the first array, and check them against the character at the same index in the other array. Then, return a bool that is if the mismatches is less than 2. This method assumes both of the ID strings are the same length, which is how the input is formatted. If the IDs weren’t all the same length, you’d probably want to check for that.

The rest of my findCommonLetters function looks like this:
{% highlight swift %}
var resultString = ""
var filtered: [String] = []

for id in array {
    filtered = array.filter() { filterString(id, $0) }
    if filtered.count > 1 { break }
}

let array1 = Array(filtered[0])
let array2 = Array(filtered[1])
for i in 0..<array1.count {
    if array1[i] == array2[i] { resultString += String(array1[i]) }
}
{% endhighlight %}
First, I define a string and an array to hold my results. Then I loop through the IDs, filter them using my helper function,  and break out of the loop if the count of the filtered array is greater than 1. Then I grab references to the two ID arrays and loop through them, check if the letters are the same and append the letter to my result string if they are.

The answer for my input was `"ymdrchgpvwfloluktajxijsqb"` and it takes about 0.15 seconds to find it running as a Swift script.

## Reflections
- This problem gave me the opportunity to explore `.filter()` a little more, as well as defining the logic for it in an external function.
- I did have to resort to accessing elements of the arrays by index in a few places, which is not particularly “Swifty”, but I couldn’t come up with a reasonable alternative in a short amount of time.

You can find all the code for my attempts so far in [my advent of code 2018 repository on GitHub](https://github.com/dillon-mce/advent-of-code-2018).
