# Public and private dependencies

## Intro

### Preface

We can safely proclaim that TODOinfshare . This is true for literally every area of expertise we can think of, and naturally, even truer when talking about the science discipline which brought on the informational revolution itself - that being informatics, and in a narrower sense computer programming.

With thousands of lines of code and programming knowledge shared every minute (TODO cit. needed?), it is only logical that many of the programmers (the author included), when faced with a problem that isn't hyper-specific to the task or a project they are working on, will turn to the internet for solutions. We may be talking about smaller code chunks, like the at the moment infamout left-pad, for which peple might argue that the only reason for not writing them on their own is lazyness. Or, we can talk about complex libraries whose implementation is simply not feasible within the bounds of time available for the project, or that aren't within the area of expertise of the programmer at hand (and really, if we're to take the example to the extreme, you shouldn't be required to write a secure database server solution every time you're creating a web application).

Either way, a need for a method of easy code sharing arises, and not only to ease the installation of a more elaborate solutions for our programs. We could argue that the DRY ("don't repeat yourself") principle, which is well known by the programming community, can be expanded to include the whole universe of code on the internet - in a sense that it's useless to 'reinvent the wheel', even for a function with just a few lines of code, if there is already a tried-and-true variant shared on the web. This also promotes readability - when a library does a single, well defined task across multiple application, a developer that is familiar with it can instantly know it's purpose seeing it again in other context, in contrast with seeing a custom method doing the same job, but written by a different programmer (and in worse case, having unpredictable side effects).

Last but not least, there is the other side to the benefits of an easy and unified way of code sharing, the one of code reusability and the view of someone writing a would-be package function. Again, there is a need for a simple tool allowing us to export our library into the world, so that we can later include it without carrying a collection of files accross multiple repositories. Later, when bug-fixes, modifications or additional functionality is needed, we have a well separated bundle instead of a set of broken copies, all of which would need to be rewritten.

Of course, the solution to many of the aforementioned problems lies in the concept of openly available libraries (packages, modules or simply a bundles of functions, methods, classes or lines of code in general). Still, libraries on their own don't deal with the problem of ease of their accessibility. Every (reasonably advanced) Windows user knows the pain of hunting the internet for missing .dll files, where adding one only triggers an error of two more that are still required. And of course, as the title suggest, the big solution we're building for is in the concept of Package managers. Yet, the trouble does not end there.

### Package managers and dependencies

TODOHERE

## Terminology

Let us first define the terms and vocabulary that will be used throughout this paper. This section should serve just as an informal rundown, some of the terms mentioned here will be brought up again in the following chapters within a more formal context and definitions. Yet, since the discussion in the Package managers chapter, which leads to the establishment of our formal model (and highlights the reasons for it), requires us to use some of the vocabulary which, though settled within the programming community, may still be unfamiliar, we offer this ...

Package - may also be called a module or an library, a set of functions or classes that performs a well defined, enclosed task. It does so on it's own unless we're talking about a plugin.

Plugin - a special kind of package that works in synergy with a different one or changes the behaviour of a different package, thus requiring it to work.

Dependency - a term used to describe a package that is required either by our own code or by a different package for it to work properly (or at all), although in most situations and environments the program we're writing is also considered a package or module, making the former and the latter option essentially the same thing.

Dependency tree - describes the dependencies of a certain program or module and the relations between them. Note that circular dependencies (i.e. package A requiring B requiring C which requires A again is an example of a circular dependency of length 3), despite being uncommon, is something that does happen within the context of package managers. This means that the dependency 'tree' may in general take the form of a DAG (directed acyclic graph) - but we'll still use the term dependency tree since it's well established (and sounds better than 'dependency directed acyclic graph').

Private dependendency - a dependency that is used only by the package that required it and is not shared within the dependency tree.

Public dependency - or a shared dependency, a dependency required by two or more packages within the dependency tree. More specifically, we're talking about the situation where the given packages are sharing the exact same files (or functions or classes), in contrast with using two copies of the same module in certain version - the latter still being considered a situation with two separate private dependencies.

Package manager - in general, describes a program meant to resolve the dependency tree given a certain recipe to do so. The amount of proactivity doing this varies largly between different managers and environments.

## Package managers

In this section, we'll go through the list of currently available package managers, for both node.js (or javascript in general) packages and package or module systems in other languages or environments. We'll be particularly interested in a way that they handle shared dependencies. Naturally, those that deal with both private and shared ones and are thus closest to our line of work will also be the ones we take the closest look at. On the other hand, managers which enforce a single package version across the whole dependency tree essentially model the borderline situation of making every dependency public in our own mixed model, that will be presented later in this paper.

### Unix repositories

## Hardness of shared dependencies problem

### Resolving a dependency tree

### Resolving a single version of a package

### Simulated annealing

## Implementation

### Node module system

## Results

## Future work

## References

