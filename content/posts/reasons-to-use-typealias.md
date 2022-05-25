---
title: "5 Reasons to Use Typealias"
date: 2022-05-25
tags: ["Swift"]
---

Swift has many features that allow us as the users of the language to customize how we write code, while still giving us all the protections of a statically typed language. Today we're going to talk about one of those features which seems small, but it packs a lot of punch.

The `typealias` keyword lets you define a "type alias", a custom name for a class, struct or any other existing type in Swift. Once you declare it you can use that new name anywhere you would have used the name of the original type.

```swift
typealias NewName = ExistingType

let newName = NewName() // the type of newName will actually be ExistingType
```

Simple, right? Basically you can give your type a nickname.

## Reasons to use it
We've all had nicknames and used them for our friends and family, but unlike your childhood nicknames, type aliases can provide a lot of benefit to you as a developer. Here's a list of the top five.

### 1. Clarity
One use for type aliases is to add clarity around what a type is used for. `Double` is a primitive type in Swift and it can be used in any number of ways, but when it is used in the context of dates and times it might be good to add a little more explanation about what we _mean_ with the value. Foundation does exactly that by adding a type alias called `TimeInterval` and documenting that "A `TimeInterval` value is always specified in seconds…".

Another good example comes from handling money. It is relatively common practice when dealing with money in code is to multiply the value by 100 and work with it as `Int` values to avoid rounding errors. If that is what you're doing, you might add some clarity to your code by adding a `Cents` type alias to communicate what you mean with the `Int` value.

```swift
/// Describes the value of one-hundreth (1/100) of a dollar (USD).
typealias Cents = Int

struct Product {
	let name: String
	let price: Cents
}

func buy(product: Product, currentBalance: Cents) throws {
	guard currentBalance > product.price else { throw .NotEnoughMoney }
	currentBalance -= product.price
	currentUser.add(product)
}
```

That's a lot clearer than having `Int` all over the place and either adding comments to describe the meaning or assuming that the reader knows what it means. (You know [what happens when you assume.](https://media.giphy.com/media/l1J9yApnGts8EXYoo/giphy-downsized-large.gif))

### 2.  Brevity
Another use is to take a long name and make it shorter. If you've ever worked with `AVFoundation` for instance, you'll know that there are some incredibly long delegate names out there (e.g. `AVCaptureFileOutputRecordingDelegate`) and it can be nice to provide a short alternative. Or maybe you are trying to keep the line width to a reasonable limit and the completion handler definition is too long to fit.

```swift
// Say you have this function that you are trying to make shorter
private func fetchFavorites(type: ItemType, completion: @escaping (Result<[Item], Error>) -> Void)

// you can typealias the result type
typealias ItemResult = Result<[Item], Error>

// and you can typealias the closure type
typealias ItemHandler = (ItemResult) -> Void

// then your function could look like this
private func fetchFavorites(type: ItemType, completion: @escaping ItemHandler)
```

Much shorter! Taking 98 characters down to 78.

### 3. DRY (Don't Repeat Yourself)
That leads to another good reason to use `typealias`. If you define a type alias for a specific result type, you can reuse that everywhere you need that result type. Maybe you have a function that gets _all_ the items of a type, or _all_ the items that belong to a user. Both of those could use this same type alias.

```swift
private func fetchAll(type: ItemType, completion: @escaping ItemHandler)
private func fetchAll(for user: User, compeltion: @escaping ItemHandler)
```

On top of that, the alias in this context makes it easier to make changes down the line. Maybe your backend folks want to change the returned result from an array of items to an object that contains some other information alongside the array. All you need to do is update the type alias and the compiler will then show you where you need to handle it.
```swift
typealias ItemResult = Result<ItemResponse, Error>
```

### 4. Combine Protocols
You can also use `typealias` to combine multiple protocols into one name. You may want to do this if you have multiple protocols that are commonly used together, but that define methods that aren't required to be handled by the same type. A good example of this from Foundation is `Codable`, which is really just a type alias for `Decodable` and `Encodable`. This way if your type can just be encodable or decodable if you only need the functionality of one or the other, or it can be _both_ by adopting the one protocol.

Maybe in your code the `UICollectionViewDelegate` and `UICollectionViewDataSource` are always the same object. You could define a type alias to encompass both. Same for `UICollectionViewDragDelegate` and `UICollectionViewDropDelegate`.

```swift
typealias CollectionViewDataSourceDelegate = UICollectionViewDataSource & UICollectionViewDelegate

typealias CollectionViewDragDropDelegate = UICollectionViewDragDelegate & UICollectionViewDropDelegate

typealias CollectionViewSuperDelegate = CollectionViewDataSourceDelegate & CollectionViewDragDropDelegate

// Use the type alias that combines them all
class ViewController: CollectionViewSuperDelegate {
    override func viewDidLoad {
		super.viewDidLoad()

		// and then this type can be all four of these things
		collectionView.dataSource = self
		collectionView.delegate = self
		collectionView.dragDelegate = self
		collectionView.dropDelegate = self
	}
}
```

### 5. Better Auto-complete
Finally, if you're lazy like me and rely on auto-complete to do most of your typing for you, you're in luck. Type aliases will show up there and definitely speed things up when they are specifying a generic type. `ItemResult` will auto-complete as a whole unit, but `Result<[Item], Error>` would take more keystrokes because you would either have to auto-complete `Result`, `Item` and `Error` separately, or just type it all out yourself.

### Bonus. Light-weight Type Replacement
With a type alias, you can name a tuple with labels and it will basically function as a `struct` without a lot of the bells and whistles. **I would not recommend doing this in production.** It can be helpful when you're prototyping, messing around with a script or _maybe_ in a test suite. But in almost all cases you should turn your prototype into a real `struct`, or pick an existing type that fits the shape of your data, before pushing your code to production.

```swift
// this is valid Swift
// you can name this tuple
typealias Coordinate = (x: Int, y: Int)

// use it as a parameter in your functions
func manhattanDistance(from first: Coordinate, to second: Coordinate) -> Int {
    return abs(first.x - second.x) + abs(first.y - second.y)
}

// and pass literal tuples to that function
manhattanDistance(from: (1,2), to: (3,4)) // results in 4

// but there is already a common type in most contexts where you write Swift
// called CGPoint that you should probably use instead
func manhattanDistance(from first: CGPoint, to second: CGPoint) -> CGFloat {…}
```

## Why You (Mostly) Shouldn't Use It
Type aliases are very useful and there are many good reasons to use them, but that doesn't mean you should rename every type in your code. So next time you're going to reach for it, be sure to weigh the cost against the benefits.

Type aliases add complexity to your code. Mental overhead. Any time a reader of your code (including future you) comes across an alias they will either have to option+click it in Xcode to see what it actually is, or they will have to switch it out for the real meaning in their head. If it is the first time they have encountered it, they may have to look up the actual definition several times before they internalize it. Sometimes this trade off is worth it, if you are reaping the benefits of one or more reasons listed above. But other times it is not.

You might find yourself in a situation where you want to rename something because of your preference, but you won't actually benefit from it, or the benefit is so small that it isn't worth the added complexity. I would advise against using a type alias in this case. Especially if you work on a codebase with a team larger than just you. It actually makes your code harder to read. And that is almost never a good thing.

```swift
// this alias actually reduces clarity _and_ brevity
typealias Number = Int

// this one adds brevity but reduces clarity
typealias TableController = UITableViewController

// these don't add much benefit and are kind of the opposite of dry
typealias StringArray = Array<String>
typealias IntArray = Array<Int>
typealias DoubleArray = Array<Double>
```

The cost gets even higher if you and/or your team spend a lot of time looking at your code outside of Xcode. If you have a regular code review process (you should) there is a good chance that significant amounts of time will be spent reading and reasoning about code for the first time from the GitHub/GitLab/Bitbucket/etc diff interface where you don't have the convenience of being able to option-click a type name to see its definition. There are times when the trade off is still worth it, but it is good to have a conversation with your team about what your collective line is, or to follow the established practice if it exists.

## Wrap Up
Type aliases are just one of the many features of Swift that makes it really customizable and extensible while still being totally type-safe. We've covered some of the many good reasons to use them, including adding clarity, reducing line-length, reducing unnecessary repetition, combining protocols, and improving auto-complete. And we've talked about how type aliases _always_ come at the cost of added complexity, so we should weigh the cost against the benefit before we use them.

What do you think? When do you use type aliases in your code? What makes the trade off worth it for you?
