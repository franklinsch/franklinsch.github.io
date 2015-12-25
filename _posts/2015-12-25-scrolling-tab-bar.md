---
layout: post
title: "A scrolling animation for Tab Bar Controllers (iOS)"
date: 2015-12-25
categories: iOS
---

Here's a simple Friday (Christmas) morning project I just finished: animating the transition between View Controllers in a TabBarController. [GitHub](https://github.com/franklinsch/iOSScrollingTabBarAnimation)

![iOSScrollingTabBarAnimation](https://github.com/franklinsch/iOSScrollingTabBarAnimation/blob/master/demo.gif?raw=true)

### What it takes

* Subclass `UITabBarControllerDelegate`

```swift
class ScrollingTabBarControllerDelegate: NSObject, UITabBarControllerDelegate {
    func tabBarController(tabBarController: UITabBarController, animationControllerForTransitionFromViewController fromVC: UIViewController, toViewController toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return ScrollingTransitionAnimator()
    }
}
```

* Return an instance of a class that adopts `UIViewControllerAnimatedTransitioning`, which you need to create

```swift
class ScrollingTransitionAnimator: NSObject, UIViewControllerAnimatedTransitioning {
```

* Implement `transitionDuration(transitionContext:)`

```swift
func transitionDuration(transitionContext: UIViewControllerContextTransitioning?) -> NSTimeInterval {
  return 0.4
}

```

* Implement `animateTransition(transitionContext:)`

We would like something like:

```swift
func animateTransition(transitionContext: UIViewControllerContextTransitioning) {
  self.transitionContext = transitionContext

    let containerView = transitionContext.containerView()
    let fromViewController = transitionContext.viewControllerForKey(UITransitionContextFromViewControllerKey)
    let toViewController = transitionContext.viewControllerForKey(UITransitionContextToViewControllerKey)

    containerView?.addSubview(toViewController!.view)

    var viewWidth = CGRectGetWidth(toViewController!.view.bounds)

    if tabBarController.selectedIndex < lastIndex {
      viewWidth = -viewWidth
    }

  toViewController!.view.transform = CGAffineTransformMakeTranslation(viewWidth, 0)

    UIView.animateWithDuration(self.transitionDuration((self.transitionContext)), delay: 0.0, usingSpringWithDamping: 1.2, initialSpringVelocity: 2.5, options: .TransitionNone, animations: {
        toViewController!.view.transform = CGAffineTransformIdentity
        fromViewController!.view.transform = CGAffineTransformMakeTranslation(-viewWidth, 0)
        }, completion: { _ in
        self.transitionContext?.completeTransition(!self.transitionContext!.transitionWasCancelled())
        fromViewController!.view.transform = CGAffineTransformIdentity
        })
}
```

But how to we get `tabBarController.selectedIndex` and `lastIndex`? We need them to know in which direction the scrolling should occur.

Let's pass them in `ScrollingTransitionAnimator`'s constructor:

```swift
init(tabBarController: UITabBarController, lastIndex: Int) {
  self.tabBarController = tabBarController
  self.lastIndex = lastIndex
}
```

And update the `delegate` accordingly:

```swift
class ScrollingTabBarControllerDelegate: NSObject, UITabBarControllerDelegate {
  func tabBarController(tabBarController: UITabBarController, animationControllerForTransitionFromViewController fromVC: UIViewController, toViewController toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
    return ScrollingTransitionAnimator(tabBarController: tabBarController, lastIndex: tabBarController.selectedIndex)
  }
}
```

### Wrapping it up

Tadaa!
Now all we need to do is set our `UITabBarController`'s `delegate` to an instance of `ScrollingTabBarControllerDelegate` and we're done!

The full code is available on [GitHub](https://github.com/franklinsch/iOSScrollingTabBarAnimation).
