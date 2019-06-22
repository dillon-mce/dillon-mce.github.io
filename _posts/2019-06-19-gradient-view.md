---
title: "Gradient View Snippet"
tags: swift snippets
excerpt: "A simple gradient view snippet you can drop into your project."
---
A while back I wanted to have a gradient as the background of my view controller. At the time, I just googled and found some hacky piece of code that worked, but wasn't very robust. Then later, in another project I tried to use the same code to put a gradient behind a label to help it pop out against whatever was behind it. It didn't render correctly because it was using `frame` when it should have been using `bounds` or vice versa. It was also adding a `CAGradientLayer` to the view, instead of changing the `layer` it already had. So I decided to sit down and figure out how to swap that out.

It turns out that `UIView` has an overridable property called `layerClass`, which lets you set the type of its `layer` to any subclass of `CALayer`. Armed with that knowledge, I came up with the following snippet:

<script src="https://gist.github.com/dillon-mce/bb4293cac35142342c66a2102a9c3a3f.js"></script>

I added a convenience method to quickly setup a gradient that covers most use cases that I have. But it also leaves direct access to the `gradientLayer` open so that you can do more low-level configuration if you need it.

You can use it anywhere you would use a `UIView`. For instance, you can set the class of the view of a view controller to be a `GradientView` instead and then set it to look however you would like:
```
var gradientView: GradientView {
    return view as! GradientView
}

@IBOutlet var bottomView: GradientView!

override func viewDidLoad() {
    super.viewDidLoad()
    gradientView.setupGradient(startColor: .blue,
                               endColor: .magenta)
}
```

This results in a view controller that looks like this:

{% include figure image_path="/assets/images/snippets/2019-06-19-gradient.png" alt="Screenshot of basic gradient"%}

You can adjust it at runtime if you want:
```
// In viewDidLoad
let tapRecognizer = UITapGestureRecognizer(target: self,
                                           action: #selector(changeGradient))
view.addGestureRecognizer(tapRecognizer)

@objc func changeGradient() {
    let startX = CGFloat.random(in: 0...1)
    let startY = CGFloat.random(in: 0...1)
    let endX = 1 - startX
    let endY = 1 - startY
    let start = CGPoint(x: startX, y: startY)
    let end = CGPoint(x: endX, y: endY)
    gradientView.setupGradient(startColor: .magenta,
                               endColor: .blue,
                               startPoint: start,
                               endPoint: end)
}
```

{% include figure image_path="/assets/images/snippets/2019-06-19-various-gradients.png" alt="Screenshots of various gradients."%}

You can add it to other views and stack them, and use whatever colors you want. They make great transitions between elements. Have fun with it.

```
@IBOutlet var bottomView: GradientView!

// In viewDidLoad
bottomView.setupGradient(startColor: .clear,
                         endColor: .white)
```

{% include figure image_path="/assets/images/snippets/2019-06-19-gradient-with-white.png" alt="Screenshots of gradient with white at the bottom."%}

Feel free to use it however you want, and if you have suggestions for ways to extend or improve it. Let me know!
