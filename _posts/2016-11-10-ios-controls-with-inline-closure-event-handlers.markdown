---
layout: post
title:  "iOS standard UI controls with inline event handler"
date:   2016-11-10 09:00:00
categories: []
tags: [ iOS, Functional, UIKit, Closure]
#image: "/images/2016/11/testbuildfailed.png"
comments: true
published: true
---


Simplicity is a key factor in software development. If you are an experienced developer, you kind of feel when something is wrong if many steps are required to perform a simple action.
Do we really need to create a delegate or a method and `addTarget` to simply handle a UI event?

Then I recently became interested in functional programming and the natural way to go functional in Objective-C was using ReactiveCocoa library. I remember the first time when I used it; I got completely amazed about the way it worked and how powerful it was. Additionally, it solved the UI control event problem by just using `rac_signalForControlEvents` and attach a code block.

Then Swift was out, which included the map/filter/reduce, and I was using ReactiveCocoa mainly to handle UI events. So I realized that it was an overkill.

To solve this I created this helper library that allows me to focus on the important part of my apps and quickly add code to UI events.

See how simple it is to add some code to a UI event:

```swift
// Many ways to do this
barButton.onClick {(barBtn: UIBarButtonItem) in
    print("Clicked");
};

button.onClick { (btn) in
    print("Button clicked");
}

swt.onChange(handler: { (ctrl : UISwitch) in
    print("swt changed");
})

slider.onChange { (sld) in
    print("slider change to \(sld.value)");
}

stepper.onChange { (obj) in
    print("stepper change to \(obj.value)");
}

segControll.onChange { (obj) in
    print("seg change to \(obj.selectedSegmentIndex)");
}

```

It currently supports the following controls:
* UIBarButtonItem
* UIButton
* UISegmentedControl
* UIStepper
* UISlider
* UISwitch
* UIDatePicker

It is available in cocoapods:

`pod "UIKevent"`
