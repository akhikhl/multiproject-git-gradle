#multiproject-git-gradle

##Overview

This is gradle script for multi-project git-based setup, configuration and build. It supports automated cloning/pulling 
git-repositories (from any repositories, supported by JGit) and typical gradle tasks ("build", "install", etc) that can be
run against some or all projects. Projects can be supplied with inter-project dependencies, so that particular order 
of assembly/testing/etc is guaranteed.

Many thanks to the creators of gradle-git plugin ( https://github.com/ajoberstar/gradle-git ) for their excellent work,
which made is multiproject-git-gradle possible and which is used by it.

**Content of this document**

* [Required files](#required-files)
* [Command line syntax](#command-line-syntax)
* [Supported tasks](#supported-tasks)
  * [build task](#build-task)
  * [update task](#update-task)
  * [buildExamples task](#build-examples-task)
* [Configuration syntax](#configuration-syntax)


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
* "gitBase" property specifies from where to get projects.

Implied semantics:

* Each project will be cloned/pulled from the relevant git repository. For example,
"ProjectA" will be cloned/pulled from "https://github.com/someUser/ProjectA.git"
 (see more information on this in chapter [configuring git-repositories](#configuring-git-repositories)).
* Projects, as they declared in the example above, could not be built by multiproject-git-gradle. The script "understands"
how to clone/pull these projects and that's it. See more information in chapter [configuring project build](#configuring-project-build).

Another example shows that projects can be specified as strings or objects:

```groovy
ext {
  gitBase = "https://github.com/someUser"
  projects = [
    "ProjectA",
    [ name: "ProjectB" ],
    [ name: "ProjectC", build: true ]
  ]
}
```

###Configuring git-repositories

There are two ways to specify git-repositories: via gitBase property and via gitSource property.

####Configuring gitBase property

gitBase specifies "base" URL, from where the projects come. It supports all protocols supported by JGit library
(for example, "http", "https", "git", "ssh"). gitBase can be specified "globally" or for individual projects:

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

* Whenever a project has gitBase property, the script will use it for calculating git-repository URI the following way:
URI = gitBase + "/" + name + ".git"
* Whenever a project does not have gitBase property, the script will use "global" gitBase property for calculating git-repository.

In this concrete example (above) "ProjectA" will be cloned/pulled from "https://github.com/someUser/ProjectA.git",
"ProjectB" will be cloned/pulled from "https://github.com/anotherUser/ProjectB.git".

####Configuring gitSource property

gitSource property represents complete URI to git-repository and is not combined with anything.
It supports all protocols supported by JGit library (for example, "http", "https", "git", "ssh").

```groovy
ext {
  gitBase = "https://github.com/someUser"
  projects = [ 
    "ProjectA", 
    [ name: "ProjectB", gitBase: "https://github.com/anotherUser" ],
    [ name: "ProjectC", gitSource: "https://anotherdomain.com/someDifferentProjectName.git" ]
    ... 
  ]
}
```

Implied semantics:

* Whenever a project has gitSource property, the script will use it as complete URI for cloning/pulling.
gitBase property (either per-project or global) is ignored for such project.

* gitSource can be specified only for individual projects, there is no global gitSource property.

In this concrete example (above) "ProjectA" will be cloned/pulled from "https://github.com/someUser/ProjectA.git",
"ProjectB" will be cloned/pulled from "https://github.com/anotherUser/ProjectB.git", "ProjectC" from 
"https://anotherdomain.com/someDifferentProjectName.git".

###Configuring project build

####Simple build

The simplest way to configure project for build is to supply it with property "build=true":

```groovy
ext {
  gitBase = "https://github.com/someUser"
  projects = [
    "ProjectA",
    [ name: "ProjectB", build: true ],
    [ name: "ProjectC", build: true ]
  ]
}
```

Implied semantics:

* Whenever [build](#build-task) task is being performed, the script will try to run "gradle build" in the sub-folders
"ProjectB" and "ProjectC". The order, in which builds are performed, is undefined (but can be fixed via 
[dependsOn](#configuring-project-dependencies) property).

####Build within particular project subfolder

Sometimes you need to specify which subfolder of the project folder should be built:


```groovy
ext {
  gitBase = "https://github.com/someUser"
  projects = [
    "ProjectA",
    [ name: "ProjectB", build: true ],
    [ name: "ProjectC", build: "libs" ]
  ]
}
```

Implied semantics:

* Whenever [build](#build-task) task is being performed, the script will try to run "gradle build" in the sub-folders
"ProjectB" and "ProjectC/libs".

####Specifying build tasks

Some artifacts should be installed into maven/ivy repository, not simply built. For those you can specify buildTasks:

```groovy
ext {
  gitBase = "https://github.com/someUser"
  projects = [
    "ProjectA",
    [ name: "ProjectB", build: true ],
    [ name: "ProjectC", build: "libs", buildTasks: [ "install" ] ]
  ]
}
```

Implied semantics:

* buildTasks property, when specified, must be an array, containing one or more task names.
* buildTasks should be recognized by project-specific gradle script(s). For example, "ProjectC" 
must apply gradle-maven plugin, in order to "understand" install task.
* order, in which buildTasks are performed within the given project, is completely defined by project-specific
script, not by multiproject-git-gradle.
* when buildTasks property is omitted, multiproject-git-gradle performs "build" task against the given project.
 
###Configuring project dependencies

You can specify inter-project dependencies the following way:

```groovy
ext {
  gitBase = "https://github.com/someUser"
  projects = [
    "ProjectA",
    [ name: "ProjectB", build: true, dependsOn: "ProjectA" ],
    [ name: "ProjectC", build: true, dependsOn: [ "ProjectA" ] ]
    [ name: "ProjectD", build: true, dependsOn: [ "ProjectB", "ProjectC" ] ]
  ]
}
```

Implied semantics:

* dependsOn can be specified as a string or an array of strings.
* dependsOn refers to projects (not tasks).
* dependsOn defines the order in which projects are updated (cloned/pulled from their git-repositories) and built.
* dependsOn is transitive. In the example above, "ProjectD" directly depends on "ProjectB", "ProjectC" and, indirectly,
on "ProjectA".

###Configuring project examples

####Simple examples

The simplest way to configure project examples for build is to supply it with property "examples=true":

```groovy
ext {
  gitBase = "https://github.com/someUser"
  projects = [
    "ProjectA",
    [ name: "ProjectB", build: true, examples: true ],
    [ name: "ProjectC", build: true, examples: true ]
  ]
}
```

Implied semantics:

* Whenever [buildExamples](#build-examples-task) task is being performed, the script will try to run "gradle build" 
in the sub-folders "ProjectB/examples" and "ProjectC/examples". 
* buildExamples task for every project depends on build task for that project.
* The order, in which example builds are performed (relative to each other), is undefined.

####Build examples within particular project subfolder

Sometimes you need to specify which subfolder of the project folder contains examples:


```groovy
ext {
  gitBase = "https://github.com/someUser"
  projects = [
    "ProjectA",
    [ name: "ProjectB", build: true, examples: true ],
    [ name: "ProjectC", build: true, examples: "somewhere/inTheProjectTree" ]
  ]
}
```

Implied semantics:

* Whenever [buildExamples](#build-examples-task) task is being performed, the script will try to run "gradle build"
in the sub-folders "ProjectB/examples" and "ProjectC/somewhere/inTheProjectTree".

**More information is coming soon.**
