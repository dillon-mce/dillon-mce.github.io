---
title: "AoC 2018 - Day 3"
tags: adventofcode
excerpt: "My understanding of day 3’s first problem is this: given a string that is a list of fabric claims like `/"#1 @ 1,3: 4x4\n#2 @ 3,1: 4x4\n#3 @ 5,5: 2x2\"` on a large grid of fabric (with the coordinates starting at the top left), return an Int which is the number square inches claimed more than once."
---
## [Problem 1](https://adventofcode.com/2018/day/3)
My understanding of day 3’s first problem is this: given a string that is a list of fabric claims like `"#1 @ 1,3: 4x4\n#2 @ 3,1: 4x4\n#3 @ 5,5: 2x2"` on a large grid of fabric (with the coordinates starting at the top left), return an Int which is the number square inches claimed more than once. The sample data returns 4.

### Method
To solve this problem I needed:
-  To define a Point struct to deal with coordinates
- Define a Claim struct that has an id, a size (x, y, h, w)
	- It also needs a computed property that is a set of the points it contains
	- A method that returns the points that intersect with another claim
	- A method to check if there is any overlap between claims. At first I thought I would only need the first two, but I found that it was too calculation time intensive, so I added a faster function that just does a little math to see if they overlap, and only getting all their points and finding the intersection if we know they overlap.
- A helper method to parse the input into an array of Claims
- The main function which will have a set of points, loop through all the claims, check them for overlapping area with all the other claims, and if it finds any, get their intersecting points and add them to the set.

### Implementation
My Point struct is pretty straightforward, it looks like this:
```swift
struct Point: Hashable, CustomStringConvertible {
    let x: Int
    let y: Int

    var description: String {
        return "Point(x: \(x), y: \(y))"
    }

}
```
It is hashable, so that I can make a set of Points, and I added a custom description, to make things a little easier to read when I printed stuff out.

The Claim struct looks like this:
```swift
struct Claim: Hashable {
    let id: String
    let x: Int
    let y: Int
    let height: Int
    let width: Int

    var internalPoints: Set<Point> {
        var set: Set<Point> = Set()
        for x in x..<(x + width) {
            for y in y..<(y + height) {
                let point = Point(x: x, y: y)
                set.insert(point)
            }
        }
        return set
    }
}
```
I probably could have defined it to use a point instead of an x and y, and maybe also defined the height and width off of a second point, but as I was thinking through this problem, I started with the Claim struct and by the time I had a Point, I didn’t want to go back and rewrite the code I had. It may make some of the math easier though. Probably worth looking into at some point. Other than that, the only interesting thing here is the internal point property, which calculates a set of all the internal points each time it is called. This cuts down on memory usage, but increases calculation time, which is why I try to only call it it when I have to.

The function that returns a set of intersecting points looks like this:
```swift
func getIntersectionPoints(with claim: Claim) -> Set<Point> {
    return self.internalPoints.intersection(claim.internalPoints)
}
```
It just finds and returns the intersection of the claim it is called on’s internal points and the claim it is called with’s internal points.

The function which calculates the overlapping area looks like this:
```swift
func overlappingArea(with claim: Claim) -> Int {
    let overlapX = min((self.x + self.width), (claim.x + claim.width)) - max(self.x, claim.x)
    let overlapY = min((self.y + self.height), (claim.y + claim.height)) - max(self.y, claim.y)

    guard overlapX > 0, overlapY > 0 else { return 0 }

    return overlapX * overlapY

}
```
I got the math from this [Geeks for Geeks article](https://www.geeksforgeeks.org/total-area-two-overlapping-rectangles/) and modified it a little to fit the need here (the math in the article assumes there is some overlap). Basically you get the whichever right side of the rectangle is smaller and subtract whichever left side is bigger and if the result is positive, they over lap on the x-axis. You do the same for the y-axis and if they are both positive, it means there is some overlap. I return an Int in case at some point I need to know how much they overlap, but I could just have easily returned a Bool because all I really need to know is if they overlap or not.

My function to parse the claim looks like this:
```swift
func parseInput(_ string: String) -> [Claim] {
    let array = string.components(separatedBy: .newlines)
    var results: [Claim] = []

    for (index, item) in array.enumerated() {
        let secondArray = item.components(separatedBy: .whitespaces)
        let id = secondArray[0]
        let originArray = secondArray[2].components(separatedBy: .punctuationCharacters)
        let boundsArray = secondArray[3].components(separatedBy: .lowercaseLetters)
        guard let x = Int(originArray[0]),
            let y = Int(originArray[1]),
            let width = Int(boundsArray[0]),
            let height = Int(boundsArray[1]) else {
                print("Couldn't convert something correctly. Check loop \(index + 1)")
                print("Array: \(secondArray)")
                continue
        }
        let claim = Claim(id: id, x: x, y: y, height: height, width: width)
        results.append(claim)
    }

    return results
}
```
It is a little bit of a mess. I start by splitting the input string into an array by newlines and giving myself an array to hold the claims that are made. Then I loop through the array to make a Claim out of each line. I used `.enumerated()` so I could grab the index for my print out if something failed.

To make a Claim, I split each line by whitespaces. The id is the first element of that array. The origin point is the third element in the array, which I split on punctuation. The height and width are the fourth element, which I split on lowercase letters, because it is an `"x"` in the input. I then pull out the individual elements, turn them into Ints and store them in variables. It that is successful, I make a claim from the pieces and append it to the array.

Finally, my main function looks like this:
```swift
func addOverlappingArea(_ string: String) -> Int {
    let claims = parseInput(string)
    var points: Set<Point> = Set()
    for i in 0..<claims.count-1 {
        for j in i+1..<claims.count {
            let claim1 = claims[i]
            let claim2 = claims[j]
            if claim1.overlappingArea(with: claim2) > 0 {
                points.formUnion(claim1.getIntersectionPoints(with: claim2))
            }
        }
    }
    return points.count
}
```
I get an array of claims from my `parseInput()` function and make an empty array of points to hold the overlapping points. Then I loop through the claims and check each one against all the others to see if they overlap, and if they do I get a set of of the intersecting points and add that to my points set.

The answer for my input was 111326 and it takes about 0.6 seconds to find the answer using the swift script method described in [my AoC setup post]({{"/Advent-of-Code-2018-Setup" | absolute_url}}).

## [Problem 2](https://adventofcode.com/2018/day/3#part2)
Part two was much simpler, now that I had all the infrastructure built out. The problem is basically: given a string of claims, return the ID of the single claim that doesn’t overlap with any others. The sample data returns #3.

### Method
- Use the same `parseInput()` method to get an array of claims.
- Make a set of Claims to hold the claims with no overlaps.
- Loop through all the claims, checking them against all the others and if they have any overlap, remove both from the set.
- At the end, if there is an element left, return it.

### Implementation
The only new code for this part was the main function:
```swift
func findNoOverlaps(_ string: String) -> Claim? {
    let claims = parseInput(string)
    var noOverlaps: Set<Claim> = Set(claims)
    for i in 0..<claims.count {
        for j in i+1..<claims.count {
            let claim1 = claims[i]
            let claim2 = claims[j]
            if claim1.overlappingArea(with: claim2) > 0 {
                noOverlaps.remove(claim1)
                noOverlaps.remove(claim2)
            }
        }
    }
    return noOverlaps.count == 1 ? noOverlaps.removeFirst() : nil
}
```
I get my claims from the `parseInput()` function, and make a set equal from that array of claims. Then I loop through each claim, checking it against all the other claims for an overlap. If there is any, I remove both from the set. This will loop through all of the claims, even if they have already been removed from the set, but I didn’t want to take the time to write it in a more efficient way because it already takes a negligible amount of time for the given input.

The answer for my input was claim #1019 and it takes about 0.2 seconds to find it running the script on the command line.

### Reflections
- This problem really stretched my memory of high school geometry class, and forced me to think about how to deal with a coordinate system in code.
- It also gave me the opportunity to explore the capabilities of Sets a little more, as well as making custom structs that are Hashable.

You can find all the code for my attempts so far in [my advent of code 2018 repository on GitHub](https://github.com/dillon-mce/advent-of-code-2018).
