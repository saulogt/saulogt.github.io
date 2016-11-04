---
layout: post
title:  "Error on build test target while using cocoapods"
date:   2016-10-27 09:00:00
categories: []
tags: [ iOS, XCode, XCTest, Cocoapods, Firebase, Snapshot ]
image: "/images/2016/10/testbuildfailed.png"
comments: true
published: true
---

I was trying to automate the screenshots of my app in order to send them to the itunes connect, but I got stuck in test target build, which is not directly related.
I choose to use the fantastic tool Snapshot that is part of the Fastlane toolbox. It can take the screenshots in an automatic way using the UITest introduced in XCode 7. It is definitely not easy to configure, but it can save a lot of time on the long run, especially if you support many languages.

Well, the problem is that the snapshot tool runs the regular test before running the uitest and here is where the error happened.

When the error happened I run only the test target and this was the result:

![Error](/images/2016/10/testbuildfailed.png)

Error = Missing required module 'Firebase'

Due to the combination of factors it just doesn't build the test target. The combination is:

* XCTest
* Cocoapods
* Firebase library

The Google Firebase does something special in the include and library paths so the Cocoapods is not able to inherit the configuration correctly.

The workaround, as described by Jason in stackoverflow (link bellow), was to replace the content of `MyAppTests.<configuration>.xcconfig` with the content of `MyApp.<configuration>.xcconfig`.

<http://stackoverflow.com/questions/38216090/xcode-unit-testing-with-cocoapods#40209662>

I'm not sure how long it will still work, but it solved the problem for now.

My next task is to do this automatically in the Podfile, so that it can run every time I do `pod install`

here is my Podfile with the dirty solution:

{% highlight ruby %}

# Uncomment this line to define a global platform for your project
# platform :ios, '9.0'

target 'personal_budget' do
  # Comment this line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for personal budget
  pod 'Firebase'
  pod 'Firebase/Auth'
  pod 'Firebase/Core'
  pod 'Firebase/Database'

  target 'personal_budgetTests' do
    inherit! :search_paths
    # Pods for testing
  end

  target 'personal_budgetUITests' do
    inherit! :search_paths
    # Pods for testing
  end

end

# The workaround starts here !!!!!
targetWorkaround = "Pods-personal_budgetTests"

post_install do |installer|
    installer.pods_project.targets.each do |target|
        puts "#{target.name}"
        if target.name == targetWorkaround
            puts "Replacing!!!"

            puts `cp "./Pods/Target Support Files/Pods-personal_budget/Pods-personal_budget.debug.xcconfig" "./Pods/Target Support Files/Pods-personal_budgetTests/Pods-personal_budgetTests.debug.xcconfig"`
            puts `cp "./Pods/Target Support Files/Pods-personal_budget/Pods-personal_budget.release.xcconfig" "./Pods/Target Support Files/Pods-personal_budgetTests/Pods-personal_budgetTests.release.xcconfig"`

        end
    end
end

{% endhighlight %}
