---
layout: post
title: "Animating UILabels for stroke drawing"
date: 2017-02-3
categories: iOS
---

It's been a while since my previous post, [A scrolling animation for Tab Bar Controllers (iOS)]({% post_url 2015-12-25-scrolling-tab-bar %}). Let's stay in the topic of animations, and see how we can create a `UILabel` subclass which gets animated whenever it's displayed, like so:

*TODO*

## UIStrokeAnimatedLabel, a UILabel subclass

Ideally, we'd like to create a subclass of `UILabel`. This is so that wherever in the iOS SDK a `UILabel` is used, we can use our `UIStrokeAnimatedLabel` instead. And on top of being able to use it in code, we could also use it in Interface Builder. We could drag in a `UILabel` and change its class to our subclass, and see the changes at runtime.

UILabel also already provides, quite naturally, a great abstraction about how a label should be represented. It maintains state about the text, font, dimensions, and more. Developers already know about these fields, and we'll use them to re-implement the method that handles the drawing of the label.

Let's start with the following:

```
class UIStrokeAnimatedLabel: UILabel {
}
```

Because of our label is animated, we need to store some properties representing some paramaters associated with the animation.

Let's add some basic properties:

```
var animationDuration: Double = 1.0
var animationEnabled: Bool = true
```

We've also provided default values.

Because we're subclassing `UILabel`, we have access to the properties of that class. For the color of our stroke, let's use the `textColor` property of `UILabel`.
