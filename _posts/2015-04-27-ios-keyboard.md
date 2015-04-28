---
layout: post
title: Keyboard Management in iOS
---

One of the first stumbling blocks I ran into while learning iOS development is managing the keyboard. It was a little surprising that the standard keyboard interface found in most apps requires a fair amount of boilerplate code. Rather than write another tutorial about managing the keyboard, I wanted to highlight a couple of the complications I encountered.

<!--more-->

For an thorough introduction to interacting with the keyboard, check out these tutorials:

Rafal Augustyniak’s [blog](http://macoscope.com/blog/working-with-keyboard-on-ios/) gives the most complete explanation of working with the iOS keyboard I have seen, complete with animations demonstrating the order of keyboard notifications.

[This](http://creativecoefficient.net/swift/keyboard-management/) Creative Coefficient post is a straightforward code example with a sample implementation for registering and responding to keyboard notifications. I use this setup in my apps.

For a comprehensive overview of working with UIScrollView and AutoLayout, check out Mike Woelmer’s post [here](http://spin.atomicobject.com/2014/03/05/uiscrollview-autolayout-ios/).

The most frustrating aspect of handling keyboard input is manually moving content out of the way. This requires a fairly complicated setup, at least for a beginner. Textfields must be placed in a scroll view in order to respond to keyboard events. Getting the scroll view configured using Autolayout was difficult. My advice is to *make sure to place the text fields inside a wrapper view*. This makes it significantly easier to position content inside the scroll view’s content view. I would definitely recommend watching Paul Hegarty's lectures on [Developing iOS 8 App with Swift](https://itunes.apple.com/us/course/developing-ios-8-apps-swift/id961180099) before getting started; lecture 8 deals with Autolayout.

The second bit of functionality I found unintuitive was navigating between textfields. Most of the introductory tutorials I read dealt with positioning content in the storyboard using Autolayout. While storyboards make designing user interfaces significantly easier, there are still plenty of instances where it is necessary to create elements in the code. Adding the ubiquitous button bar above the keyboard is one such instance.

I create the button bar in a private method that is called in viewDidLoad.

{% highlight swift %}

private func addKeyboardButtonBar() {
    var toolbar: UIToolbar = UIToolbar(frame: CGRectZero)
    toolbar.barStyle = UIBarStyle.Default

    var flexSpace = UIBarButtonItem(barButtonSystemItem: UIBarButtonSystemItem.FlexibleSpace, target: nil, action: nil)
    var done: UIBarButtonItem = UIBarButtonItem(title: "Done", style: UIBarButtonItemStyle.Done, target: self, action: Selector("done"))

    let prev = UIBarButtonItem(title: "Prev", style: UIBarButtonItemStyle.Plain, target: self, action: Selector("moveField"))
    let next = UIBarButtonItem(title: "Next", style: UIBarButtonItemStyle.Plain, target: self, action: Selector("moveField"))

    toolbar.items = [prev, next, flexSpace, done]
    toolbar.sizeToFit()

    firstTextField.inputAccessoryView = toolbar
    secondTextField.inputAccessoryView = toolbar
}

{% endhighlight %}

{% highlight swift %}

override func viewDidLoad() {
    super.viewDidLoad()
    addKeyboardButtonBar()
}

{% endhighlight %}

The method adds UIBarButtonItems to move between text fields and hide the keyboard. You then assign the toolbar to the textfields' input accessory views.

I implement a moveField() and done() function to handle actions from the previous/next and done buttons in the toolbar. The button items for previous and next are standard text buttons. Surprisingly, the simple left/right arrows that you typically see are not a built in style. You would have to create those with a custom UIImage.

{% highlight swift %}

func done() {
    firstTextField.resignFirstResponder()
    secondTextField.resignFirstResponder()
}

func moveField() {
    if activeTextField == firstTextField {
        firstTextField.resignFirstResponder()
        secondTextField.becomeFirstResponder()
    } else {
        firstTextField.resignFirstResponder()
        secondTextField.becomeFirstResponder()
    }
}

{% endhighlight %}

In this case there are only two text fields to move between. If you have multiple fields, it might be easier to create an array of UITextField outlets and use an index to page through them.
