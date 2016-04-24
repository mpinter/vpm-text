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

Let us first define the terms and vocabulary that will be used throughout this paper. This section should serve just as an informal rundown, some of the terms mentioned here will be brought up again in the following chapter within a more formal context and definitions. Though the vocabulary is largely estabilished within the programming community, and we will try not to deviate from the settled terminology, we still offer this brief list just in case.

Package - may also be called a module or an library, a set of functions or classes that performs a well defined, enclosed task. It does so on it's own unless we're talking about a plugin.

Plugin - a special kind of package that works in synergy with a different one or changes the behaviour of a different package, thus requiring it to work.

Dependency - a term used to describe a package that is required either by our own code or by a different package for it to work properly (or at all), although in most situations and environments the program we're writing is also considered a package or module, making the former and the latter option essentially the same thing.

Dependency tree - describes the dependencies of a certain program or module and the relations between them. Note that circular dependencies (i.e. package A requiring B requiring C which requires A again is an example of a circular dependency of length 3), despite being uncommon, is something that does happen within the context of package managers. This means that the dependency 'tree' may in general take the form of a general directed graph - but we'll still use the term dependency tree since it's well established (and sounds better than 'directed dependency graph').

Private dependendency - a dependency that is used only by the package that required it and is not shared within the dependency tree.

Public dependency - or a shared dependency, a dependency required by two or more packages within the dependency tree. More specifically, we're talking about the situation where the given packages are sharing the exact same files (or functions or classes), in contrast with using two copies of the same module in certain version - the latter still being considered a situation with two separate private dependencies.

Package manager - in general, describes a program meant to resolve the dependency tree given a certain recipe to do so. The amount of proactivity doing this varies largly between different managers and environments.

## Hardness of shared dependencies problem

### Resolving a dependency tree

### Resolving a single version of a package

## Package managers

In this section, we'll go through the list of currently available package managers, for both node.js (or javascript in general) packages and package or module systems in other languages or environments. We'll be particularly interested in a way that they handle shared dependencies. Naturally, those that deal with both private and shared ones and are thus closest to our line of work will also be the ones we take the closest look at. On the other hand, managers which enforce a single package version across the whole dependency tree essentially model the borderline situation of making every dependency public in our own mixed model, that will be presented later in this paper.

### Unix repositories

### TODO moar

### TODO more that work with semantic versioning

## Discussion on current models (? or do this within the previous segment ?)

### Semantic versioning and dependencies

### The need for shared dependencies

First of let us establish that we will be talking about shared dependencies in the sense of two packages requiring to agree on a same version on one of more of their dependencies (and in some cases also the need to agree on its subdependencies), in contrast with two packages linking to the same package installed on the disk - although the former may often imply the latter, the opposite implication is much rarer (at least in the world of node.js).

## Public dependencies - our model

We will now present our suggested model for shared package dependency resolution. If it is to be compared with NPM's design philosophy, the key difference lies in peer dependencies being replaced by public dependencies - everything else either stays the same (at least while we are talking about the high level concept, not the implementation of a concrete package manager), or is a result of the aforementioned change.

### Overview

We propose a model with both private and shared dependencies. Our concept of dependency sharing is realized via public dependencies - from now on, when refering to a public dependency, we will be talking about the design proposed as follows:

Public dependency is a dependency whose API may be accessed by the module requiring the package within which it is defined.

Meaning that if package A depends on package B and exports from it a function, class or in any way provides direct access to package B, the dependency between the two packages should be defined as public. Each of these dependencies are inherited along any number of public branches and a single step along a private branch (explained further in the following section), which creates an inherited (public) dependency:

Inherited dependency locks a package it represents to the same version as was the public dependency it originated from - it is not automatically installed and exists solely to cause (and thus warn about) conflicts in the dependency tree

Inherited dependencies are exported the same way as public ones - apart from this, they serve exactly the same purpose as peer dependencies do in NPMv3 and higher (before that, peer dependencies were automatically installed, now they serve mainly as a source of warnings and suggestions). As mentioned before, private dependencies are handled in a way that is very similar to any other package manager supporting this concept, except for the times when they play a role in aforementioned public dependency inheritance.

TODOmaybe something more here ?

Let us first take a closer look on the key concepts behind our proposition, and then see how it works within the current standard of node.js modules, ruled by NPM.

#### Dependency inheritance

The primary factor that differentiates public and peer dependencies is the notion of inheritance (more in regards to this comparison later). In essence, each public dependency and inherited public dependency is exported in the direction towards the root (from child to parent) along any number public relations (or 'branches'), becoming an inherited dependency after first such export, and then finally along a single private branch - from there, it can't continue, not even using public branches.

The reasoning behind this follows the definition of what it means for dependency when it is marked as public - that one or more of its functions or classes may be accessed from it's parent. Let u first look at the situation with nested public dependencies. Suppose that package A is a public dependency of B which is in turn a public dependency of C. This means that there exists a function (or a class) in B that has access to A's API, and there is also one that is exported to C. Without any additional information, we can not reasonably rule out the possibility of this being the same function (or class) - meaning that A's API is exported further down into C, despite the fact it never explicitly asked for it.

Now, if the package C requires another version of A directly - lets call it A', and again, we can't make any assumptions of whether it plans to use the function exported from A through B (if such function exists), but similarly we can't rule out the possibility of B existing solely to augment A's functionality, and C's intent to use it in conjunction with some other function provided by package A'. Therefore, the only reasonable stance we can take in this situation is to conclude the worst scenario and force both A and A' to agree on a single version.

Similar situation arises not only for subtrees with purely public relations, but also for a single level of private branches (thus the reason public dependecies are exported along this extra step). Let us once more create an example with root package R, this time with two private dependencies B and C. Both of these specify a public dependency A, let us again theorize about their version not agreeing, consequently spawning package A as B's (public) dependency, and package A' as C's dependency. Now again from root's perspective, both B and C exports A's functionality in some way - whether it is a function a class or anything else. Either way, we are facing a situation where the two entities of possibly different versions (of the same package) may interact, which can once again yield unpredictable results. Hence, our best bet is a consent between the versons of A and A'.Expanding the idea for the option where one dependency is exported along a private branch and the other along a public one should be trivial.

While this may sound as an additional layer of intricacy, especially if comparing to different approaches within the field. Yet (despite being fans of the KISS principle), we argue that in this case it is needed and without it, certain problems arise when dealing with nested public dependencies (as is illustrated in the section regarding peer dependencies). Additionaly - still in defense of the proposed complexity - we should keep in mind that with the osstrich based 'ignore the problem' approach being the most widespread one, we are hardly expected to stay on the same level of complexity as a method where such complexity is inherently non-existent. The other side to the coin is that in a way, when our concept is put in contrast with peer dependencies, it actually unifies the 'source of truth' for required dependencies - as in, the flow of contrains is always from the child to the parent.

A certain detrement which emerges from this behaviour is nevertheless present, and will be further elaborated upon at the end of the next section, since it is perhaps more closely related to the matter presented there.

#### Self-containment of public dependencies

In our model, we can say that each dependency, whether it is public or private is self-contained - or in other words - does not assume any information about the outside world. The dependency stores all the neccessary information about packages it needs for it to work either directly in itself, or inherited from one of it's subdependencies - yet, still contained within the subtree of the dependency without polluting the 'global' scope (even if it is only the scope of the parent). You always get the full info required for the current package to install by looking at all of it's children, and you can be sure there is never an additional 'hidden' source of such information.

At first glance, this may seem as a more of a semantic than a functional change - since despite our containment for the current package we nevertheless export the same constrains to the parent as we would have done using peer dependencies. The difference lies in the matter of exporting quite a few more of these because of the way the inheritance works.

While we consider the fact that current setup allows us to comfortably reason about the needed dependencies as important enough, there is still a severe drawback associated with this (and in part with the previous) feature, which also has to be recognized when working on a package manager utilizing our dependency model. Although we indeed gain the ability to resolve each subtree on its own, with all possible conflicts modeled in the form of children of our subtrees root, the state of this root itself is now defined not only by the version of a package it represents, but also by its chosen public dependencies and public subdependencies - or more importantly, by the ones we export as inherted from our root. Firstly, this means that we can have (and in fact may often need) multiple copies of the same package in the same version installed, each one resolved using (and exporting) different public dependencies. Secondly, we can no longer choose to use a installed package as a dependency simply based on it's version - which implies that the ied package manager based approach of differentiating packages based on their content (whether we are taking symbolic links into account or not - the problem may arise at a deeper level) is not avaliable to us anymore.

#### Comparison with peer dependencies

The model is deliberatly designed to take the current node.js packages standard into consideration. In fact, it is fully backwards compatible with npm's model, which deals with necessity for shared dependencie through the concept of peer dependencies (more on the in NPM's section in the third chapter). At first glance, if you compare the behaviour (in terms of satisfiability) of a one level deep public dependency and a peer dependency, it will be exactly the same. This is in part because we want our model to be immediatly useble under present circumstances - with NPM being the standard, literally every javascript package adheres to the rules set by it. It would be foolish to think of coming out of nowhere with an altogether different proposition, especially when the current one (despite the problems presented throughout this paper) works, and expect everyone to jump on a bandwagon. On the other hand, since both the public and the peer dependencies exist to deal with the same issue, namely the need to have shared dependencies (as defined earlier), it isn't all that surprising that both concepts end up working similarly.

Before we continue with the comparion, a short note about peer dependency handling and the change that was made to it moving from NPMv2 to NPMv3 - as mentioned in the chapter three, peer dependencies are no longer installed automatically in the newest NPM version, instead they only serve as a source of warnings and errors. Thus, since packages working previously with v2 had no need to specify the dependencies marked as peer for the second time in their parent, many are not compatible with version 3 (which had it's first public release in september 2015, and started to really spread maybe towards the end of that year). At the time of writing, v2 is still in long term support mode, and probably will be for some time, and project that had not migrated over to NPMv3 are still a plenty. The reason for writing this is that our model is compatible with approaches used in both of the major NPM versions - because of the way we intall the functionality at the lower level, akin to the way of NPMv2, and then export inherited public dependencies which serve as a guideline similar to peer dependencies in NPMv3.

Each kind of resolvable (or unresolvable) dependency tree modeled using public dependencies can be modeled via peer dependencies, while preserving it's resolvability. The second implication does not hold for a special subclass of dependency trees, which we are going to elaborate upon in the following section. The formal proof of these theses will be presented further down the chapter.

##### Stricter restrictions on certain classes of problems

Despite all of the aforementioned backwards compatibility, there are certain forms of dependency trees accepted by NPM, whether it is of version 2 or 3, that we want our model to deliberatly fail on. These are closely related to the way the public dependency inheritance is designed, that is - it was shaped to the proposed form to account for this additional strictness. We shall first show the emerging problem on an example. We will model it using the NPM's peer dependency model, and later we will show the same example modeled using public dependencies (TODOimg) and the way they change the resolution status. We will also be using semantic versioning since it has become the standard with node.js

Suppose we have the following situation - TODO example

TODOimg1 img2

Our model therefore proposes to keep the versions of public dependencies consistent across the whole public branch (and a single private one). This way, when a situation such as in one of the previous examples emerges, we can be sure that every API that was exported and reused again uses the same package version, which was not the case with previous model.

In a way, this is removing some of the programmers freedom in exchange for protection from a weird class of hard to track down bugs resulting from mixing APIs of two different versions of a same package - again, we feel that it isn't too wise to trust the package creators to detect this kind of conflicts, since they indeed may apper extremely rarely. Still, the rarer and more cryptic the bug happens to be, the harder it will be for the unlucky person who happens to run into it to correctly track it down. The important message of this section is, that although the imposed rule may be stricter than might be needed in some cases, it is still the best that can be done without making assumptions about installed packages that can't be programmatically checked. In addition, the proposed design creates a reasonable safeguard for situations with a large amount of nested public or peer dependencies, problems with which can be naturally hard to untangle.

##### Key differences

Let us now round up the main differences between the public and peer dependency approaches.

1. Public dependencies are self-contained

As we had already discussed before, what we consider a huge advantage of our public dependency concept is that all the information neccessary for the successfull installation of the whole subtree is stored only in given subtrees root and it's successors. This is in stark contrast with the design of peer dependencies, which need to make assumptions about the outside world - or maybe even force the outside world to behave according to our subtrees idea, for it to successfully install. We believe this is a bad design decision that makes the dependency tree as a whole harder to reason about, and maybe more importantly harder to satisfy. In fact, even the community behind the NPM is perhaps slowly trying to phase out this concept (TODOcit issue), as may be already seen in softening the requirements of peer dependencies in version 3 of NPM.

2. Public dependencies are exported along other public dependencies

In a certain aspect, peer dependencies are like public ones that are exported only a single step up the dependency tree - in contrast of public dependencies which bubble along any public branch and a single private branch. This means that with public dependencies it is computationaly harder to determine whether a conflict exists within the current hierarchy or not, but in turn, it buys us an additional level of security agains the problems mentioned before.

3. Inherited dependencies are not mandatory

Whilst inherited public dependencies directly resemble peer dependencies in many ways, maybe even more so now in version 3 of NPM, the semantic role between the two is quite different. Each peer dependency essentialy tells us to 'require a certain version of a given dependency to be installed on a parent package' (despite the fact that avoiding any installed version completely only yields an ignorable error) , while the condition set byt an inherited public dependency is a bit softer - 'make sure that if any other dependency for the same package appears on this level, it matches the version set by the public dependency inherited from'.
Of course, when being compared to NPMv2, the difference is obvious, taking older NPMs automatic installation into account - although one may argue that in our concept all of the public dependencies are installed no matter what, which is true and may be considered a drawback (and once again, is a discussion for the implementation chapter). Yet, the installed public dependencies are hidden in the 'local' scope of the dependency which required it, and are exposed to the outside world (in a way that it can access it) only when they are explicitly required there.

### Formalization

We shall now prepare the theoretical apparatus we are going to use to formaly prove the correctness of our approach and the one sided equivalence in regards to the peer dependency based model.

Firstly, let us establish the dependency tree as a graph of nodes - which represent a package resolved in a concrete version - and edges, which represent a dependency requirement, whether it is public or private. In this and the following section, we will be adressing these edges (or relations between nodes) as dependencies, as opposed to the term dependency denoting a required package itself up to this point. Without explicitly stating otherwise, the term dependency will now be used exclusively in this way.

For each node, let us define two sets - privateExports and publicExports. These will mark the dependencies exported from the children of the given node along either a public or private dependency. The privateExports set then stays private to the node, while publicExports is the set exported from it by the means of both public and private dependency. The reference to node itself is contained in it's own publicExports set, which is in turn always a subset of publicExports.

The only source of errors comes from conflicting versions in the privateExports set (naturally, since everything else within the node is a subset of it). The only action available to us during the dependency tree resolution is the choice of version for a given package. In our proofs, we will often make the decision non-deterministically, since we have already proven the NP-completeness of this problem in general.

The characteristics we set out to formally prove are as follows. First and foremost it is the correctness of our proposed design - we will prove by induction that with each step the resolvability assumed by our model satisfies the constrains we have set to require from a resolvable dependency tree. Secondly, we are to prove the one sided transformability of peer dependency model into our concept, and show that the other side of this implication fails if and only if we are faced with the kind of dependency tree that was described earlier (with nested, 'cascading' dependencies).

Thus, our set of axiom is:

TODO

With TODO(odovdzovacie) rules being:

TODO

### Proof of correctness

## Implementation

### Node module system

### Simulated annealing

## Results

## Future work

## References
