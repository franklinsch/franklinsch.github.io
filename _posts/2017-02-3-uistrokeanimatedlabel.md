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

Because we're subclassing `UILabel`, we have access to the properties of that class. For the color of our stroke, let's use the `textColor` property of `UILabel`:

```
lazy var strokeColor: UIColor = {
  return self.textColor
}()
```

## Drawing
Let's override `draw(rect:)`. This method is responsible for drawing the label into its view in a `UILabel`. Let's fill it in like so:

```
if !animationEnabled {
  super.draw(rect)
  return

  performStrokeAnimation()
}
```

Let's define `performStrokeAnimation()`.

```
private func performStrokeAnimation() {
    guard let text = text else {
      return
    }

    // Split the words.
    let words = text.characters.split(separator: " ").map(String.init)

    // Create a CALayer for each word.
    let wordLayers: [CALayer] = words.flatMap({ layer(for: $0) })
    
    // Create a view with all the layers.
    let textView = view(from: wordLayers)
    
    // Center the text view.
    textView.frame.origin.x = (frame.width - textView.frame.size.width) / 2
    textView.frame.origin.y = (frame.height - textView.frame.size.height) / 2
    
    addSubview(textView)
    
    let animation = CABasicAnimation(keyPath: "strokeEnd")
    animation.duration = animationDuration
    animation.fromValue = 0
    animation.toValue = 1
    
    // Perform the animation.
    animateSublayers(textView.layer, animation: animation)
}
```

Easy enough, right? Now let's define our helper methods.

## Helper methods

```
private func layer(for text: String) -> CALayer? {
  // Create an empty array of Unicode characters
  var unichars = [UniChar](text.utf16)

  // Create an empty array of glyphs.
  var glyphs = [CGGlyph](repeating: 0, count: unichars.count)

    // Put the glyphs into the glyphs variable.
    guard CTFontGetGlyphsForCharacters(font, &unichars, &glyphs, unichars.count) else {
      return nil
    }

  // Get the bounding box of the reference character, 'l' is our case.
  let strokeWidthReferenceCharacterBoundingBox = self.boundingBox(for: strokeWidthReferenceCharacter, using: font)

    let strokeWidth: CGFloat

    switch self.strokeWidth {
      case .absolute(value: let value):
                            strokeWidth = value
      case .relative(scale: let scale):
                            strokeWidth = strokeWidthReferenceCharacterBoundingBox.width * scale
    }

  let layers: [CALayer] = glyphs.flatMap({ glyph in
      guard let path = CTFontCreatePathForGlyph(font, glyph, nil) else {
      return nil
      }

      let layer = CAShapeLayer()
      layer.path = path
      layer.bounds = path.boundingBox
      layer.isGeometryFlipped = true
      layer.strokeColor = strokeColor.cgColor
      layer.fillColor = UIColor.clear.cgColor
      layer.lineWidth = strokeWidth

      return layer
      })

  let groupLayer = CAShapeLayer()

    let letterHeight = strokeWidthReferenceCharacterBoundingBox.height

    var offset = CGFloat(0)

    let spacingReferenceCharacterBoundingBox = self.boundingBox(for: spacingReferenceCharacter, using: font)

    for layer in layers {
      let yOrigin = letterHeight - layer.bounds.height
        layer.frame.origin = CGPoint(x: offset, y: yOrigin)

        let characterSpacing: CGFloat

        switch self.characterSpacing {
          case .absolute(value: let value):
                                characterSpacing = value
          case .relative(scale: let scale):
                                characterSpacing = spacingReferenceCharacterBoundingBox.width * scale
        }

      offset = layer.frame.maxX + characterSpacing
        groupLayer.addSublayer(layer)
    }

  groupLayer.bounds = CGRect(x: layers.first!.frame.origin.x, y: layers.first!.frame.origin.y, width: layers.last!.frame.maxX, height: layers.first!.frame.size.height)

    groupLayer.frame.origin = CGPoint(x: 0, y: 0)

    return groupLayer
}
```
