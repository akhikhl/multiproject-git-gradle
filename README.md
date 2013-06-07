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

then the default task [build](#build-task) is executed.

##Supported tasks

###build task

build task allows to build multiple gradle projects in an automated way. It does the following:

Iterates all projects described in [configuration](#configuring-projects), performs the following for each project:

1. Checks whether project exists in the file system. If not, the project is cloned from [git-location](#configuring-git-locations).

2. Checks whether the project has [build](#configuring-project-build) attribute. If it does not, the project is skipped (not built).

3. Checks whether the project has [dependsOn](#configuring-project-dependencies) attribute. if it does, the dependencies are built first.

4. The project itself is being built, according to [build](#configuring-project-build) attribute.

Note that "build" task does not depend on "update" task, but all "build" steps are performed strictly after 
corresponding "update" steps. That means: 

a) if you routinely run "gradle build", it will not pull the changes from git-sources, but only compile things for you.
Only if some projects are missing, they will be cloned from git-sources.

b) if you run "gradle update build", it is guaranteed, that every project is first updated (pulled from repository)
and only then built.

###update task

update task allows to clone/pull multiple projects (not necessarily gradle-projects) from git-sources in an automated way. It does the following:

Iterates all projects described in [configuration](#configuring-projects), checks each project, whether it exists, then:

1. If project does not exist, it is cloned from [git-location](#configuring-git-locations).

2. If project exists, it is pulled from [git-location](#configuring-git-locations).

###buildExamples task

buildExamples task allows to build multiple "example" gradle projects in an automated way. It does the following:

First it builds all projects, as described in [build task](#build-task).

Then it iterates all projects described in [configuration](#configuring-projects), performs the following for each project:

1. Checks whether the project has [examples](#configuring-project-examples) attribute. If it does not, the project is skipped (not built).

2. Tries to perform "gradle build" in [examples sub-folder](#configuring-project-examples) of the project folder.

Note that "buildExamples" task depends on "build" task, but does not depend on "update" task.

##Configuration syntax

###Configuring projects

###Configuring git-locations

###Configuring project build

**More information is coming soon.**
