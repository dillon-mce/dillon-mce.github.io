---
title: "AoC 2018 - Day 4"
tags: adventofcode
excerpt: "My understanding of Day 4’s problem is this: given a string like the one below, return an Int that is the minute at which the guard who spent the most total minutes asleep was asleep the most, times that guard’s ID number."
---
## [Problem 1](https://adventofcode.com/2018/day/4)
My understanding of Day 4’s problem is this: given a string like the one below, return an Int that is the minute at which the guard who spent the most total minutes asleep was asleep the most, times that guard’s ID number. The sample data returns `240`.
Sample data:
```
[1518-11-01 00:00] Guard #10 begins shift
[1518-11-01 00:05] falls asleep
[1518-11-01 00:25] wakes up
[1518-11-01 00:30] falls asleep
[1518-11-01 00:55] wakes up
[1518-11-01 23:58] Guard #99 begins shift
[1518-11-02 00:40] falls asleep
[1518-11-02 00:50] wakes up
[1518-11-03 00:05] Guard #10 begins shift
[1518-11-03 00:24] falls asleep
[1518-11-03 00:29] wakes up
[1518-11-04 00:02] Guard #99 begins shift
[1518-11-04 00:36] falls asleep
[1518-11-04 00:46] wakes up
[1518-11-05 00:03] Guard #99 begins shift
[1518-11-05 00:45] falls asleep
[1518-11-05 00:55] wakes up
```

### Method
My method for solving this problem looks like this:
- Parse the data into a dictionary with the Guard’s IDs as the key, and an array of the dates that they are asleep and awake as the value.
- Count up all the minutes each guard is asleep in a dictionary with their ID as the key and an Int of the count of minutes as the value.
- Once I find the guard who is asleep for the most minutes, need to figure out which minute they were asleep the most at.
	- Make a dictionary of the minute as the key and an Int of the number of times the guard was asleep at that minute as the value.
	- Loop through each minute, see if the date range contains that minute and add 1 to the associated key in the dictionary
- Return the minute with the largest value times the guard’s ID

### Implementation
My function for parsing the input into a dictionary of guard ID’s and arrays of dates looks like this:
```swift
func parseInput(_ string: String) -> [String: [Date]] {
    let inputData = string.components(separatedBy: .newlines).sorted()
    let dateFormatter = DateFormatter()
    dateFormatter.dateFormat = "yyyy-MM-dd HH:mm"
    var currentGuard = ""
    var timestampDict: [String: [Date]] = [:]

    for i in 0..<inputData.count {
        let components = inputData[i].components(separatedBy: CharacterSet(charactersIn: "[]"))
        if components[2].hasPrefix(" Guard") {
            let num = components[2].components(separatedBy: .whitespaces)[2]
            currentGuard = num
            continue
        }

        guard let date = dateFormatter.date(from: components[1]) else { continue }

        timestampDict[currentGuard, default: []].append(date)
    }

    return timestampDict
}
```
I split the input data into an array separated by newlines, and I sort it because the input was not in chronological order. At first I was worried that I would have to write some logic to get it to sort correctly, but it worked out of the box because of the format of the dates in the input string. I make a `dateFormatter` whose data format matches that of the data, a `currentGuard` variable to hold the guard I’m currently working with, and a `timestampDict` variable to hold the parsed data.
Because each guard has a line that says when they begin their shift, followed by alternating lines saying when they fell asleep and when they woke back up, and because every guard starts their shift awake and always wakes back up for every time they fall asleep, all I need is the dates following the line that says which guard started their shift. I know that the even-indexed elements of the array will all be “falling asleep” dates and all the odd-indexed elements will be “waking up” dates. This becomes relevant later.
So I split each line on the bracket characters and if the line starts with `"Guard"`, then I know it is the beginning of a new shift, so I just set `currentGuard` to that guard’s ID number and continue. For each subsequent line until I hit another that starts with “Guard” I just pull out the date and append it to the array associated with the current Guard’s ID. And at the end I return the dictionary.

My function for counting the minutes each guard was asleep looks like this:
```swift
func countMinutesAsleep(_ guardDict: [String: [Date]]) -> [String: Int] {
    var sleepCount: [String: Int] = [:]
    for (key, value) in guardDict {
        for index in 0..<value.count-1 {
            if index % 2 == 0 {
                let interval = DateInterval(start: value[index], end: value[index+1])
                let minutes = Int(interval.duration/60)
                sleepCount[key, default: 0] += minutes
            }
        }
    }
    return sleepCount
}
```
I start by making a dictionary to hold the guard IDs and the number of minutes they’ve been asleep. Then I loop through all the keys and values in the dictionary. Inside of that I loop through each index in 0 up to 1 less that the value’s count. The value here being the array of dates associated with that guard. I only check the even-indexed elements, because those are the falling asleep dates. I make a DateInterval starting at the falling asleep date and ending at the waking up date. Then I get an Int of the interval property on that. The interval property value is n seconds, so I divide that by 60 to get the minute value. Finally I add that to the value for the dictionary at the current key. Once I’m out of both loops, I return the dictionary.

My function for counting the times a particular guard was a sleep at each minute looks like this:
```swift
func countTimesAsleepAtMinute(_ dates: [Date]) -> [Int: Int] {
    let minuteFormatter = DateFormatter()
    minuteFormatter.dateFormat = "mm"

    var minuteDict: [Int: Int] = [:]
    for minute in 0..<60 {
        for index in 0..<dates.count-1 {
            if index % 2 == 0 {
                guard let fallAsleep = Int(minuteFormatter.string(from: dates[index])),
                    let wakeUp = Int(minuteFormatter.string(from: dates[index+1])) else { continue }
                if minute >= fallAsleep && minute < wakeUp {
                    minuteDict[minute, default: 0] += 1
                }
            }
        }
    }
    return minuteDict
} I
```
I make a dateFormatter that will pull out just the minute value from a date (because that is all that is relevant in the given data), and a dictionary to hold the minutes and counts. Then I loop through every value from 0 up to 60 (all of the minute values), and inside of that loop through all of the dates in the array. Again, I only check the even-indexed items because those are the “falling asleep” dates. I use my minute formatted to get the `fallAsleep` minute and the `wakeUp` minute and if the current minute is between those two values, I add 1 to the dictionary value at the current minute’s key. This is not the most efficient way to do this, but it works and is efficient enough for the given data.

Finally, my main function to solve the problem looks like this:
```swift
func findSleepyGuard1(_ string: String) -> Int {
    let timestampDict = parseInput(string)
    let sleepCount = countMinutesAsleep(timestampDict)

    guard let biggest = sleepCount.max(by: { $0.value < $1.value }) else { return -1 }

    let sleepyGuardArray = timestampDict[biggest.key] ?? []
    let minuteDict = countTimesAsleepAtMinute(sleepyGuardArray)

    guard let frequentlyAsleep = minuteDict.max(by: { $0.value < $1.value }) else { return -1 }

    let minute = frequentlyAsleep.key
    let guardId = Int(biggest.key.components(separatedBy: CharacterSet(charactersIn: "#")).joined()) ?? 0

    return minute * guardId
}
```
I get a dictionary of the times each guard is asleep from the `pasreInput()` function and a dictionary of how many minutes each guard spent asleep from the `countMinutesAsleep()` function. I pull the guard who was asleep the most out of that dictionary, and then I call 	`countTimesAsleepAtMinute()` with their associated dates. I pull the minute they were frequently asleep out of that dictionary and then multiply it by the guard’s ID number.

The answer for my input was `85296` and it takes about 40 milliseconds to find it using the Swift script method described in [my AoC Setup post]({{"/Advent-of-Code-2018-Setup" | absolute_url}}).

## [Problem 2](https://adventofcode.com/2018/day/4#part2)
I would describe the second problem like this: given the same input String, return an Int which is the ID number of the guard who is asleep at the same minute the most, times the minute at which they are asleep the most. The sample data returns `4455`.

### Method
This is how I solved this problem:
- Use the same `parseInput()` function to get a dictionary of guard IDs and arrays of dates.
- Write a function that does the same thing as `countTimesAsleepAtMinute()` in the previous part, but does it for every guard and returns a dictionary of guard IDs and minute dictionaries.
- Once I have that dictionary, find the guard who was asleep the most frequently at any given minute.
- Return the minute times the guard’s ID

### Implementation
My new `countTimesAsleepAtMinute()` function looks like this:
```swift
func countTimesAsleepAtMinute(_ dates: [String: [Date]]) -> [String: [Int: Int]] {
    let minuteFormatter = DateFormatter()
    minuteFormatter.dateFormat = "mm"

    var minuteDict: [String: [Int: Int]] = [:]
    for (guardID, times) in dates {
        for minute in 0..<60 {
            for index in 0..<times.count-1 {
                if index % 2 == 0 {
                    guard let fallAsleep = Int(minuteFormatter.string(from: times[index])),
                        let wakeUp = Int(minuteFormatter.string(from: times[index+1])) else { continue }
                    if minute >= fallAsleep && minute < wakeUp {
                        minuteDict[guardID, default: [:]][minute, default: 0] += 1
                    }
                }
            }
        }
    }
    return minuteDict
}
```
The only real difference from the previous one is that I’ve added yet another loop outside of the others that loops through all the keys and values in the dictionary we’re given, and it saves the resulting array into the result dictionary, using the guard’s ID as the key. This is where the efficiency of those inner loops starts to make a difference, but for the given data, the extra time it takes is still negligible.

My main function looks like this:
```swift
func findSleepyGuard2(_ string: String) -> Int {
    let timestampDict = parseInput(string)
    let minuteDict = countTimesAsleepAtMinute(timestampDict)

    var mostMinutes: (key: Int, value: Int) = (0, 0)
    var consistentlySleepyGuard = ""
    for (guardId, dict) in minuteDict {
        guard let max = dict.max(by: { $0.value < $1.value }) else { continue }
        //print("Guard \(guardId) was asleep at minute \(max.key) \(max.value) times")
        if max.value > mostMinutes.value {
            mostMinutes = max
            consistentlySleepyGuard = guardId
        }
    }

    let minute = mostMinutes.key
    let guardId = Int(consistentlySleepyGuard.components(separatedBy: CharacterSet(charactersIn: "#")).joined()) ?? 0

    return minute * guardId
}
```
I get a dictionary of guard IDs and arrays of dates from the `parseInput()` function and I get a dictionary of guard IDs and dictionaries of minutes and minute counts from the `countTimesAsleepAtMinute()` function. Then I make variables to hold a tuple of the mostMinutes info and a string of the most consistently sleep guard. Then I loop through all the keys and values in the dictionary, pull the max value out of the value dictionary and if it’s value is bigger than the current `mostMinutes`, set `mostMinutes` to it and set the `consistentlySleepyGuard` to the current key. At the end, I return the guard ID number times the minute.

The answer for my input was `58559` and it takes about 0.2 seconds to find it using the script method.

## Reflections
- This problem gave me the opportunity to practice with dictionaries and arrays, and nesting them within each other.
- It also forced me to think about the computation complexity nested loops.

You can find all the code for my attempts so far in [my advent of code 2018 repository on GitHub](https://github.com/dillon-mce/advent-of-code-2018).
