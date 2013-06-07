#multiproject-git-gradle

##Overview

This is gradle script for multi-project git-based setup, configuration and build. It supports automated cloning/pulling 
git-repositories (from any locations, supported by JGit) and typical gradle tasks ("build", "install", etc) that can be
run against some or all projects. Projects can be supplied with inter-project dependencies, so that particular order 
of assembly/testing/etc is guaranteed.

Many thanks to the creators of gradle-git plugin ( https://github.com/ajoberstar/gradle-git ) for their excellent work,
which made is multiproject-git-gradle possible and which is used by it.

##Required files

To start using multiproject-git-gradle, you need two files: 

"build.gradle"  - taken from this repository, this is gradle script. 

"config.gradle" - you write it yourself, according to your needs. You can use "config.gradle" from this repository 
as an example/starting-point for editing. See more information on this in chapter
[Configuration syntax](#configuration-syntax).

##Command line syntax

Like with any other gradle script:

```shell
gradle [some task names specified here]
```

See [Supported Tasks](#supported-tasks) for more information on concrete tasks and their semantics.

It is possible to start gradle without task:

```shell
gradle
```

then the default task [build](#build-task] is executed.

##Supported tasks

###Build task

Iterates all projects described in [configuration](#configuring-projects), performs the following for each project:

1. Checks whether project exists in the file system. If not, the project is [updated](#update-task) 
(actually, cloned) from [git-location](#configuring-git-locations).

2. Checks whether the project has [build](#configuring-project-build) attributes. If it does not, the project is skipped (not built).

3. Checks whether the project has [dependsOn](#configuring-project-dependencies) attribute. if it does, the dependencies are built first.

4. The project itself is being built, according to [build](#configuring-project-build) attributes.

###Update task

Iterates all projects described in [configuration](#configuring-projects), performs the following for each project:

Checks whether project exists in the file system. 

1. If project does not exist, it is cloned from [git-location](#configuring-git-locations).

2. If project exists, it is pulled from [git-location](#configuring-git-locations).

##Configuration syntax

###Configuring projects

###Configuring git-locations

###Configuring project build

**More information is coming soon.**
