#multiproject-git-gradle

##Overview

This is gradle script for multi-project git-based setup, configuration and build. It supports automated cloning/pulling 
git-repositories (from any locations, supported by JGit) and typical gradle tasks ("build", "install", etc) that can be
run against some or all projects. Projects can be supplied with inter-project dependencies.

Many thanks to the creators of gradle-git plugin ( https://github.com/ajoberstar/gradle-git ) for their excellent work,
which made is multiproject-git-gradle possible and which is used by it.

##Usage

###Required files

To start using multiproject-git-gradle, you need two files: 

"build.gradle"  - taken from this repository, this is gradle script. 

"config.gradle" - you write it yourself, according to your needs. You can use "config.gradle" from this repository 
as an example/starting-point for editing. See more information on this in chapter
[Configuration syntax](#configuration-syntax).

###Command line syntax

Like with any other gradle script:

```shell
gradle [some task names specified here]
```

See "Supported Tasks" for more information on concrete tasks and their semantics.

###Supported tasks

###Configuration syntax

**More information is coming soon.**
