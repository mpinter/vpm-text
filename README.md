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

### Operating System package managers (GNU / Linux)

By the sheer nature of the way the packages tend to build on one another, the outset of package managers can be tracked within the FOSS (which stands for Free and Open-source Software) community. More specifically dpkg, which is considered to be one of the earliest examples of a package management software, emerged as a part of the Debian project. While it did not feature any form of automatic dependency reolution at the time of it's inception, it is also regarded as the first one with widely known tool for such process, in the form of APT (Advanced Packaging Tool).

In the words of Ian Murdock, one of the creators of the Debian linux distribution, the concept of package management is the biggest advancement that linux has bought to the computer industry. (cit-3). Murdock was also one of the original creators of dpkg, although the package has been rewritten by numerous programmers since then. While system packages are different by their very nature from the libraries for a programming language or environment (such as the target of our endeavor - node.js and javascript), the package managers created to handle them deal with the very same problem we have defined earlier - that is, the dependency graph represents essentially a general satisfiability problem, with the domain of available packages being way too large to explore as a whole. Thus, the same kind of difficulties arises during the dependency resolution. In fact, due to the sheer size of some of the system repositories, and the lengthy period for which some of the packages present in them are maintained (some may very well have 20 or more years already), these problems may be even more severe. Thus, we can maybe observe that even the package managers and their approach towards the dependency tree resolution might be, if we are to say so, more mature.

We will focus on Linux-based operating systems, since package managers on different OSes are mostly either much simpler in their way of resolving dependencies - that is, they often do not do so at all, or are ports of Linux managers or managers strongly inspired by ones available for Linux.  By the first of the options we mean mostly the various kinds of App stores or other distribution platforms, which are, strictly speaking, also a form of package managers (albeit, the apps or packages in them rarely have any decentralized dependencies - those which aren't already expected to be a part of the OS itself).

Packages on Linux systems are usually distributed in one of the two de-facto standard format - either DEB or RPM. While .deb is essentialy a tar archive with additional metadata, .rpm is an ad-hoc binary format designed specifically for this purpose(cit1). The most important fields of metadata specified by both of these formats are (besides the obvious - name of the package and it's version) the dependecies of a given package, conflicts - packages which can't be installed alongside of the one being currently installed, and then also the so called pre-dependencies - packages that need to be installed before the installation of our package beings (as opposed to regular dependencies which can be deployed on the system concurrently with it's installation). For all intents and purposes of this paper, we do not need to go into any further specification of either of them, we will just note that from the following list APT uses the .deb format (probably along with OPIUM, since it's comparion tests are ran against APT, despite this fact not being mentioned in OPIUM's research), while ZYpp, YUM, DNF use the RPM package format.

####APT

Advanced Packaging Tool is probably the most high-profile package manager, whether it is because of functionality or simply as a result of being the primary package manager of Debian and Debian-based Ubuntu (which is currently the most wide-spread linux distribution). As mentioned before, it works with the .deb package format, though flavours that run with .rpm exist. APT has two modes of operation - the immediate and the interactive mode. The former offers a fast way to solve most dependency problems, while the latter allows for user input to provide feedback to the resolver and thus, guide it towards the correct solution.

The immedate mode is essetially a bruteforce algorithm - the manager will list through the depedencies attempt to install each of the packages there - or check if it is already installed, or whether it's already satisfied as a suggestion (suggestions are not installed if another package with the same suggestion in same version is already installed). If such package is not available, or is conflicting within the current setup, it will attempt to intall the highest-priority (an optional field for .deb packages - can be Required, Important, Standard, Optional, Extra, in order of decreasing priority value) package whose candidate version provides the target of the current alternative.(cit-apt) When even this option fails, it looks for alternatives specified within the package dependencies (as a disjunction of packages which may replace each other). If any of the previous step succeeds, the algorithm will be called recursively on the newly installed package's dependencies.

 As it tends to be with most bruteforce algorithms in various programming applications, the immediate mode provides a good baseline but is in no way a complete solution to a wide array of situations which may happen during the resolution algorithm. Therefore APT provides users with the aforementioned interactive mode, where they may manually choose the dependencies they want to install along with their versions. This approach, while no doubt functional, is far from being user friendly.

####OPIUM

Optimal Package Install/Uninstall Manager can be considered more of a science project, or a proof-of-concept of a research paper(cit4). It uses off-the-shelf SAT solvers in conjunction ILP solvers to resolve the dependency tree prior to proceeding with installation. OPIUM claims to be complete, in the sense that if a solution exists, it is guarateed to find it. In addition, it optimizes the cost of this solution (the number of intalled or uninstalled packages) - this comes as a natural requirement, since otherwise a 'trivial' strategy of uninstalling every package on a system to make sure it does not conflict with the new installation would provide a vald solution. Still, the use of generic SAT solvers, while sufficient, have proven to be maybe the greatest holdback of this project - meaning they might not have been quite suitable for the problem at hand. More precisely, heuristics used in said general solvers, whether the ones solving satisfiability or those computing integer linear programming (which is used specifically in the case where uninstalling certain packages was needed to proceed (todo cit?)) have shown to be not all that appropriate for the hierarchies yielded by package repositories(TODOcit). This was addressed in the next package manager on this list.

####ZYpp

In the words of OPIUM's creators - 'Opium runs fast enough to be usable'(cit4). The natural step forward while maintaining the idea of using SAT solver to resolve the dependency tree prior to the package installation itself, is to use a solver dedicated to the task of solving the package hierarchies. This was realized in ZYpp (or previously libzypp), a Novell sponsored package manager for openSUSE and other SUSE linux distributions, used in many of the core packages that this distribution is known for (like YaST)(cit5). In contrast with OPIUM, while both are built upon the same idea, ZYpp was the one built for production purposes. It is also worth noting that at the moment of writing this paper, the DNF (Dandified YUM), which is the default package manager for Fedora distribution starting from version 22, uses the libsolv solver library from ZYpp.

#### Other

The other, currently fairly large package managers that did not make it on this list (because of reasons usually related to the fact that their resolution algorithm was not interesting to us) are as follows. Yum, which has just been replaced on Fedora and is still the main package manager for CentOS, and is considered by the community to be broken and obsolete, with documentation either missing or being cryptic.(cit-yum). Pacman, the package manager of Arch-linux, using it's own binary package format. There is also the briefly mentioned DNF that uses the resolve algorithm of ZYpp. And finally Portage, that again uses a SAT solver with custom heuristics (cit-port), which is an approach already talked about in ZYpp's section.

#### Note on difference between system and programming language packages

Citing from OPIUM's paper once more - the three problems a package manager for linux/gnu packages is trying to solve are:

Install Problem : Determine if a new package can be installed and, if so, determine how.

Minimum Install Problem : Determine the optimal way to install a new package, where optimality is determined by an objective function whose value is to be minimized.

Uninstall Problem : Given a new package to install, determine the minimal number of packages (possibly none) that must be removed from the system in order to make the package installable.(cit-4)

This is different to our situation within the node.js environment. That is, despite the title of this section, the discussed differencies may perhaps be appliable only in the context of javascript modules, since that is our target platform and the point of our interest.

The only time we are essentially solving the uninstall problem is with two conflicting versions of a same package. In a way, we are only upgradrading or degrading a single package. Although this may lead to change of it's dependencies and some packages becoming no longer needed, they (the 'obsolete' ones) in no way interfere with the installation of the new package - even if a public dependency which might disrupt the new installation exists among them, it will no longer be used and therefore has no effect. In fact, we can view the uninstall problem simply as the removal of needless modules. Yet, we can look at the install and minimum install problems in basically the same way.

There is also the notion of pre-dependencies being specified in both the .deb and .rpm formats. As noted in the Debian documentation: Pre-Depends should be used sparingly, preferably only by packages whose premature upgrade or installation would hamper the ability of the system to continue with any upgrade that might be in progress. (cit-predep) This option is not really needed in the context of javascript modules, since their configuration or any kind of interaction between the dependencies happens only at runtime.

### Programming environment package managers

"It is a truth universally acknowledged that a programming language must be in want of a package manager."(cit-pkgman) And indeed, we can say that the want is so prevalent that it occasionaly materializes even in the form of multiple package managers for a single language. Critics of this practice call for a possibility of centralization - using one package manager across multiple languages, yet, as the communities of distinct environments stay relatively separated, principles of each manager are different and conventions move at different paces and often in different directions, we are left with a specific set of tools for each language for at least the next couple of years.

We will be talking about 'environments' more so than 'programming languages', to cover instances such as Bower or .NET framework, which span across a few of those, or something like node.js, which may be viewd as a subset of javascripts habitat. Albeit, in most cases they will be used as synonyms, refering to the language itself, and whatever system of libraries (modules, packages... ) it supports.

#### Semantic versioning

A brief note on semantic versioning before we move on to examples. As we were talking about different conventions, semver is perhaps the one actually being adopted by multiple separate programming communities and package managers. It is a standard that dictates the way a package developer should change the version of a package, which itself is of a format MAJOR.MINOR.PATCH, according to the following set of rules:

MAJOR version when making incompatible API changes
MINOR version when adding functionality in a backwards-compatible manner
PATCH version when making backwards-compatible bug fixes(cit-semver)

Semver is most prominent in the NPM community, though the other high-profile package manager pushing it is Ruby's Bundler.

Despite the (short) documentation of the standard opening with claims of it helping with the so-called 'dependency hell'(cit-semver), and we do not wish to challenge these claims, from our perspective, that is - of someone being concerned primary about automatic dependency resolution - it is not of much use at all. The semver documentation specifically states that it has no intention of documenting the changes in dependencies - meaning that a patch version is free to remove every dependency that the package had since it inception as well as replace them with a completely new set, whilst still adhearing to the standard which semver has set.

All in all, this means that while humans can use the information provided by it to instantly know how far can they push with upgrades before their application starts to break (provided that everything works as intended), package managers are still left in the dark in terms of knowing which version changes the dependencies or how severe this change is. We will get back to semantic versioning in the implementation chapter, but until then it serves no further purpose for this paper.

#### Bundler

#### Pip

#### NuGet ?

#### Other

#### Node.js

As the title of this paper suggests, we will be most interested in package managers set to work within the node.js environment. At the time of writing this paper, and to our knowledge, there are two such package managers available - NPM being the obvious one and also the one that is production ready (and used in production every day by thousands of programmers around the globe), with ied being more akin to a small experiment, taking on a different approach (and not yet working for general cases).

##### NPM

While we are going to compare  ... allowing the developers much more freedom with .. One may say that this additional degree of freedom is what lead a lot of people to perceive NPM as being prone to errors and inconsistencies. When in fact, if a different architecture of module dependencies, or a philosophy of one of the previously mentioned package managers (like bundler) was in place, many of the ready-to-use packages would not be even feasible to install on the given setup. Or, at minimum, the developers of these package would have to work much harder to keep their package compatible with the highest available version of their dependency, so that this version may be shared accross the whole project.

TODO

We could speculate that it was this freedom that made npm or maybe perhaps node.js as popular as it is today. The growth rate of NPM's repository is also currently unmatched (TODOimg cit-modcount), though if we were to take a more cynical look at this fact, the amount of modules which are a wild ideas at best, and unusable thrash at worst, is also non-trivial. TODO

##### IED

## Discussion on current models (? or do this within the previous segment ?)

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

While this may sound as an additional layer of intricacy, especially if comparing to different approaches within the field. Yet (despite being fans of the KISS principle), we argue that in this case it is needed and without it, certain problems arise when dealing with nested public dependencies. Additionaly - still in defense of the proposed complexity - we should keep in mind that with the osstrich based 'ignore the problem' approach being the most widespread one, we are hardly expected to stay on the same level of complexity as a method where such complexity is inherently non-existent. The other side to the coin is that in a way, when our concept is put in contrast with peer dependencies, it actually unifies the 'source of truth' for required dependencies - as in, the flow of contrains is always from the child to the parent.

In a way, this is removing some of the programmers freedom in exchange for protection from a weird class of hard to track down bugs resulting from mixing APIs of two different versions of a same package - again, we feel that it isn't too wise to trust the package creators to detect this kind of conflicts, since they indeed may apper extremely rarely. Still, the rarer and more cryptic the bug happens to be, the harder it will be for the unlucky person who happens to run into it to correctly track it down. The important message of this section is, that although the imposed rule may be stricter than might be needed in some cases, it is still the best that can be done without making assumptions about installed packages that can't be programmatically checked. In addition, the proposed design creates a reasonable safeguard for situations with a large amount of nested public or peer dependencies, problems with which can be naturally hard to untangle. As we had mentioned peer dependencies, it is worthy of note that they actually lock down verions in very similar fashion, whilst making the dependency tree much less legible in case of a deep level of nesting (more on this in the peer dependency comparison segment).

A certain detrement which emerges from this behaviour is nevertheless present, and will be further elaborated upon at the end of the next section, since it is perhaps more closely related to the matter presented there.

#### Self-containment of public dependencies

In our model, we can say that each dependency, whether it is public or private is self-contained - or in other words - does not assume any information about the outside world. The dependency stores all the neccessary information about packages it needs for it to work either directly in itself, or inherited from one of it's subdependencies - yet, still contained within the subtree of the dependency without polluting the 'global' scope (even if it is only the scope of the parent). You always get the full info required for the current package to install by looking at all of it's children, and you can be sure there is never an additional 'hidden' source of such information.

At first glance, this may seem as a more of a semantic than a functional change - since despite our containment for the current package we nevertheless export the same constrains to the parent as we would have done using peer dependencies. The difference lies in the matter of exporting quite a few more of these because of the way the inheritance works.

While we consider the fact that current setup allows us to comfortably reason about the needed dependencies as important enough, there is still a severe drawback associated with this (and in part with the previous) feature, which also has to be recognized when working on a package manager utilizing our dependency model. Although we indeed gain the ability to resolve each subtree on its own, with all possible conflicts modeled in the form of children of our subtrees root, the state of this root itself is now defined not only by the version of a package it represents, but also by its chosen public dependencies and public subdependencies - or more importantly, by the ones we export as inherted from our root. Firstly, this means that we can have (and in fact may often need) multiple copies of the same package in the same version installed, each one resolved using (and exporting) different public dependencies. Secondly, we can no longer choose to use a installed package as a dependency simply based on it's version - which implies that the ied package manager based approach of differentiating packages based on their content (whether we are taking symbolic links into account or not - the problem may arise at a deeper level) is not avaliable to us anymore.

#### Comparison with peer dependencies

The model is deliberatly designed to take the current node.js packages standard into consideration. In fact, it is fully backwards compatible with npm's model, which deals with necessity for shared dependencie through the concept of peer dependencies (more on the in NPM's section in the third chapter). At first glance, if you compare the behaviour (in terms of satisfiability) of a one level deep public dependency and a peer dependency, it will be exactly the same. This is in part because we want our model to be immediatly useble under present circumstances - with NPM being the standard, literally every javascript package adheres to the rules set by it. It would be foolish to think of coming out of nowhere with an altogether different proposition, especially when the current one (despite the problems presented throughout this paper) works, and expect everyone to jump on a bandwagon. On the other hand, since both the public and the peer dependencies exist to deal with the same issue, namely the need to have shared dependencies (as defined earlier), it isn't all that surprising that both concepts end up working similarly.

Before we continue with the comparion, a short note about peer dependency handling and the change that was made to it moving from NPMv2 to NPMv3 - as mentioned in the chapter three, peer dependencies are no longer installed automatically in the newest NPM version, instead they only serve as a source of warnings and errors. Thus, since packages working previously with v2 had no need to specify the dependencies marked as peer for the second time in their parent, many are not compatible with version 3 (which had it's first public release in september 2015, and started to really spread maybe towards the end of that year). At the time of writing, v2 is still in long term support mode, and probably will be for some time, and project that had not migrated over to NPMv3 are still a plenty. The reason for writing this is that our model is compatible with approaches used in both of the major NPM versions - because of the way we intall the functionality at the lower level, akin to the way of NPMv2, and then export inherited public dependencies which serve as a guideline similar to peer dependencies in NPMv3.

Each kind of resolvable (or unresolvable) dependency tree modeled using public dependencies can be modeled via peer dependencies, while preserving it's resolvability. The same is true for the opposite implication - every hierachy using peer dependencies can be rewritten into public ones without changing the status of it being resolvable. The formal proof of these theses will be presented further down the chapter.

##### Key differences

Let us now round up the main differences between the public and peer dependency approaches.

1. Public dependencies are self-contained

As we had already discussed before, what we consider a huge advantage of our public dependency concept is that all the information neccessary for the successfull installation of the whole subtree is stored only in given subtrees root and it's successors. This is in stark contrast with the design of peer dependencies, which need to make assumptions about the outside world - or maybe even force the outside world to behave according to our subtrees idea, for it to successfully install. We believe this is a bad design decision that makes the dependency tree as a whole harder to reason about, and maybe more importantly harder to satisfy. In fact, even the community behind the NPM is perhaps slowly trying to phase out this concept (TODOcit issue), as may be already seen in softening the requirements of peer dependencies in version 3 of NPM.

2. Public dependencies are exported along other public dependencies

In a certain aspect, peer dependencies are like public ones that are exported only a single step up the dependency tree - in contrast of public dependencies which bubble along any public branch and a single private branch. On the other hand, peer dependencies are copied over the dependency tree in case of a deep nesting. This means that with public dependencies it is computationaly harder to determine whether a conflict exists within the current hierarchy or not, but in turn, it buys us a level of clarity in our dependency tree, whereas peer dependencies 'pollute' every level of nesting they unwind onto.

3. Inherited dependencies are not mandatory

Whilst inherited public dependencies directly resemble peer dependencies in many ways, maybe even more so now in version 3 of NPM, the semantic role between the two is quite different. Each peer dependency essentialy tells us to 'require a certain version of a given dependency to be installed on a parent package' (despite the fact that avoiding any installed version completely only yields an ignorable error) , while the condition set byt an inherited public dependency is a bit softer - 'make sure that if any other dependency for the same package appears on this level, it matches the version set by the public dependency inherited from'.
Of course, when being compared to NPMv2, the difference is obvious, taking older NPMs automatic installation into account - although one may argue that in our concept all of the public dependencies are installed no matter what, which is true and may be considered a drawback (and once again, is a discussion for the implementation chapter). Yet, the installed public dependencies are hidden in the 'local' scope of the dependency which required it, and are exposed to the outside world (in a way that it can access it) only when they are explicitly required there.

### Formalization

We shall now prepare the theoretical apparatus we are going to use to formaly prove the correctness of our approach and the one sided equivalence in regards to the peer dependency based model.

Firstly, let us establish the dependency tree as a graph of nodes - which represent a package resolved in a concrete version - and edges, which represent a dependency requirement, whether it is public or private. In this and the following section, we will be adressing these edges (or relations between nodes) as dependencies, as opposed to the term dependency denoting a required package itself up to this point. Without explicitly stating otherwise, the term dependency will now be used exclusively in this way.

For each node, let us define two sets - privateExports and publicExports. These will mark the dependencies exported from the children of the given node along either a public or private dependency. The privateExports set then stays private to the node, while publicExports is the set exported from it by the means of both public and private dependency. The reference to node itself is contained in it's own publicExports set, which is in turn always a subset of publicExports.

The only source of errors comes from conflicting versions in the privateExports set (naturally, since everything else within the node is a subset of it). The only action available to us during the dependency tree resolution is the choice of version for a given package. In our proofs, we will often make the decision non-deterministically, since we have already proven the NP-completeness of this problem in general.

The characteristics we set out to formally prove are as follows. First and foremost it is the correctness of our proposed design - we will prove by induction that with each step the resolvability assumed by our model satisfies the constrains we have set to require from a resolvable dependency tree. Secondly, we are to prove the one sided transformability of peer dependency model into our concept, TODO-other-side-works-too

Thus, our set of axiom is:

TODO

With TODO(odovdzovacie) rules being:

TODO

### Proof of correctness

## Implementation

### Node module system

### Simulated annealing

TODO continue here! for presentation purposes
Simulated annealing, in general, is a Monte Carlo method of for approximating a global optimum of a given function. It is an adaptation of a slightly older Metropolis-Hastings algorithm, essentially using technique from the area of study of thermodynamics to further improve the chances of it converging to the correct result. It has first apperd in a paper from 1983(cit-anneal).

The method is especially usefull for finding a maximum (or a minimum) of a function which is hard to resolve on the entire domain but can be computed for a single point - or in other words sampled. We will later show how our problem is easily reducible to fit into the described setting. The high-level idea is taken from metallurgy, where the term describes the process of heated metals being allowed to cool down slowly, with their atoms being able to migrate along the crystal lattice. The higher the temperature, the easier it is for an atom to break it's bond and move, thus, as the metal is getting cooler, less and less changes are happening to it's structure. Analogically, when we 'simulate' the annealing process, we will start off with higher temeperature ...blah todo



## Results

TODO - 3 packages - some opensource, wordy, vpm

## Future work

## References

1) https://hal.archives-ouvertes.fr/file/index/docid/149566/filename/ase.pdf

2) https://www.debian.org/doc/manuals/aptitude/ch02s03s01.en.html

3) https://www.techdirt.com/articles/20160112/16582733316/ian-murdock-his-own-words-what-made-debian-such-community-project.shtml

4) https://cseweb.ucsd.edu/~lerner/papers/opium.pdf

5) http://doc.opensuse.org/projects/libzypp/HEAD/

yum) http://dnf.baseurl.org/2015/05/11/yum-is-dead-long-live-dnf/

yum2) http://yum.baseurl.org/api/yum/yum/depsolve.html

predep) https://www.debian.org/doc/debian-policy/ch-relationships.html

port) http://dev.gentoo.org/~zmedico/portage/doc/pt02.html

pkgman) http://blog.ezyang.com/2014/08/the-fundamental-problem-of-programming-language-package-management/

sem) http://semver.org/

modcount) http://www.modulecounts.com/