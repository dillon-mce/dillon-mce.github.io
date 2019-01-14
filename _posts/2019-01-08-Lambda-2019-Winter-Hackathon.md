---
title: "Lambda 2019 Hackathon"
tags: lambda
gallery:
  - url: /assets/images/hackathon/RA-Build-Checker-Screen.png
    image_path: /assets/images/hackathon/RA-Build-Checker-Screen.png
    alt: "Build Checker Screen"
  - url: /assets/images/hackathon/RA-Products-Screen.png
    image_path: /assets/images/hackathon/RA-Products-Screen.png
    alt: "Products Screen"
build-checker-gallery:
  - url: /assets/images/hackathon/Build-Checker-Screenshot.png
    image_path: /assets/images/hackathon/Build-Checker-Screenshot.png
    alt: "Build Checker Screenshot"
  - url: /assets/images/hackathon/Products-Screen-Screenshot.png
    image_path: /assets/images/hackathon/Products-Screen-Screenshot.png
    alt: "Products Screenshot"
med-checker-gallery:
  - url: /assets/images/hackathon/Check-Meds-Screenshot.png
    image_path: /assets/images/hackathon/Check-Meds-Screenshot.png
    alt: "Med Checker Screenshot"
  - url: /assets/images/hackathon/Check-Meds-Screenshot-2.png
    image_path: /assets/images/hackathon/Check-Meds-Screenshot-2.png
    alt: "Filtered Med Checker Screenshot"
---
Our team decided to build an app that would assist Life Insurance agents in quickly figuring out what products might be a good fit for their clients. At the moment the only way to do that is to look up the information in a series of charts provided by the various carriers of life insurance products. [These charts](https://naauniversity.com/assets/uploads/2018/02/BuildCharts021918-4.pdf) have a good amount of variation between the carriers and they haven’t really been consolidated well anywhere that we could find. [The medication charts](https://naauniversity.com/assets/uploads/2014/08/MedLists3opt.pdf) are even worse. They are super long and very inconvenient to work with. We felt that an app that reduced this complexity and only showed the user (the insurance agent) the information that they cared about would be welcomed in the industry and hopefully provide enough value that people would be willing to pay for it.

The main features we landed on building during the hackathon were a "build checker" where you can submit a combination of "gender/age/height/weight" and get back a list of plans they qualify for, and "medication checker" where you can submit a specific medication and see if it disqualifies you from a certain plan. The first thing that needed to happen (and bulk of the work) was in compiling and organizing the data on the back end. The data we had access to was not very well organized and not consistently formatted between carriers, so it took our data science guys ([A Apte](https://github.com/a-apte) and [Crawford Collins](https://github.com/crawftv)) most of the 30 hours to get everything in a format that we could query against. At the same time, having our own dataset would give us a slight advantage against any potential competitor as it is a barrier to building a similar app.

While they worked on getting the data into a usable format, [William VanDolah](https://github.com/wvandolah) got the back end set up so that we would have an API to send requests to. [Adam Hinkley](https://github.com/adamhinckley) built out the front end for the web app, and I built out the iOS app. The rest of this post will be about my experience working on the iOS section because that is pretty much all I know about. If the other guys write about their experience, I will be sure to link it here. You can find [the repository with all of my code here.](https://github.com/dillon-mce/winter-hackathon-2019/)

## UI Mockup
I’m not really a UI or UX designer, but the first thing I did – because it always helps me to wrap my mind around the project I am setting out to build – was to layout my idea for the UI in Sketch. Here are my original concepts for the design.

{% include gallery %}

I liked the idea of having calm colors, a having it take up most of the interface, because I like that look and I felt like it would make the whole experience feel cohesive. I also felt like it would lend it self to looking “professional” and “established”, which I assumed would be important to people trying to sell life insurance. That lead me to having white be the main color for any text or elements directly over the background color and having white “cards” that would hold information floating over the background. I also went for a relatively subtle gradient on the background, to add a little more depth. Overall, I believe it leads to a pretty sleek-looking interface.

## Lay Out Views
After I laid  out my idea for the UI, I shared it with the group to get their feedback, and so Adam could do what he could to style the web app in the same way. The consensus seemed to be that we liked the design, so we moved forward with it. I knew the back end stuff wouldn’t be ready yet, so I decided to get my storyboards laid out before I did any of the actual code for the app.

### Build Checker View
For the build checker view I just made a centered stack view, with a fixed width to put my elements in because I wasn’t sure exactly how many elements we would end up with. At one point we were talking about filtering by product type “Term” or what not. The final version has a segmented control for choosing gender, three text fields for age, height and weight respectively, and then a search button. I have the text fields set to only display the number keypad, hoping that will limit the input, but it may be better to transition those to a picker view or some other control at some point.

The search button sends you to a search results view which consists of a stack view with a title label, a container view to hold a table view of results and a clear button.

{% include gallery id="build-checker-gallery" %}

*Sidenote: I realize that it is a little normative to only have two distinct options for the segmented control, but gender only plays a role in one of the carrier’s products and those were the only options that they gave. I felt it was easier to just give the two options that were available for the API rather than having people type in something that wouldn't work.*

### Meds Checker View
The med checker view is a horizontally centered stack view with a fixed width and fixed spacing above and below it. At first it consisted of three text fields, one for the carrier, one for the product, and one for the medication you wanted to search for. But over time we realized it would be better if there was just a list of products that the user could choose from (which would also be only the products that we have data loaded for), so we switched out the first two text fields for a picker view. I also added a tableview that would let me display a list of all the medications we had data on. I made the third field into a “search field” which would filter the medications displayed in the table view. Finally, I added a clear button that would clear the results and reset the view.

{% include gallery id="med-checker-gallery" %}

For that search field I tried using a search controller with the search bar attached to the tableview, but it would fly up to the top of the screen when you tapped on it for some reason, and I couldn’t figure out a way to stop that from happening. Then I tried a search bar, but I couldn’t get that to look the way I wanted to. So I ended up just using the text field that I already had, observing the “textDidChangeNotification” on it, and filtering the tableview when that was called (but more about that in the implementation section).

### Miscellany
I also wrote a few extensions to help me get the look I had laid out in Sketch. First, I wrote an extension on `UIColor` to give me a place to hold my colors for the app.
```swift
extension UIColor {
///Initializes a UIColor with 8-bit RGBA values. Provide values from 0 to 255
    convenience init(_ red: CGFloat, _ green: CGFloat, _ blue: CGFloat, _ alpha: CGFloat = 255) {
        self.init(red: red/255, green: green/255, blue: blue/255, alpha: alpha/255)    }

    static let lightBlue = UIColor(66, 207, 242)
    static let placeholderBlue = UIColor(80, 158, 210, 127)
    static let textBlue = UIColor(80, 158, 210)
    static let darkBlue = UIColor(87, 99, 176)
}
```

Then I wrote an extension on `UIView` that would add a gradient to the view that you called it on, with the specified colors.
```swift
extension UIView {
    func addGradient(startColor: UIColor, endColor: UIColor) {
        let gradient: CAGradientLayer = CAGradientLayer()
        gradient.colors = [startColor.cgColor, endColor.cgColor]
        gradient.locations = [0.0, 1.0]
        gradient.startPoint = CGPoint(x: 0.3, y: 0.0)
        gradient.endPoint = CGPoint(x: 0.7, y: 1.1)
        gradient.frame = self.layer.frame

        self.layer.insertSublayer(gradient, at: 0)
    }
}
```
I did this originally because I thought I would need to call this in the setup for all of my views, but eventually I realized that I could just make a custom subclass of UIViewController, where I could set up the view once instead of in every view. I left this extension though, because it does abstract the process and I can always reuse it in the future if I need to.

I also added a simple little extension on `UIButton` that sets up the look that I laid out in Sketch.
```swift
extension UIButton {
    func setupCustomLook() {
        self.layer.borderColor = UIColor.white.cgColor
        self.layer.borderWidth = 2
        self.layer.cornerRadius = 6
    }
}
```

Although I didn’t add it until later, I’ll include my subclass of UIViewController here, since it’s only purpose is to take care of styling my views.

```swift
class RAViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        view.addGradient(startColor: .lightBlue, endColor: .darkBlue)
    }

    override var preferredStatusBarStyle: UIStatusBarStyle {
        return .lightContent
    }

}
```
All I do here is add the gradient to the view in `viewDidLoad()` and set the preferred style of the status bar to light content, but this way I only have to do it once, and I just make each of my view controllers a subclass of this class.

## Implementation
I’ll try to keep it brief in this section, and only talk about things that are interesting and not super repetitive.

### Model
I made several structs to hold the data in the app, and to make it easier to convert to JSON when I needed to POST it to the API. I ended up with a `Person` struct, a `MedicationQuery` struct, a `MedicationResponse` struct, and a  `Plan` struct . The medication query and responses could probably be consolidated, but since the backend was changing so frequently during the hackathon, I found it easiest to keep them separate. I also had a `MedicationModel` which handles the list of medications available to query and a `SearchModel` which handles the networking requests. Nothing super interesting, I just needed to adjust the structs over time as the end points and responses from the back end changed.

### Controllers
There is not really anything worth noting in the `BuildSearchViewController` . It just sets itself up and when you tap on search, it tries to make a `Person` out of the data, and if it can, it passes that on to the `SearchResultsViewController`

The `SearchResultsViewController` doesn’t do a whole lot else either. It just takes that Person object and fires off a network request with it.
```swift
if let person = person {
            SearchModel.shared.findQualifyingPlans(for: person) {
                self.resultsTableViewController.updateResults()
            }
        }
```

The `PlanResultsTableViewConroller` throws up a spinner view when it first appears, and then gets rid of it in the `updateResults()` function.
```swift
func updateResults() {
        DispatchQueue.main.async {
            if let spinnerView = self.spinnerView {
                UIViewController.removeSpinner(spinner: spinnerView)
            }
            self.spinnerView = nil
            self.tableView.reloadData()
        }
    }
```

The spinner view is an extension on UIViewController that I adapted from something I found on Stack Overflow.
```swift
class func displaySpinner(onView: UIView, thickness: CGFloat = 24.0) -> UIView {
        let frame = CGRect(x: onView.bounds.origin.x, y: onView.bounds.height/4, width: onView.bounds.width, height: onView.bounds.height/2)
        let spinnerView = IndeterminateLoadingView(frame: frame, strokeColor: .white, thickness: thickness)
        spinnerView.backgroundColor = .clear
        spinnerView.startAnimating()

        DispatchQueue.main.async {
            onView.addSubview(spinnerView)
        }

        return spinnerView
    }
```
It passes back a reference to the spinner view, so that you can cancel it later. That is another function in the same extension:
```swift
class func removeSpinner(spinner: UIView) {
        DispatchQueue.main.async {
            if let spinner = spinner as? IndeterminateLoadingView {
                spinner.stopAnimating()
            }
            spinner.removeFromSuperview()
        }
    }
```

The `IndeterminateLoadingView` is a custom animation I adapted from a project in Sprint 9 of the iOS course at Lambda. I think the most interesting part is the animations themselves but if feel free to take a look at [the whole file](https://github.com/dillon-mce/winter-hackathon-2019/blob/master/Risk%20Assessment/Risk%20Assessment/Animations/IndeterminateLoadingView.swift) if you’re interested.
```swift
private func setupAnimations() -> [CABasicAnimation] {
        let addStrokeAnimation = CABasicAnimation(keyPath: "strokeEnd")
        addStrokeAnimation.fromValue = 0.0
        addStrokeAnimation.toValue = 1.0
        addStrokeAnimation.beginTime = 0
        addStrokeAnimation.duration = duration/2
        addStrokeAnimation.isRemovedOnCompletion = true
        addStrokeAnimation.timingFunction = CAMediaTimingFunction(name: .easeIn)

        let rotateWheelAnimation = CABasicAnimation(keyPath: "transform.rotation")
        rotateWheelAnimation.fromValue = 0.0
        rotateWheelAnimation.toValue = CGFloat.pi * 2
        rotateWheelAnimation.duration = duration
        rotateWheelAnimation.timingFunction = CAMediaTimingFunction(name: .linear)

        let removeStrokeAnimation = CABasicAnimation(keyPath: "strokeStart")
        removeStrokeAnimation.fromValue = 0.0
        removeStrokeAnimation.toValue = 1.0
        removeStrokeAnimation.beginTime = duration/2
        removeStrokeAnimation.duration = duration/2
        removeStrokeAnimation.isRemovedOnCompletion = true
        removeStrokeAnimation.timingFunction = CAMediaTimingFunction(name: .easeOut)

        return [addStrokeAnimation, rotateWheelAnimation, removeStrokeAnimation]
    }
```
In the first half of the animation cycle, the stroke for the whole circle is added with “ease in” timing, and in the second half the stroke is removed with “ease out” timing. And, over the course of one full cycle, the wheel will also make one full rotation. The resulting animation is an interesting-looking, indeterminate loading wheel.

{% include figure image_path="/assets/images/hackathon/Loading-Screen.GIF" alt="Loading Screen" caption="Sorry about the quality the gif. It looks a lot better in the actual app, but you can get a feel for the animation."%}

The only other thing that I am doing that is somewhat different (and that I am sure there is a better way to do) is using the `textDidChangeNotification` on the medication search text view. I have never done that before, but it was fairly simple. I just adopted the `UITextFieldDelegate` protocol on my view controller, set it as the text field’s delegate in the `viewDidLoad` method, and added an observer for the notification. This is the function that I called with that observer:

```swift
@objc func searchText() {
        MedicationModel.shared.filter(by: medicationTextField.text)
        tableView.reloadData()
        tableView.scrollToRow(at: IndexPath(row: 0, section: 0), at: .top, animated: false)
    }
```
I just have my model filter by the text that is in the field, reload the tableView and scroll to the top. Like I said, I’m sure there is a more efficient way to do this, but this was the only way I could find in that short period of time that would let me style things in the way I wanted.

## Wrap up
Overall, I had a great time working on this hackathon with a few other Lambda students that I had never met before. It was exciting to work on a project with such a strict deadline and in competition with other people. Here are a few things that I learned that I’ll take into my next hackathon:
- **Plan well.** It is easy to waste a bunch of time on stuff that doesn’t really matter for your MVP (in our case it was authentication). Spend more time up front planning and you’ll waste way less time writing code that you don’t need.
- **Know your team.** Get to know your team early and as much as you can. I am guessing it is easier if you are in person, but even if you are remote, it is worth getting to know each other as people a little bit. Establishing open lines of communication. Getting a feel for everyone’s skill level, so you know how to delegate tasks. Etc.
- **Get a solid team lead.** Make sure there is a team lead who knows the MVP, knows the scope of the project, and can explain the “why” behind all the required features. Adam did this in our case, and I think he did a fine job, but there were a few times where some explanation or intention behind a feature was not clear to me and I wasted time because of it.
