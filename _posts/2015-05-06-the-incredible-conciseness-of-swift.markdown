---
layout: post
title:  "The incredible conciseness of Swift"
date:   2015-05-06 23:24:41
categories: []
tags: [ iOS, swift, apple, programming ]
image: "/images/swift/swift-og.png"
---

This post shows how little code is needed to write a closure declaration and call in the new Apple's Swift compared to the old Objective-C

![Swift Logo](/images/swift/swift-og.png)

First of all, I need to show you how it used to be made in the old way.
It is just a variable declaration of a function type with two doubles as input and another double as output, an attribution and a function call.


{% highlight objective-c %}

//function variable declaration
double (^myFunction)(double, double);

//function variable attribution
myFunction = ^(double firstValue, double secondValue) {
   return firstValue * secondValue;
};

//function call
double x = myFunction(3,4);

{% endhighlight %}

It is a relatively simple code, right? Lets do the same thing in Swift and evolve to a more concise usage.

{% highlight swift %}
//function variable declaration
var myFunction: (Double, Double) -> Double

//function variable attribution
myFunction = {(firstValue: Double, secondValue: Double) in
    return firstValue * secondValue
}

//function call
var x: Double = myFunction(3,4)

{% endhighlight %}

As you can see, there is nothing new, and it is even more verbose than before.
However, if you look closer you'll realize that we don't need so much code, since Swift relies on its strongly typed syntax and its type inference.

But what is type inference? It is the language capability of guessing the type of a variable looking into its first attribution.

For example:
```var b: Boolean = true``` can be written as ```var b = true``` and the variable ```b``` will be a Boolean variable.
It is really important to notice that it is not a dynamic language, that means ```b``` will always be Boolean.

As we already know that, we can take the type from the attribution and the function call:

{% highlight swift %}
var myFunction: (Double, Double) -> Double

//We can get rid of the Double type here as soon as it doesn't need
myFunction = {(firstValue, secondValue) in
    return firstValue * secondValue
}
//As well as here. The compiler knows that the function returns a Double value
var x = myFunction(3,4)
{% endhighlight %}

In fact we don't even need the keyword ```return``` as the compiler infers that the expression must be returned.

{% highlight swift %}
myFunction = {(firstValue, secondValue) in
    firstValue * secondValue
}
{% endhighlight %}

It can be way shorter if we use anonymous parameters. To use anonymous parameters, just name than $0 to the first parameter, $1 to the second and so on.

The last version of the shortened code becomes like this:

{% highlight swift %}

var myFunction: (Double, Double) -> Double

myFunction = { $0 * $1 }

var x = myFunction(3,4)

{% endhighlight %}

Less code, more productivity, fewer bugs. It gives an idea of the language capabilities and the power of type inference in modern languages.
