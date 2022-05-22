---
date: 2018-12-07
title: Advent of Code 2018 - Day 6
slug: aoc2018/day6
tags: ["Advent of Code"]
description: My understanding of Day 6’s first problem is given a list of coordinates like the one below, return an Int which is the size of the largest open area that isn’t infinite (using Manhattan or taxicab distance) around a point.
---
## [Problem 1](https://adventofcode.com/2018/day/6)
My understanding of Day 6’s first problem is this: given a list of coordinates like the one below, return an Int which is the size of the largest open area that isn’t infinite (using Manhattan or taxicab distance) around a point. The sample data returns 17.
```
1, 1
1, 6
8, 3
3, 4
5, 5
8, 9
```

### Method
My method for solving this problem looks like this:
- Make a Point struct that has an X and Y coordinate and a method to calculate its distance from another point.
- Break the input into an array, and make an array of points out of it.
- Figure out the smallest and largest X and Y. These will be our bounds.
- Make an  `areaDict` dictionary to hold the points we are given in the input as keys, and array’s of the points that belong to them as the values.
- Cycle through all the Xs and Ys from the smallest to the largest of each
	- Make another dictionary that holds each point given in the input as keys, and its distance from the current point as values
	- Populate that dictionary with the distances
	- Find the smallest distance, and if there is only one, add the point to the `areaDict` dictionary
- Filter the area dictionary to get rid of any that point that has values which touch the bounds. Those are infinite.
- Find the biggest value in the filtered dictionary and return the count of its points.

### Implementation
My `Point` struct looks like this:
```swift
struct Point: Hashable {
    var x: Int
    var y: Int

    func calculateDistance(from point: Point) -> Int {
        return abs(self.x - point.x) + abs(self.y - point.y)
    }
}
```
It is Hashable so that I can make it the key in a dictionary, and it has variables for the X value and the Y value of the point. `calculateDistance()` takes a point and calculates the Manhattan or taxicab distance from the point. The equation for that is the absolute value of X1 - X2 plus the absolute value of Y1 - Y2. You can read more about it in [this Wikipedia article](https://en.wikipedia.org/wiki/Taxicab_geometry) .

My `parseInput()` function looks like this:
```swift
func parseInput(_ string: String) -> [Point] {
    let array = string.components(separatedBy: CharacterSet(charactersIn: "\n"))

    let pointArray = array.compactMap() { string -> Point? in
        let array = string.components(separatedBy: .punctuationCharacters).joined().components(separatedBy: .whitespaces)
        guard array.count > 1, let x = Int(array[0]), let y = Int(array[1]) else { return nil }
        let point = Point(x: x, y: y)
        return point
    }

    return pointArray
}
```
I first split up the input string into an array of each line and then I make an array of Points by calling `compactMap()` on that array. Inside of the closure, I try to pull out the X and Y values and make Ints out of them, if that doesn’t work, I just skip that line by returning nil. This works here because the input data is perfectly uniform. If I am able to get the values, I make a Point out of them and return it.

The first part of my main function looks like this:
```swift
let pointArray = parseInput(string)

guard let maxX = pointArray.max(by: { $0.x < $1.x }), let maxY = pointArray.max(by: { $0.y < $1.y }), let minX = pointArray.min(by: { $0.x < $1.x }), let minY = pointArray.min(by: { $0.y < $1.y }) else { return -1 }
```
I get an array of Points by calling the `parseInput()` function. After that,  I pull out the smallest and largest X, and the smallest and largest Y. That gives me the bounds of the grid that we care about.

The main part of the function looks like this:
```swift
var areaDict: [Point: [Point]] = [:]
for x in minX.x...maxX.x {
    for y in minY.y...maxY.y {
        var distanceDict: [Point: Int] = [:]
        let testPoint = Point(x: x, y: y)
        for point in pointArray {
                distanceDict[point] = point.calculateDistance(from: testPoint)
        }
        if let min = distanceDict.min(by: { $0.value < $1.value }) {
            if distanceDict.values.filter({ $0 == min.value }).count == 1 {
                areaDict[min.key, default: []].append(testPoint)
            }
        }
    }
}
```
First, I make a dictionary to hold each point from the input data, and an array of points that it is closest to. Then I loop through all of the Xs and Ys that we care about (the ones inside the bounds), make a Point out of each, and a dictionary to hold its distance to each of the input points. Then I calculate the distance to each of the input points and store them in that dictionary. Finally, I find the smallest distance in that dictionary and if there is only one Point that is that distance, add the Point we’re currently checking to the array in the `areaDict` at the Point it is closest to.

Finally, after populating the `areaDict`, the rest of the function looks like this:
```swift
let filteredDict = areaDict.filter { (key: Point, value: [Point]) -> Bool in
        let array = value.filter() {
            return $0.x == minX.x || $0.x == maxX.x || $0.y == minY.y || $0.y == maxY.y
        }
        return !(array.count > 0)
    }

    let biggestArea = filteredDict.max(by: { $0.value.count < $1.value.count })

return biggestArea?.value.count ?? -1
```
I get a new dictionary which is `areaDict` filtered of all the input Points that have any closest Points that touch the bounds, because that means they are infinite on this grid. I then find the input Point that has the largest count of points (i.e. area) and then return that value. I use the nil coalescing operator because `biggestArea` is optional, and that was the easiest/ fastest way to unwrap it. I return -1 as the default because that will never be a value the function should produce so it will be a sign to me that something has gone wrong.

The answer for my input was `4398` and it takes about 3.5 seconds to calculate on the command line using the method described in [my AoC Setup post]({{< ref "2018-12-10-Advent-of-Code-2018-Setup" >}}).

## [Problem 2](https://adventofcode.com/2018/day/6#part2)
I would describe the second problem like this: given the same list of coordinates, return an Int which is the total area in which each Point’s distance from all of the given coordinates adds up to something less than a given target value. The sample data returns 16 with a target value of 32.

### Method
This is how I solved this problem:
- Get an array of Points and find the bounds the same as before.
- Make a Set of Points that will hold all of the “safe points”
- Cycle through each X and Y within the bounds and check if they are “safe” by adding up its distances from each of the input Points.
- If they are safe, add them to the set.
- Return the count of safe points.

### Implementation
I used the same `Point` and `parseInput()` function as before, and the first few lines of my main function for this part is identical to the main function from the first part, except that I wrote this function to take in a `targetValue` which is an Int, so that I could test against the sample data and my input data with different target values.

The section that is different looks like this:
```swift
var safePoints: Set<Point> = Set()
for x in minX.x...maxX.x {
    for y in minY.y...maxY.y {
        var distanceDict: [Point: Int] = [:]
        let testPoint = Point(x: x, y: y)
        for point in pointArray {
            distanceDict[point] = point.calculateDistance(from: testPoint)
        }
        let sum = distanceDict.values.reduce(0, +)
        if sum < targetValue {
            safePoints.insert(testPoint)
        }
    }
}

return safePoints.count
```
First, I make a Set to hold the safe points and set it to an empty Set. Then I loop through all the Xs and Ys in the bounds, make a Point out of each one and a dictionary to hold the distances from each of the input Points. Then I just reduce that dictionary to a single value and if it is less than the `targetValue` I insert the point into the `safePoints` set. At the end, I return the count of points.

The answer for my input with a target value of 10,000 is `39560` and it takes about 2.5 seconds to calculate on the command line.

## Reflections
- This was another interesting geometrical problem. Lots of practice keeping track of and comparing coordinates.
- It also forced me to think a little abstractly about the bounds to the grid and how to tell if a certain point was closest to an infinite number of points or not.

You can find all the code for my attempts so far in [my advent of code 2018 repository on GitHub](https://github.com/dillon-mce/advent-of-code-2018).
