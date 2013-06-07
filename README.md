#multiproject-git-gradle

##Overview

This is gradle script for multi-project git-based setup, configuration and build. It supports automated cloning/pulling 
git-repositories (from any repositories, supported by JGit) and typical gradle tasks ("build", "install", etc) that can be
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

build task allows to build multiple gradle projects (from different git-repositories) in an automated way. It does the following:

Iterates all projects described in [configuration](#configuring-projects), performs the following for each project:

1. Checks whether project exists in the file system. If not, the project is cloned from [git-repository](#configuring-git-repositories).

2. Checks whether the project has [build](#configuring-project-build) property. If it does not, the project is skipped (not built).

3. Checks whether the project has [dependsOn](#configuring-project-dependencies) property. if it does, the dependencies are built first.

4. The project itself is being built, according to [build](#configuring-project-build) property.

Note that "build" task does not depend on "update" task, but all "build" steps are performed strictly after 
corresponding "update" steps. That means: 

a) if you routinely run "gradle build", it will not pull the changes from git-sources, but only compile things for you.
Only if some projects are missing, they will be cloned from git-sources.

b) if you run "gradle update build", it is guaranteed, that every project is first updated (pulled from repository)
and only then built.

###update task

update task allows to clone/pull multiple projects (not necessarily gradle-projects) from git-sources in an automated way. It does the following:

Iterates all projects described in [configuration](#configuring-projects), checks each project, whether it exists, then:

1. If project does not exist, it is cloned from [git-repository](#configuring-git-repositories).

2. If project exists, it is pulled from [git-repository](#configuring-git-repositories).

###buildExamples task

buildExamples task allows to build multiple "example" gradle projects in an automated way. It does the following:

First it builds all projects, as described in [build task](#build-task).

Then it iterates all projects described in [configuration](#configuring-projects), performs the following for each project:

1. Checks whether the project has [examples](#configuring-project-examples) property. If it does not, the project is skipped (not built).

2. Tries to perform "gradle build" in [examples sub-folder](#configuring-project-examples) of the project folder.

Note that "buildExamples" task depends on "build" task, but does not depend on "update" task.

##Configuration syntax

"build.gradle" (of multiproject-git-gradle) reads "config.gradle" file in order to "understand" project structure.
"config.gradle" is being interpreted by gradle, therefore it should comply to gradle syntax.

Absolutely minimal version of "config.gradle" (which is useless, but "well-formed" in a sence 
of multiproject-git-gradle) looks like this:

```groovy
ext {
  projects = []
}
```

###Configuring projects

The simplest usable configuration looks like this:

```groovy
ext {
  gitBase = "https://github.com/someUser"
  projects = [ "ProjectA", "ProjectB", ... ]
}
```

(URL and project names here and below are fictitious, for demonstration only).

Note new things:

* "projects" element contains project names as strings.
* "gitBase" property specifies from where to get projects (see more information on this in chapter [configuring git-repositories](#configuring-git-repositories)).

Implied semantics:

* Each project will be cloned/pulled from the address, calculated as gitBase + "/" + projectName + ".git". For example,
"ProjectA" will be cloned/pulled from "https://github.com/someUser/ProjectA.git".
* Projects, as they declared in the example above, could not be built by multiproject-git-gradle. The script "understands"
how to clone/pull these projects and that's it. See more information in chapter [configuring project build](#configuring-project-build).

###Configuring git-repositories

There are two ways to specify git-repositories: via gitBase property and via gitSource property.

####Configuring gitBase property

gitBase can be specified "globally" or for individual projects:

```groovy
ext {
  gitBase = "https://github.com/someUser"
  projects = [ 
    "ProjectA", 
    [ name: "ProjectB", gitBase: "https://github.com/anotherUser" ]
    ... 
  ]
}
```

Implied semantics:

* Whenever a project has gitBase property, the script will use it for calculating git-repository.
* Whenever a project does not have gitBase property, the script will use "global" gitBase property for calculating git-repository.

In this concrete example (above) "ProjectA" will be cloned/pulled from "https://github.com/someUser/ProjectA.git",
"ProjectB" will be cloned/pulled from "https://github.com/anotherUser/ProjectB.git".

####Configuring gitSource property

gitSource property represents complete URI to git-repository and is not combined with anything: 

```groovy
ext {
  gitBase = "https://github.com/someUser"
  projects = [ 
    "ProjectA", 
    [ name: "ProjectB", gitBase: "https://github.com/anotherUser" ],
    [ name: "ProjectC", gitSource: "git@github.com:thirdUser/someDifferentProjectName.git" ]
    ... 
  ]
}
```

Implied semantics:

* Whenever a project has gitSource property, the script will use it as complete URI for cloning/pulling.
gitBase property (either per-project or global) is ignored for such project.

* gitSource can be specified only for individual projects, there is no global gitSource property.

###Configuring project build

###Configuring project examples

**More information is coming soon.**
