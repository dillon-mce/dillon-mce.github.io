---
title: "Stop Throwing Away Errors"
date: 2022-08-27
tags: ["iOS", "Xcode", "Swift", "Debugging"]
---

We all know how it goes when we are trying to get something working. We want to get there as fast as we can and we get focused on the "happy path". Maybe we're not that familiar with the technology/language/concept, but we have one specific goal or task we are trying to achieve and we don't care so much about the edge cases or what will happen when things go wrong.

In an exploratory phase this is usually fine. Maybe you're writing a little script to solve some problem for yourself and no one else has to deal with it. Maybe you're just trying to figure out how something works and this code will never see the light of day anyways. Whatever the reason, it is probably ok to not handle all the edge cases and errors at first.

But one place I see a lot of newer iOS developers get stuck is when they think they've got the code for the happy path written, they go to run it, and it just... doesn't... work.

I had a psychology professor who used to say "When people don't know what to do, they do what they know how to do." He was saying it in the context of people with various addictions, but it is true of all people, and this is another good example. When newer devs get stuck here, they will reach for tools they are familiar with. They will add `print` statements and set break points and try to narrow down where the problem is. Sometimes this helps. It depends on where the bug is coming from. But there are many types of bugs that these tools are not particularly helpful in solving. One of those areas is in decoding.

I have lost count of the number of newer developers I have helped figure out decoding bugs, and I often see the same mistake. **They throw away errors that would help them track down the bug.** In this article we're going to look at some decoding bugs. They are a good example of a class of bug where the errors that are thrown are often the best tool for understanding what has gone wrong. It also happens to be a context where it is fairly easy to show a few specific examples, so it works well for an article of this size. But the message is true everywhere. Errors and the messages they carry are there for a reason and they are often a helpful tool for finding bugs and making your code more robust.

So don't throw them away!

Let's look at a specific example. To protect the egos of the various people I have helped with decoding bugs over the years (at least one of whom now works at Apple), I have made a sample project instead of using someone's actual code. But it is illustrative of the exact kinds of issues I have seen people run into in the real world.

## Sample App Overview
My sample app lets you look up random characters from the television show "Bob's Burgers". You can tap a button and it will load in a random character, show you their picture, their name, occupation and who voiced the character. There is also a link to the fandom wiki for that character. It uses [this api](https://bobs-burgers-api-ui.herokuapp.com) to fetch the information. Here's how it is organized:

There is a character model:
```swift
struct Character: Codable {
    let id: Int
    let name: String
    let image: URL
    let occupation: String
    let voicedBy: String
    let wikiUrl: URL

    enum CodingKeys: String, CodingKey {
        case id, name, image, occupation
        case voicedBy = "voiced_by"
        case wikiUrl = "wiki_url"
    }
}
```

There is a class that encapsulates access to the api with this interface:
```swift
class BobsBurgersApi {
    static let shared = BobsBurgersApi()

    func fetchCharacter(id: Int) async -> Character? { ... }
}
```

There is a view model that interacts with the api and provides the primitive building blocks for the view to display:
```swift
@MainActor
class CharacterViewModel: ObservableObject {

    @Published var character: Character?

    var title: String { character?.name ?? "" }
    var subtitle: String { character?.occupation ?? "" }
    var detail: String { (character?.voicedBy).map { "Voiced by: \($0)" } ?? "" }
    var learnMore: URL { character?.wikiUrl ?? URL(string: "https://bobs-burgers.fandom.com")! }

    private var api: BobsBurgersApi { .shared }

    func changeCharacter() {
        Task {
            character = await api.fetchCharacter(id: .random(in: 1...501))
        }
    }
}
```

And finally there is a `SwiftUI` view that shows the information:
```swift
struct CharacterView: View {
    @StateObject var viewModel = CharacterViewModel()

    var body: some View {
        VStack {
            AsyncImage(url: viewModel.character?.image) { phase in
                phase.image?.resizable()
            }
            .aspectRatio(contentMode: .fit)
            .frame(width: 300, height: 300, alignment: .center)
            Spacer()
            Text(viewModel.title)
                .font(.title)
            Text(viewModel.subtitle)
            Text(viewModel.detail)
                .font(.caption)
            if !viewModel.title.isEmpty {
                Link("Learn More", destination: viewModel.learnMore)
                    .font(.caption)
            }
            Button("New Character") {
                viewModel.changeCharacter()
            }.padding()
        }
        .multilineTextAlignment(.center)
        .padding()
    }
}
```

*I'm only loosely familiar with SwiftUI and I used this opportunity to explore it a little bit, so I am sure there are better/more idiosyncratic/efficient ways to write this view, but it isn't really the point of this article. Any SwiftUI experts out there, feel free to let me know how I could improve this. If I like it better than what I've written I'll up date this article and credit you (if you would like me to).*

## Fixing the Decoding
Now that you have an overview of the app as a whole, let's take a deeper look at how the decoding is working currently. Here's where we're starting:
```swift
// in BobsBurgersApi
private static let baseUrl = URL(string: "https://bobsburgers-api.herokuapp.com")!

func fetchCharacter(id: Int) async -> Character? {
    let url = Self.baseUrl
        .appendingPathComponent("character")
        .appendingPathComponent("\(id)")
    guard let (data, _) = try? await URLSession.shared.data(from: url) else {
        return nil
    }
    let character = try? JSONDecoder().decode(Character.self, from: data)
    return character
}
```

Very often I will come across code that is written like this, at least at first. The logic is fairly straightforward.
1. It builds the url by appending "character" and the given id to the api's base url
2. It tries to get the data from that url
3. It tries to make a `Character` from that data
4. If it is successful, it returns that character otherwise it returns `nil`

Right now, we'll find that this always fails to load a character and we don't get any feedback. So, reaching for familiar tools, we might add print statements like this:
```swift
func fetchCharacter(id: Int) async -> Character? {
    let url = Self.baseUrl
        .appendingPathComponent("character")
        .appendingPathComponent("\(id)")
    guard let (data, _) = try? await URLSession.shared.data(from: url) else {
        print("Couldn't fetch data")
        return nil
    }
    let character = try? JSONDecoder().decode(Character.self, from: data)
    if character == nil { print("Couldn't decode character") }
    return character
}

// prints: Couldn't decode character
```

This helps us a tiny bit, because now we can see that it is the decoding that is failing and not fetching the data, but it doesn't tell us anything about _why_ it is failing. Next, we could try printing the data itself out with something like this:
```swift
if character == nil { print(String(data: data, encoding: .utf8)!) }

// prints: {"error":"Error while retreiving data with id 157 in route character."}
```

Again this is a little more helpful. And in this case, it is enough to track down the problem. The api is not returning a character model. This is not a decoding problem, but either a problem with the server or with the request we are making. Looking back at [the documentation](https://bobs-burgers-api-ui.herokuapp.com/documentation#characters) I notice that the route is actually `/characters/:id`, not `character/:id`. So we update our code to use "characters" and now it prints:
```
{"id":3,"name":"Adam","image":"https://bobsburgers-api.herokuapp.com/images/characters/3.jpg","gender":"Male","hairColor":"Brown","relatives":[{"name":"Unnamed wife"}],"firstEpisode":"\"Mr. Lonely Farts\"","voicedBy":"Brian Huskey","url":"https://bobsburgers-api.herokuapp.com/characters/3","wikiUrl":"https://bobs-burgers.fandom.com/wiki/Adam"}
```
So now we get an actual character model back, but the decoding is still failing. What is the problem?

We could look through the keys and values one by one and make sure they line up with the `Character` struct that we have defined. This could work in this context because it is a relatively small model, but it can get very cumbersome if you are fetching a long list or a large/complex model. A better way (and the point of this whole article) is to use the error we get to give us some clues. So lets modify our `fetchCharacter` function to catch those errors instead of throwing them away and turning things in to optionals. Now it looks like this:
```swift
func fetchCharacter(id: Int) async -> Character? {
    let url = Self.baseUrl
        .appendingPathComponent("characters")
        .appendingPathComponent("\(id)")
    do {
        let (data, _) = try await URLSession.shared.data(from: url)
        let character = try JSONDecoder().decode(Character.self, from: data)
        return character
    } catch {
        print(error)
        return nil
    }
}
// note that this code is basically the same amount of code as we had before
// so don't try to use concision as a reason throw away errors
```

And it prints this when we run it:
```
keyNotFound(CodingKeys(stringValue: "voiced_by", intValue: nil), Swift.DecodingError.Context(codingPath: [], debugDescription: "No value associated with key CodingKeys(stringValue: \"voiced_by\", intValue: nil) (\"voiced_by\").", underlyingError: nil))
```

This may look intimidating (and that might be the reason many newer developers discard these errors), but it isn't as bad as it may seem and it is here to help. The error is `keyNotFound` and if we look a little further we see that the key it couldn't find is `"voiced_by"`. So if we hop back over to [the documentation](https://bobs-burgers-api-ui.herokuapp.com/documentation#characters) and scroll down to the "Character Schema", we see that the key is actually called `"voicedBy"`. At the same time we might notice that the one we have defined as `"wiki_url"` is actually `"wikiUrl"`. (If we didn't happen to notice, the error would tell us the next time we ran it.)

So let's fix that, and re-run. In this case we can actually just get rid of the `CodingKeys` enum because all of our property names match the keys in the   JSON.
```swift
//enum CodingKeys: String, CodingKey {
//    case id, name, image, occupation
//    case voicedBy = "voiced_by"
//    case wikiUrl = "wiki_url"
//}
```
Now we can re-run and see some actual characters on screen!

{{< figure src="bobs-characters.png" alt="Screenshots showing the app running and properly displaying characters from Bob's Burgers." >}}

However, if you tap through characters long enough, you'll find that some of them fail to decode. Let's see what the errors have to say:
```
keyNotFound(CodingKeys(stringValue: "voicedBy", intValue: nil), Swift.DecodingError.Context(codingPath: [], debugDescription: "No value associated with key CodingKeys(stringValue: \"voicedBy\", intValue: nil) (\"voicedBy\").", underlyingError: nil))
```
Interesting. It is the same error as before, only now we know that we have the key name right. Back to [the documentation](https://bobs-burgers-api-ui.herokuapp.com/documentation#characters)! Down in the "Character Schema" section we can see:

| Key     | Type                | Description                       |
| ------- | ------------------- | --------------------------------- |
|voicedBy | string \| undefined | The voice actor for the character |

That is this api's way of communicating that `voicedBy` will be a `String` if there is one, or it will be undefined (or `nil`) if there isn't. In Swift that is actually a different type that we call an optional string (`String?`, or `Optional<String>`), so let's update our model. While we're at it, let's check if anything else should be optional.

The only other one I see is `occupation`, so we'll update that one as well. Now our model looks like this:
```swift
struct Character: Codable {
    let id: Int
    let name: String
    let image: URL
    let occupation: String?
    let voicedBy: String?
    let wikiUrl: URL
}
```

Now, we can tap through Bob's Burger's characters to our heart's content and the decoding should never fail. But if it did, we'd still be printing that error to the console and should be able to track down the issue quickly.

## Nested Example
I noticed in the docs that some characters have a `relatives` array. If a certain character has known relatives who are also in the system, it will return them in an array of a slimmed down character model. That seems like interesting information, so let's add it to our app.

I'll call this version of the model `Relative` to make it clear what we're dealing with:

```swift
extension Character {
    struct Relative: Codable {
        let name: String
        let wikiUrl: URL
        let url: URL
    }
}

// in Character struct
let relatives: [Relative]?
```

Then, so we can see the relatives on screen, I'll add a new property to the view model and display it on the view:
```swift
// in CharacterViewModel
var subtitle2: String {
    let relatives = character?.relatives?.map(\.name).joined(separator: ", ")
    return relatives.map { "Relatives: \($0)" } ?? ""
}

// in CharacterView, after subtitle
if !viewModel.subtitle2.isEmpty {
    Text(viewModel.subtitle2)
}

```

Now, if you tap through characters, you'll find that many of them don't have relatives but the ones that do will show up in that list. Occasionally though, you'll find one that will fail decoding and it will print something like this error:

```
keyNotFound(CodingKeys(stringValue: "wikiUrl", intValue: nil), Swift.DecodingError.Context(codingPath: [CodingKeys(stringValue: "relatives", intValue: nil), _JSONKey(stringValue: "Index 0", intValue: 0)], debugDescription: "No value associated with key CodingKeys(stringValue: \"wikiUrl\", intValue: nil) (\"wikiUrl\").", underlyingError: nil))
```

It may take a little more effort to get to the bottom of what this error is talking about. It says that it can't find a `wikiUrl` and we might be tempted to think that is the `wikiUrl` on our `Character` model, because it has one of those. But if we keep reading, we can see in the `DecodingError.Context` that it is looking at the key `relatives` (which we know is an array), at the 0th index item (meaning the first item in the array), and there it can't find a key called `wikiUrl`. That means that the first relative in our array doesn't have a `wikiUrl`.

This is an interesting one because [the docs](https://bobs-burgers-api-ui.herokuapp.com/documentation#characters) say that `wikiUrl` is non-optional, but apparently it is. I printed out the json for this character and here is what it returns for `relatives`:
```
"relatives" : [
    {
        "name" : "Unnamed child"
    }
]
```

Apparently, there is at least one `Relative` who doesn't have a `wikiUrl` or a `url`, which we can account for in our model by making those optional (or just not decoding them, since we aren't using them).

```swift
struct Relative: Codable {
    let name: String
    let wikiUrl: URL?
    let url: URL?
}

struct Relative: Codable {
    let name: String
//    let wikiUrl: URL
//    let url: URL
}
```

*This is a good reminder that the docs are a good place to start, but they aren't always up to date. If you have access to the developer who is maintaining the backend, it would be good to alert them to the discrepancy so they can either bring the backend logic up to spec or update the docs to match the actual logic. Or, if it is an open source API, you could open an Issue and/or make a contribution!*

You can see from this slightly more complex example that these errors are helpful for getting to the root issue even if the problem is deeply nested in your JSON. They may not be in the most human-readable form, but it is possible to work your way down them and gain valuable insights. And, as with most things, you'll get better with practice and will eventually be able to skim them to get everything you need.

## Wrap up
The examples in this simple app may seem trivial, but they are reflective of the sort of debugging that is common to work through when you're trying to get a new network interface up and running. I have seen people spend lots of time trying to work out what is wrong with their decoding and come up with all sorts of odd and complex ways to try to pinpoint the problem. But as we have looked at today, errors, while they might seem intimidating and hard to read at first, are often full of helpful information that tells us exactly what the problem is.

So next time you're tempted to throw that error away, [don't!](https://media.giphy.com/media/IRkqguqMTKUne/giphy.gif)

Let me know your opinions on errors and when/when not to use them. And find the full starter project on [this branch](https://github.com/dillon-mce/dont-throw-away-errors/tree/starter-project) and the full finished product on [this branch](https://github.com/dillon-mce/dont-throw-away-errors).

## Post Scripts
Not all errors are as helpful in debugging your code as they are in the context of decoding. Just as a screw driver is very helpful for driving screws, but less so for driving nails, errors are helpful for tracking down and fixing some problems and less so for others. My intention here was just to point out that they _are_ a useful tool and that they are often overlooked by newer developers. Part of growing as a developer is learning the tools that are available to you and over time you will develop a sense of which one(s) will be helpful in different situations.

Another thing that we did not explore in this article is when/how to communicate to the user that some error has occurred. This will depend on your content/context and a little bit on personal taste. But generally speaking it is good practice to throw errors up the chain at least to the layer where your business logic lives. That way you at least have the information there and can make decisions about how to handle it in ways that make sense for your app.
