# iOS Library Prefer Nonatomic Properties
* Proposal: [SDL-0009](0009-ios-library-prefer-nonatomic.md)
* Author: [Joel Fischer](https://github.com/joeljfischer)
* Status: **In review**
* Impacted Platforms: iOS

## Introduction
This proposal is to change the atomicity of most if not all properties in the iOS library currently labelled 'atomic' or unlabelled (which defaults to 'atomic').

## Motivation
Atomic properties work well in certain situations regarding threading but are by default much slower because they enforce locks around getting and setting the property. Nonatomic does not enforce these locks and is therefore significantly faster at accessing the property data than an atomic property. With nonatomic properties, you must be more careful to not attempt to set the property while setting or getting it on another thread, but for nearly all, if not all cases, this is not an issue for us. Furthermore, having properties atomic *does not* give us thread safety. For example, if we set atomic properties `firstName` and `lastName` from one thread while getting them from another, we are guaranteed those gets will succeed, but we are *not* guaranteed they are correct relative to each other. Therefore, it is better to create actually thread safe objects with nonatomic properties if necessary than to simply rely on atomic properties.

Finally, although Apple initially made the decision that Obj-C properties are atomic by default, since then they have repeatedly said that properties should be nonatomic by default. We can see this decision played out in Swift, where all properties are nonatomic by default. I'm not even aware of an option to make properties atomic. This is likely because of the above described scenario. It can be easy to think you have made an object thread-safe through atomic properties when, in fact, you have not.

To reiterate, nonatomic properties are significantly faster than atomic ones, and we should change all, or as near as possible, of our atomic or unlabelled properties to nonatomic.

## Proposed Solution
The proposed solution is to audit the iOS library and change all atomic or unlabelled properties to be nonatomic. For example, all RPC properties are currently labelled atomic.

## Potential Downsides
The only potential downside would be if somewhere in the library we are relying on the atomicity of some properties at some point. This seems pretty unlikely to me as we are not using many queues or threads, and I have never needed atomicity in any Obj-C code I have ever written.

## Impact on existing code
This would not be a breaking change unless some code needed to be redesigned, which seems very unlikely. The true impact would be a significant increase in the speed of our code.

## Alternatives considered
There are no alternatives.
