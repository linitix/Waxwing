[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)
[![Version](https://img.shields.io/cocoapods/v/Waxwing.svg?style=flat)](http://cocoapods.org/pods/Waxwing)
[![License](https://img.shields.io/cocoapods/l/Waxwing.svg?style=flat)](http://cocoapods.org/pods/Waxwing)
[![Platform](https://img.shields.io/cocoapods/p/Waxwing.svg?style=flat)](http://cocoapods.org/pods/Waxwing)
[![Build Status](https://travis-ci.org/JanGorman/Waxwing.svg?branch=master)](https://travis-ci.org/JanGorman/Waxwing)

# Waxwing

iOS version migrations in Swift. When mangling data or performing any other kind of updates you want to ensure that all relevant migrations are run in order and only once. Waxwing allows you to do just that.

## Requirements

* iOS 8+
* Swift 1.2, 2, 2.2

## Installation

Waxwing is available through [CocoaPods](http://cocoapods.org). To install
it, simply add the following line to your Podfile:

    pod "Waxwing"
    
If you don't like CocoaPods, you can also add the dependency via [Carthage](https://github.com/Carthage/Carthage) or simply include `Waxwing.swift` in your project.

## Usage

There are two ways to run your migrations, either with closures or through an `NSOperationQueue`:

``` swift
import Waxwing

…

let waxwing = Waxwing(bundle: NSBundle.mainBundle(), defaults: NSUserDefaults.standardUserDefaults())

waxwing.migrateToVersion("0.9") {
	firstMigrationCall()
	secondMigrationCall()
	…
}
```

or

``` swift
import Waxwing

…

let waxwing = Waxwing(bundle: NSBundle.mainBundle(), defaults: NSUserDefaults.standardUserDefaults())

Waxwing.migrateToVersion("0.9", [FirstMigrationClass(), SecondMigrationClass()])
```

Note that closure based migrations are run from the thread they are created on. Anything that has to run on the main thread, such as notifying your users of changes introduced with this version, needs to explictly call the method on the main thread:

``` swift
import Waxwing

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)) {
	let waxwing = Waxwing(bundle: NSBundle.mainBundle(), defaults: NSUserDefaults.standardUserDefaults())
	waxwing.migrateToVersion("0.9") {
		someMigrationFunction()
		
		dispatch_async(dispatch_get_main_queue()) {
			// Some alert that we're done updating / what's new in this version of the app
		}
	}
}
```

The `NSOperationQueue` based migrations are always run from their own queue so the same caveat applies. Also note, that if any of the migrations in the queue depend on another one having run first, you explicitly need to add that dependency. `NSOperation` of course makes this trivial through the `addDependency()` method.

You can add as many migrations as you want. They will always be executed once which makes reasoning about the state of your application a lot easier.

## NSProgress

Waxwing has built in support for [NSProgress](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSProgress_Class/index.html). Since the number of actions that are run using the closure based method cannot be determined it just reports a total unit count of 1. If you're using NSOperations, the unit count will match the number of migrations.

```swift
import Waxwing

func migrate() {
	let progress = NSProgress(totalUnitCount: 1)
	progress.becomeCurrentWithPendingUnitCount(1)
	progress.addObserver(self, forKeyPath: "fractionCompleted", options: .New, context: observerContext))
	
	waxwing.migrateToVersion("0.8", migrations: [migration1, migration2, migration3…])
	
	progress.resignCurrent()
}

override func observeValueForKeyPath(keyPath: String?, ofObject object: AnyObject?, change: [String : AnyObject]?, context: UnsafeMutablePointer<Void>) {
  if let progress = object as? NSProgress where context == observerContext && keyPath == "fractionCompleted" {
    // e.g. Update progress indicator, remember to do this on the main thread
  } else {
    super.observeValueForKeyPath(keyPath, ofObject: object, change: change, context: context)
  }
}
```

For more information on how `NSProgress` works I recommend [this article](http://oleb.net/blog/2014/03/nsprogress/) by Ole Begemann.

## Author

Jan Gorman, gorman.jan@gmail.com

## License

Waxwing is available under the MIT license. See the LICENSE file for more info.

