# Clinkle Objective-C Style Guide

At Clinkle, when our C purists friends or our LISP Purist friends say “Objective-C looks like documentation, not code,” we say “That’s a feature, not a bug!”

We belive you know you’ve discovered a good codebase when that you can’t tell which engineer wrote a particular file (until you use `git blame`, if you must).

We take no credit for the inspiration behind the meat of the guide. That goes to the team at [The New York Times](https://github.com/NYTimes/objective-c-style-guide).  We hope you agree with some of the modifcations we’ve made for our own purposes.

## Table of Contents

* [Dot-Notation Syntax](#dot-notation-syntax)
* [Spacing](#spacing)
* [Control Flow](#control-flow)
  * [Ternary Operator](#ternary-operator)
  * [Error handling](#error-handling)
* [Methods](#methods)
* [Variables](#variables)
  * [Properties vs. instance variables](#properties-vs-instance-variables)
  * [Private Properties](#private-properties)
* [Naming](#naming)
  * [Images](#images)
  * [CocoaPods and Open-Source](#cocoapods-and-open-source)
  * [Categories](#categories)
* [Object Lifecycle](#object-lifecycle)
  * [init and dealloc](#init-and-dealloc)
  * [Singletons](#singletons)
* [Constants](#constants)
* [Literals](#literals)
* [Enumerated Types](#enumerated-types)
* [Comments](#comments)
* [Booleans](#booleans)
* [Xcode project](#xcode-project)



## Dot-Notation Syntax

Dot-notation should **always** be used for accessing and mutating properties. Bracket notation is preferred in all other instances.

**For example:**
```objc
view.backgroundColor = [UIColor orangeColor];
[UIApplication sharedApplication].delegate;
```

**Not:**
```objc
[view setBackgroundColor:[UIColor orangeColor]];
UIApplication.sharedApplication.delegate;
```

## Spacing and newlines

We believe vertical whitespace is your friend. It majorly improves code readability.  Therefore:

* Indent using 4 spaces. Never indent with tabs. Be sure to set this preference in Xcode.
* All control flow braces (`if`/`else`/`switch`/`while`/`typedef` etc.) and method braces always open on a newline and close on another newline.

**For example:**
```objc
if (user.isHappy)
{
    // Do something
}
else
{
    // Do something else
}
```

* There should be exactly one blank line between methods to aid in visual clarity and organization. Whitespace within methods should separate functionality, but often there should probably be new methods.
* `@synthesize` and `@dynamic` should each be declared on their own new lines in the implementation.

## Control Flow

Conditional bodies should always use braces even when a conditional body could be written without braces (e.g., it is one line only) to prevent [errors](https://github.com/NYTimes/objective-c-style-guide/issues/26#issuecomment-22074256). These errors include adding a second line and expecting it to be part of the if-statement. Another [even more dangerous defect](http://programmers.stackexchange.com/a/16530) may happen where the line “inside” the if-statement is commented out, and the next line unwittingly becomes part of the if-statement. In addition, this style is more consistent with all other conditionals, and therefore more easily scannable.

**For example:**
```objc
if (!error)
{
    return success;
}
```

**Not:**
```objc
if (!error)
    return success;
```

or

```objc
if (!error) return success;
```

This bracket-with-newlines pattern also applies to `for()` and `while()` loops and other bracket-delimited control flows.

### Ternary Operator

The Ternary operator `?` should be used when it increases clarity or code neatness. A single condition is usually all that should be evaluated. Evaluating multiple conditions is usually more understandable as an if statement, or refactored into a seperate method.

**For example:**
```objc
result = a > b ? x : y;
```

**Not:**
```objc
result = a > b ? x = c > d ? c : d : y;
```

### Error handling

When methods return an error parameter by reference, switch on the returned value, not the error variable.

**For example:**
```objc
NSError *error;
if (![self trySomethingWithError:&error])
{
    // Handle Error
}
```

**Not:**
```objc
NSError *error;
[self trySomethingWithError:&error];
if (error)
{
    // Handle Error
}
```

Some of Apple’s APIs write garbage values to the error parameter (if non-NULL) in successful cases, so switching on the error can cause false negatives (and subsequently crash).

## Methods

In method signatures, there should be a space after the scope (-/+ symbol). There should be a newline between the method segments.  The colons should stack.

**For Example**:
```objc
- (void)setExampleText:(NSString *)text 
                 image:(UIImage *)image;
```

## Variables

Variables should be named as descriptively as possible. Single letter variable names should be avoided except in `for()` loops.

Asterisks indicating pointers belong with the variable, e.g., `NSString *text` not `NSString* text` or `NSString * text`.

[Ownership-qualifying compiler directives](http://clang.llvm.org/docs/AutomaticReferenceCounting.html#ownership-qualification) like `__weak` put the directive before the normal type declaration.  This is not consistent with Apple’s guidelines, but we find it makes refactoring or searching with regexes more consistent.

```objc
__weak typeof(self) weakSelf = self;
__weak NSString *cowPie = self.cow.pie;
```

`const` declarations behave similarly.  See [Constants](#constants).

```objc
extern const NSUInteger kNumSecondsPerYear;
static const NSString *kMothersMaidenName = @"TapatÃ­o";
```

### Properties vs. instance variables

Property definitions should be used in place of naked instance variables, exclusively. Accessing properties by their generated underscore-prefixed instance variable should be avoided except in custom setters and getters.

Properties and local variables should be camel-case with the leading word being lowercase. The property attribute `nonatomic` always precedes memory-management attributes like `strong` or `copy`. Property attributes should always be explicitly stated, never implied

**For example:**

```objc
@interface AntfarmController: NSObject

@property (nonatomic, copy) NSString *queenAnt;

@end
```

**Not:**

```objc
@interface AntfarmController : NSObject
{
    NSString *queenAnt;
}
```

### Private Properties

Private properties should be declared in class extensions (anonymous categories) in the implementation file of a class. Named categories (such as `NYTPrivate` or `private`) should never be used unless extending another class.

**For example:**

```objc
@interface NYTAdvertisement ()

@property (nonatomic, strong) GADBannerView *googleAdView;
@property (nonatomic, strong) ADBannerView *iAdView;
@property (nonatomic, strong) UIWebView *adXWebView;

@end
```

## Naming

Apple naming conventions should be adhered to wherever possible, especially those related to [memory management rules](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html) ([NARC](http://stackoverflow.com/a/2865194/340508)).

Long, descriptive method and variable names are your friend.

**For example:**

```objc
UIButton *settingsButton;
BOOL hasFinishedParsingRemoteAsset;
```

**Not**

```objc
UIButton *setBut;
BOOL done;
```

### Images

Images should be named consistently using big-endianness (most generic component first) to preserve organization and developer sanity. They should be named as one camel case string starting with a lowercase letter.

**For example:**

* `refreshBarButtonItemDefault` / `refreshBarButtonItemDefault@2x`
* `refreshBarButtonItemActive` / `refreshBarButtonItemActive@2x`

Images that are used for a similar purpose should be grouped in respective groups in a `/resources/images` folder.

### CocoaPods and Open-Source

A three-letter prefix (e.g. `CLK`) should be used for any code in the codebase that has the potential to be open-sourced.  It’s *not* necessary to use the prefix for standard application-specific classes.  Utilities classes or other code that could easily and cleanly be pulled out of the codebase and used independently on other projects should use the prefix.  The rule of thumb is “Would we ever want to open-source this class in the future?”  If so, then three-letter-prefix the class.  If there are a number of related similar classes that themselves belong in a logical unit, break them out into their own local [CocoaPod](http://cocoapods.org/).

### Categories

All categories should be three-letter prefixed, and all methods and properties (both public and private) should prefix themselves with `clk_` or equivalent.  For example:

```objc
- (void)clk_createCompanyCulture;

@property (nonatomic, assign) BOOL clk_cornerRadius;

```

## Object Lifecycle

### init and dealloc

`dealloc` methods should be placed at the top of the implementation, directly after the `@synthesize` and `@dynamic` statements. `init` should be placed directly below the `dealloc` methods of any class.

`init` methods should be structured like this:

```objc
- (instancetype)init
{
    self = [super init]; // or call the designated initalizer
    if (self)
    {
        // Custom initialization
    }

    return self;
}
```

### Singletons

Singleton objects should use a thread-safe pattern for creating their shared instance.
```objc
+ (instancetype)sharedInstance
{
   static id sharedInstance = nil;

   static dispatch_once_t onceToken;
   dispatch_once(&onceToken, ^{
      sharedInstance = [[self alloc] init];
   });

   return sharedInstance;
}
```
This will prevent [possible and sometimes prolific crashes](http://cocoasamurai.blogspot.com/2011/04/singletons-your-doing-them-wrong.html).

## Constants

Constants are preferred over in-line string literals or numbers, as they allow for easy reproduction of commonly used variables and can be quickly changed without the need for find and replace. Constants should be declared as `static` constants and not `#define`s unless explicitly being used as a macro.

Constants should be camel-case with all words capitalized and prefixed with a `k` and then your three-letter prefix.  Big-endianness in naming constants is preferred, to allow for easy grouping using global search and for ordering by logical siblings.

**For example:**

```objc
static const NSString *kCLKHeartTransplantSurgeonLastName = @"Wellington";

static const NSInteger kCLKFrogAppendageLengthRatio = 4.7;
```

**Not:**

```objc
#define SurgeonName @"Wellington"

#define FROG_LEG_LENGTH 4.7
```

## Literals

`NSString`, `NSDictionary`, `NSArray`, and `NSNumber` literals should be used whenever creating immutable instances of those objects. Pay special care that `nil` values not be passed into `NSArray` and `NSDictionary` literals, as this will cause a crash.

**For example:**

```objc
NSArray *names = @[@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul"];
NSDictionary *productManagers = @{@"iPhone" : @"Kate", @"iPad" : @"Kamal", @"Mobile Web" : @"Bill"};
NSNumber *shouldUseLiterals = @YES;
NSNumber *buildingZIPCode = @10018;
```

**Not:**

```objc
NSArray *names = [NSArray arrayWithObjects:@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul", nil];
NSDictionary *productManagers = [NSDictionary dictionaryWithObjectsAndKeys: @"Kate", @"iPhone", @"Kamal", @"iPad", @"Bill", @"Mobile Web", nil];
NSNumber *shouldUseLiterals = [NSNumber numberWithBool:YES];
NSNumber *buildingZIPCode = [NSNumber numberWithInteger:10018];
```

## Enumerated Types

When using `enum`s, it is recommended to use the new fixed underlying type specification because it has stronger type checking and code completion. The SDK now includes a macro to facilitate and encourage use of fixed underlying types â€” `NS_ENUM()`

**Example:**

```objc
typedef NS_ENUM(NSInteger, AccountType)
{
  kAccountTypeChecking,
  kAccountTypeSavings,
  kAccountTypeTaxDeferred,
  kAccountTypeRetirement,
  kAccountTypeCorporate
};
```

**not:**

```objc
typedef enum AccountType
{
  kChecking,
  kSavings,
  kTaxDeferred,
  kRetirement,
  kCorporate
} AccountType;
```

## Comments

When they are needed, comments should be used to explain **why** a particular piece of code does something. Any comments that are used must be kept up-to-date or deleted.

Block comments should generally be avoided, as code should be as self-documenting as possible, with only the need for intermittent, few-line explanations. This does not apply to those comments used to generate documentation.

## Booleans

Since `nil` resolves to `NO` it is unnecessary to compare it in conditions. Never compare something directly to `YES`, because `YES` is defined to 1 and a `BOOL` can be up to 8 bits.

This allows for more consistency across files and greater visual clarity.

For more complex boolean operators (combinations of `||` and `&&`), each individual operation has its own line within the `if` braces.

**For example:**

```objc
if (!someObject)
{
    
} 
else if (usersTShirtIsFunny
         || (userHasSeenBeardedWoman &&
             beardedWomanDidWinBlueRibbon))
{
    
} 
else if (![userIsPlayingBadminton boolValue])
{
    
}

```

**Not:**

```objc
if (someObject == nil)
{

}
else if (usersTShirtIsFunny == YES || (userHasSeenBeardedWoman && beardedWomanDidWinBlueRibbon)) // Never do this.
{
    
}
else if ([userIsPlayingBadminton boolValue] == NO) // why u no use your brain?
{
    
}
```

If the name of a `BOOL` property is expressed as an adjective, the property can omit the `is` prefix, but should specify the `is`-prefixed name for the getter. For example:

```objc
@property (nonatomic, assign, getter=isEditable) BOOL editable;
```

Text and example taken from the [Cocoa Naming Guidelines](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingIvarsAndTypes.html#//apple_ref/doc/uid/20001284-BAJGIIJE).

## Xcode project

The physical files should be kept in sync with the Xcode project files in order to avoid file sprawl. Any Xcode groups created should be reflected by folders in the filesystem. Code should be grouped not only by type, but also by feature for greater clarity.

When possible, always turn on “Treat Warnings as Errors” in the target’s Build Settings and enable as many [additional warnings](http://boredzo.org/blog/archives/2009-11-07/warnings) as possible. If you need to ignore a specific warning, use [Clang’s pragma feature](http://clang.llvm.org/docs/UsersManual.html#controlling-diagnostics-via-pragmas).

# Other Objective-C Style Guides

If ours doesn’t fit your tastes, have a look at some other style guides:

* [Google](http://google-styleguide.googlecode.com/svn/trunk/objcguide.xml)
* [GitHub](https://github.com/github/objective-c-conventions)
* [Adium](https://trac.adium.im/wiki/CodingStyle)
* [Sam Soffes](https://gist.github.com/soffes/812796)
* [CocoaDevCentral](http://cocoadevcentral.com/articles/000082.php)
* [Luke Redpath](http://lukeredpath.co.uk/blog/my-objective-c-style-guide.html)
* [Marcus Zarra](http://www.cimgf.com/zds-code-style-guide/)
