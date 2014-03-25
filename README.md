##SOApplicationDelegate

> A service-oriented application delegate for use in iOS applications.


##### Why?
A few weeks ago I read [this](http://sizeof.io/2014/02/08/service-oriented-appdelegate/) post about service-oriented application delegates, and while I really liked the pattern, I was not a fan of the implementation.  Appdelegates are one of the most abused pieces of objective-C and tend to get crammed with tons of code, so abstracting that out into more reusable, testable classes is fantastic.  However, manually implementing and then re-calling each `UIApplicationDelegate` method on each service did not seem like a very elegant solution.  Thankfully objective-c has some decent runtime methods we can use to improve on this.


##### How
When a delegate method is to be called, the calling object always first checks if the delegate responds to that method.  By overriding `- (BOOL)respondsToSelector:(SEL)aSelector` we can tell the calling class that we respond to selectors that we don't actually implement.  Generally this would cause problems as you need to implement each method an object responds to, however `NSObject` also has the method `- (void)forwardInvocation:(NSInvocation *)anInvocation` which gets called every time an unknown selector is invoked on an object.  In this method we're tasked with forwarding the invocation of a method to a target that can receive it, or erroring.

By overriding `- (void)forwardInvocation:(NSInvocation *)anInvocation` we can now use it to execute the given method invocation on each of our service objects without actually implementing *any* of the delegate methods.

##### The services
Since each service should only be registered with the AppDelegate once, we need to rely on methods that return singleton objects so that only one instance will ever be created.  Commonly this is done by creating a `shared` or `sharedInstance` method on the objects, but in a situation where you may have 10+ services repeating all that code seems to be a waste.  

Creating a parent class for all the services is the obvious next step in improving things, but this now presents a tricky dillema.  Any static `dispatch_once_t` predicate defined by a parent class will be shared by the child classes, meaning that `shared` will only ever be executed once across any of the interiting objects and we'll only ever get one registered service.

To solve this problem I've created a singleton dictionary of dispatch predicates (actually it's a dictionary of NSNumbers holding the predicates memory addresses, because NSDictionary only supports NSObject instances), along with a singleton dictionary of class instances.  These dictionaries are keyed by the current class name ( `NSStringFromClass([self class])` ) and are used to tell which `dispatch_once` blocks have been executed for each class.


---

I've had this implementation being used on a project current in development for a couple weeks now and I'm very happy with it.  The app delegate is less than 50 lines, many of which are comments, and each services lives in their own easily testable, single-responsibility class.
