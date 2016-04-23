# Public and private dependencies

## Introduction

### Preface

We can safely proclaim that the era we live in is the age of information. This is ment both in the context of value that is today associated with every form of raw data (as anyone working with analytics and big data can attest to), and on the other hand, in the ease that the publicly available informations can be accessed. This accessibility is true for literally every area of expertise we can think of, and naturally, even truer when talking about the science discipline which brought on the information revolution itself - that being informatics, and in a narrower sense computer programming.

With thousands of lines of code and programming knowledge shared every minute (TODO cit. needed?), it is only logical that many of the programmers (the author included), when faced with a problem that isn't hyper-specific to the task or a project they are working on, will turn to the internet for solutions. We may be talking about smaller code chunks, like the at the moment infamout left-pad, for which peple might argue that the only reason for not writing them on their own is lazyness. Or, we can talk about complex libraries whose implementation is simply not feasible within the bounds of time available for the project, or that aren't within the area of expertise of the programmer at hand (and really, if we're to take the example to the extreme, you shouldn't be required to write a secure database server solution every time you're creating a web application).

Either way, a need for a method of easy code sharing arises, and not only to ease the installation of a more elaborate solutions for our programs. We could argue that the DRY ("don't repeat yourself") principle, which is well known by the programming community, can be expanded to include the whole universe of code on the internet - in a sense that it's useless to 'reinvent the wheel', even for a function with just a few lines of code, if there is already a tried-and-true variant shared on the web. This also promotes readability - when a library does a single, well defined task across multiple application, a developer that is familiar with it can instantly know it's purpose seeing it again in other context, in contrast with seeing a custom method doing the same job, but written by a different programmer (and in worse case, having unpredictable side effects).

Last but not least, there is the other side to the benefits of an easy and unified way of code sharing, the one of code reusability and the view of someone writing a would-be package function. Again, there is a need for a simple tool allowing us to export our library into the world, so that we can later include it without carrying a collection of files accross multiple repositories. Later, when bug-fixes, modifications or additional functionality is needed, we have a well separated bundle instead of a set of broken copies, all of which would need to be rewritten.

Of course, the solution to many of the aforementioned problems lies in the concept of openly available libraries (packages, modules or simply a bundles of functions, methods, classes or lines of code in general). Still, libraries on their own don't deal with the problem of ease of their accessibility. Every (reasonably advanced) Windows user knows the pain of hunting the internet for missing .dll files, where adding one only triggers an error of two more that are still required. And of course, as the title suggest, the big solution we're building up for is in the concept of Package managers - yet, the trouble does not end there.

### Package managers and dependencies

With package managers, the accessibility of even smaller, user written libraries increases greatly. Despite the fact that they come in many different flavours and may encompass a wide array of additional functions, often related to the setting they operate in, the core idea behind them is to rid the programmers of the burden of finding modules on the internet, installing them manually and later, maintaining them (which mostly means simply updating to a newer version). Thus, we should establish that by package manager, we refer to a tool that allows us to (as a bare minimum), firstly, easily find a package of given name, provided that it has access to some kind of package repository, and secondly, install it, so that we can use the package (often within our desired version range) with as little further tinkering as possible. Also, a way to bump a certain package version should be at hand, in case it is available and satisfies the provided constrains within the dependency tree (more on this later).

With a tool like this at hand, it is only natural that when the need for larger and more complicated packages appears, we want to reuse some of the code already available in different modules, ideally getting it via the given package manager. And as we had said in the previous section, in most cases this need existed even before a package manager for the given setting was conceived (the only exception being environments which had dedicated package managers from their very beginning). Such action creates the concept of nested dependencies - by which we will mean the situation, where for package A that you require for you project also needs packages B and C to function properly (this should not to be confused with nested vs. flat package structure, discussion on which will be brought up later). Ideally, we would want our package manager to handle this for us - in our example, we only care about package A and it working properly, the fact it needs two other libraries to run may often stay completely obfuscated from us. If we were to adhere to the principle we have set in the previous article - installing working packages with minimal further tinkering on our side, we would expect our package manager to simply install B and C and not to bother us anymore. Yet, this is where the design philosophies of many package managers differ, and for good reason. What if this is not possible ? What if there are multiple possibilities, some of which are valid, some that are not, and some that are valid in the current situation but will make installation of the next required package impossible. As the dependency tree grows larger, more and more of these problems arise - and although it is hard, if not nigh impossible to provide a definitive answer, the focus of this paper is primarily in trying to solve some of the given issues, and in building a concept which would prevent the emerging of some problems all together. We will also show in our managers run down that in this regard, the ostrich principle of doing nothing and leaving it all to the user is the most common choice - we believe this should not be the only option, and that some of the (valuable) development time, currently spent on resolving dependencies, could be saved using well known heuristics.

If we were to take a higher-level look on the issues that arise within current generation of package managers, you could maybe write off some of the discussion as a purely semantic jabber (i.e. when talking about peer dependencies and whether specifying a need for a 'peer' is something that should exist within the world of nested dependencies) Still, anyone who ever needed to work or maintain a large and presumably also an old project, understands the need for the codebase and relations within it to, in a laymans terms, make sense. Some other may be more environment - or language - specific. You may need to avoid importing the same package twice, or having different versions of it accross the project because of the conflicting namespaces it would create. TODOcheck&priklad There is also the question of the degree of freedom that the writers of libraries should be given - mainly in the fact of making assumptions about the environment their library will be run in, along with assumptions about other already installed dependencies (again, the concept of peer dependencies comes to mind). Summing up all of these, the big question lies in the dispute between private and public dependency model (and their variants and combinatons) and we believe that in the javascript and node.js module environment, we can provide a model that is in our opinion superior to the one that is used now.

## Terminology

Let us first define the terms and vocabulary that will be used throughout this paper. This section should serve just as an informal rundown, some of the terms mentioned here will be brought up again in the following chapters within a more formal context and definitions. Yet, since the discussion in the Package managers chapter, which leads to the establishment of our formal model (and highlights the reasons for it), requires us to use some of the vocabulary which, though settled within the programming community, may still be unfamiliar, we offer this brief list beforehand.

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

### TODO moar

### Semantic versioning and dependencies

### TODO more that work with semantic versioning

## Public dependencies - our model

## Hardness of shared dependencies problem

### Resolving a dependency tree

### Resolving a single version of a package

### Simulated annealing

## Implementation

### Node module system

## Results

## Future work

## References

