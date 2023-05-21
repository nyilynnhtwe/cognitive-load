# Cognitive Load Developer's Handbook 🧠

## Why bother reading?
We spend much more time reading and understanding code than writing it. Therefore, the amount of cognitive load we need to build in our brains in order to understand the code is crucial.

This paper discusses a fundamental thinking tool that would enable us to create simple yet maintainable applications. We would look at every decision, every trendy buzzword, and every fancy technology through the lens of cognitive load.

## Cognitive load
> **Cognitive load is how much a developer needs to know in order to complete a task.**

We should strive to reduce the cognitive load in our projects as much as possible.

The average person can hold about *four facts* in working memory. . Once the cognitive load reaches this threshold, a significant effort is required to understand things.

*Let's say we've been asked to make some fixes to a completely unfamiliar project. We were told that a really smart developer had contributed to it. Lots of cool technologies, fancy libraries and trendy frameworks were used. In other words, **the previous author had a high cognitive load in his head, which we have yet to recreate.***

![Cognitive Load](/img/cognitiveload.png)

## Types of cognitive load
**Intrinsic** - is the inherent level of difficulty associated with a specific problem we're solving. It can't be reduced, it's at the very heart of software development.  

**Extraneous** - is generated by the manner in which information is presented, is imposed by factors not directly relevant to the problem we are trying to solve. Can be greatly reduced. We'll focus on this type of cognitive load.

Let's jump straight to the concrete practical examples of extraneous cognitive load.

*P.S. Contributions are welcome! Feel free to send PRs with your own examples.*

----

> **Note**
> We will refer to the level cognitive load as follows:  
> `🧠`: fresh working memory, zero cognitive load  
> `🧠++`: two facts in our working memory, cognitive load increased  
> `🤯`: working memory overflow, more than 4 facts  

## Inheritance nightmare
We're tasked to change a few things for our admin users: `🧠`

`AdminController extends UserController extends GuestController extends BaseController`

Ohh, part of the functionality lies in `BaseController`, let's have a look: `🧠+`  
Basic role mechanics got introduced in `GuestController`: `🧠++`  
Things got partially altered in `UserController`: `🧠+++`  
Finally we're here, `AdminController`, let's code stuff! `🧠++++`  

Oh, wait, there's `SuperuserController` extending `AdminController`. By modifying `AdminController` we can break things in the inherited class, so let's dive in `SuperuserController` first: `🤯`

Prefer composition over inheritance. We won't go into the details - there are enough articles on the topic.

## Too many small methods, classes or modules
> **Note**
> Method, class and module are interchangeable in this context 
 
Mantras like "methods should be shorter than 15 lines of code" or "classes should be small" turned out to be somewhat wrong.

**Shallow module** - complex interface, simple functionality  
**Deep module** - simple interface, complex functionality

![Deep module](/img/deepmodulev2.png)

Having too many shallow modules can make it difficult understand the project. **Not only we have to keep in mind each module responsibilities, but also all their interactions**. `🤯`

> Information hiding is paramount, and we don't hide as much complexity in shallow modules.

I have two pet projects, both of them are somewhat 5K lines of code. The first one has 80 shallow classes, whereas the second one has only 7 deep classes. I haven't been maintaining any of these projects for one year and a half.

Once I came back, I realized that it is tremendously hard to untangle all those interactions between these 80 classes in the first project. I would have to rebuild an enormous amount of cognitive load before start coding. On the other hand, I was able to quickly grasp the second project, because it only had a few deep classes with simple interface.

> **The best components are those that provide powerful functionality yet have simple interface.**  
John K. Ousterhout

> **Note**
> The greatest book on the topic is "A Philosophy of Software Design". Not only it covers the very essence of complexity, but it also has so far the greatest interpretation of Parnas' influential paper "On the Criteria To Be Used in Decomposing Systems into Modules".

## Too many shallow microservices
We can apply the aforementioned scale-agnostic principle to microservice architecture as well. Too many shallow microservices won't do any good - the industry is heading towards somewhat "macroservices", i.e., services that aren't that shallow. One of the worst and most difficult-to-fix phenomena is so-called distributed monolith, which is often the result of this overly granular shallow separation.

I once consulted a startup where a team of four developers introduced 17(!) microservices. They were 10 months behind schedule and appeared nowhere close to the public release. Every new requirement led to changes across 4+ microservices. TTM was unacceptably low. Cognitive load was unbearably high. `🤯`  

Is it the appropriate way to approach the uncertainty of a new system? The only team's justification was: "FAANG companies proved microservice architecture to be effective".

A well-crafted monolith with truly isolated modules is often much more convenient and flexible than a bunch of microservices. It's only when the need for separate deployments becomes crucial (e.g. development team scaling) that you should consider adding a network layer between our modules (future microservices).

## Featureful languages
We feel excited when new features got released in our favourite language. We spend some time learning these features, we build code upon them.

If there are lots of features, we may spend half an hour playing with a few lines of code, to use one or another feature. And it's kinda waste of time. But what's worse, **when you come back later, you would have to recreate that thought process!** `🤯`

**You not only have to understand this complicated program, you have to understand why a programmer decided this was the way to approach a problem from the features that are available.**
Those statements are made by none other than Rob Pike.

## Business logic and HTTP status codes
On the backend we return:
```go
401 // for expired jwt token
403 // for not enough access
418 // for banned users
```

Guys on the frontend use backend API to implement login functionality. They would have to temporary create the following cognitive load in their brains:
```go
401 is for expired jwt token // 🧠+, ok just temporary remember it
403 is for not enough access // 🧠++
418 is for banned users // 🧠+++
```
Frontend devs would introduce (hopefully) variables/functions like `isTokenExpired(status)`, so that the following generations of developers wouldn't have to recreate this kind of `status -> meaning` mapping in their brains.

Then QA folks come in play:
"Hey, I got `403` status, is that expired token or not enough access?"
**QA people can't jump straight to testing, because first they have to recreate the cognitive load guys on the backend once created.**

It's better to abstract away your business details from the HTTP transfer protocol, and return status codes right in the response body:
```json
{
    "code": "jwt_has_expired"
}
```

Cognitive load on the frontend side: `🧠` (fresh, no facts are held in mind)  
Cognitive load on the QA side: `🧠`

> As for following that mystical "RESTful API" and using all sorts of HTTP verbs and statuses, the standard simply doesn't exist. The only valid document on this matter is a thesis published by Roy Fielding, dated back in 2000, and it says nothing about verbs and statuses. People go along with very few basic HTTP statuses and POSTs only, and they're doing just fine.*

## Complicated if statements
```go
if val > someConstant && // 🧠+
    (condition2 || condition3) && // 🧠+++, prev cond should be true, one of c2 or c3 has be true
    (condition4 && !condition5) { // 🤯, we're messed up here
    ...
}
```

Introducing temporary variables with meaningful names:
```go
isValid = var > someConstant
isAllowed = condition2 || condition3
isSecure = condition4 && !condition5 
// 🧠, we don't need to remember conditions, there are descriptive variables
if isValid && isAllowed && isSecure {
    ...
}
```

## Nested ifs
```go
if isValid { // 🧠+, okay nested code applies to valid input only
    if isSecure { // 🧠++, we do stuff1 for valid and secure input only
        stuff1 // 🧠+++
    }
    stuff2 // 🧠++++, we do stuff2 for all sorts of valid input, we should keep in mind stuff1, because it may interfere with our stuff2
} 
```

Compare it with the early returns and failing fast:
```go
if !isValid
    return
 
// 🧠, we don't really care about earlier returns, if we're here then all good

stuff2 // 🧠+

if !isSecure
    return
   
// 🧠+

stuff1 // 🧠++
```

We can focus on the happy path only, thus freeing our working memory for all sort os preconditions.

## High coupling with a framework
Frameworks evolve at its own pace, which in most cases doesn't match app's own life cycle.

By relying too much on a framework, we oblige all upcoming developers to learn this framework first (or its particular version). Even though frameworks enable us to launch MVPs in a matter of days, in the long run they tend to add unnecessary complexity and cognitive load.

Worse yet, at some point frameworks can become a significant constraint when faced with a new requirement that just doesn't fit the architecture. From here onwards people end up forking a framework and maintaining its own custom version. Imagine the amount of cognitive load a newcomer would have to build (i.e. learn this custom framework) in order to deliver some value. `🤯`

**By no means we advocate to invent all the things from scratch!**

We can write code in somewhat framework-agnostic way. The business logic should not reside within a framework; rather, it should use the framework's components. Put a framework outside of your core logic. Use the framework in a library-like fashion. That would enable new contributors to bring value from day one, without the need of going through debris of framework-related complexity first.

## DDD
DDD has some great points, although it is often misinterpreted. People say "We write our code in DDD", which is kind of strange, because DDD is intended for problem space, not for solution space.

Ubiquitous language, domain, bounded contexts, aggregate, event storming are all about problem space. They are meant to help us learn the insights about the domain and extract the boundaries. DDD enables developers, domain experts and business people to communicate effectively using a single, unified language. Rather than focusing on these problem space aspects of DDD, we tend to emphasize on certain folder structures, services, repositories, and other solution space techniques. 

Chances are, the way we interpret DDD is likely to be unique and subjective. And if build code upon this understanding, i.e., we create a lot of extraneous cognitive load - future developers are doomed. `🤯`

## Hexagonal/Onion architecture
There is some engineering excitement around all this stuff.

I myself was a passionate Onion Architecture advocate for years. I applied it here and there, I encouraged other teams to do the same. The complexity of our projects went up, the sheer number of files alone has doubled. It felt like we were writing a lot of gluing code. On ever changing requirements we had to make changes across multiple layers of abstractions, it all became tedious. `🤯`

In the end, we gave it all up in favour of good old dependency inversion principle. No port/adapter terms to learn, no unnecessary layers of horizontal abstractions, no extraneous cognitive load.

Even though these layered architectures have accelerated an important shift from traditional database-centric applications towards somewhat infrastructure-externalized approach (business logic became independent of any external stuff), the idea is by no means novel.

Those architectures are not fundamental, they're just subjective, biased consequences of more fundamental principles. Why rely on these subjective interpretations? Follow the fundamentals instead: DIP, IoC, single source of truth, coupling, true invariant, complexity, cognitive load and information hiding.

## Learning from the Giants
Take a look at the overarching design principles of one of the biggest tech companies:  
`Clarity`: The code’s purpose and rationale is clear to the reader.  
`Simplicity`: The code accomplishes its goal in the simplest way possible.  
`Concision`: The code has a high signal-to-noise ratio, or intrinsic-to-extraneous cognitive load ratio.  
`Maintainability`: The code is written such that it can be easily maintained.  
`Consistency`: The code is consistent with the broader codebase.  

Does the new fancy buzzword comply with these principles? Or all it does is creating extraneous cognitive load?

<details>
  <summary>Here's a fun picture</summary>
  <img src="img/complexity.jpg">
  Code Complexity vs. Experience from <a href="https://twitter.com/flaviocopes">@flaviocopes</a>
</details>

**If you are holding a lot of facts that aren't directly relevant to the problem at hand, think about future developers, yourself included. They would have to recreate this high cognitive load.**

*Contributions are welcome!* 🚀
