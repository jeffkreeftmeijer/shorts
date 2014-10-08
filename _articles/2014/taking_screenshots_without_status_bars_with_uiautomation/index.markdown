I looked into automatically taking App Store screenshots this week.

To get [the simplest thing that would work](https://gist.github.com/jeffkreeftmeijer/962170d4b533e5c5e3d5), I distilled the essential parts from [ui-screen-shooter]( https://github.com/jonathanpenn/ui-screen-shooter) and ended up with a Ruby file that spawns simulators in different device languages and some javascript that uses [UIAutomation](https://developer.apple.com/library/ios/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/UsingtheAutomationInstrument/UsingtheAutomationInstrument.html) to run through the application and take the actual screenshots.

Instead of just taking normal screenshots, it also takes care of removing the status bar, so the images can be dropped directly into iTunes Connect. 
It turned out that was quite simple to do with `captureRectWithName()`:

``` js
target = UIATarget.localTarget()
rectWithoutStatusBar = target.rect()
rectWithoutStatusBar.origin.y = 20.0
target.captureRectWithName(rectWithoutStatusBar, 'home_without_status_bar')
```

Remember: To take 40 pixels off the top of the screenshot image [to get the minimum screenshot size](https://developer.apple.com/library/ios/documentation/LanguagesUtilities/Conceptual/iTunesConnect_Guide/Appendices/Properties.html), you set the y-origin to 20.0 points.
