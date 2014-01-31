#multiproject-git-gradle

##Overview

This is gradle script for multiple git-repository setup, configuration and build. It supports automated cloning/pulling git-repositories (from any repositories, supported by JGit) and typical gradle tasks ("build", "clean", etc) that can be
run against some or all repositories. Repositories can be supplied with inter-repository dependencies, so that particular order of assembly/testing/etc is guaranteed.

Many thanks to the creators of gradle-git plugin ( https://github.com/ajoberstar/gradle-git ) for their excellent work, 
which is used by multiproject-git-gradle.

**Content of this document**

* [Required files](#required-files)
* [Command line syntax](#command-line-syntax)
* [Supported tasks](#supported-tasks)
  * [build task](#build-task)
  * [buildApps task](#buildapps-task)
  * [buildExamples task](#buildexamples-task)
  * [clean task](#clean-task)
  * [gitBranchList task](#gitbranchlist-task)
  * [gitStatus task](#gitstatus-task)
  * [update task](#update-task)
  * [uploadArchives task](#uploadarchives-task)
* [Configuration](#configuration)  
  * [Specifying projects](#specifying-projects)
  * [Configuring inter-project dependencies](#configuring-inter-project-dependencies)
  * [Configuring git-repositories](#configuring-git-repositories) 
  * [Configuring build property](#configuring-build-property)
  * [Configuring apps property](#configuring-apps-property)
  * [Configuring examples property](#configuring-examples-property)
* [Copyright and License](#copyright-and-license)

##Required files

To start using multiproject-git-gradle, you need two files: 

"build.gradle"  - taken from this repository, this is gradle script. 

"config.gradle" - you write it yourself, according to your needs. You can use "config.gradle" from this repository 
as an example/starting-point for editing. See more information on this in chapter [Configuration](#configuration).

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

then the default task [buildApps](#buildapps-task) is executed.

##Supported tasks

###build task

build task allows to build multiple gradle projects (from different git-repositories) in an automated way. It does the following:

Iterates all projects described in [configuration](#configuration), performs the following for each project:

1. Checks whether project exists in the file system. If not, the project is cloned from [git-repository](#configuring-git-repositories).

2. Checks whether the project has [build property](#configuring-build-property) and configures project-level "build" task.

3. Checks whether the project has [dependsOn](#configuring-inter-project-dependencies) property. if it does, the dependencies are built first.

4. The project itself is being built, according to [build property](#configuring-build-property).

Note that "build" task does not depend on "update" task, but all "build" steps are performed strictly after 
corresponding "update" steps. That means: 

a) if you routinely run "gradle build", it will not pull the changes from git-sources, but only compile things for you.
Only if some projects are missing, they will be cloned from git-sources.

b) if you run "gradle update build", it is guaranteed, that every project is first updated (cloned or pulled from repository)
and only then built.

###buildApps task

buildApps is an optional task, allowing to build "apps" sub-projects of multiple gradle projects (from different git-repositories) in an automated way. 
It does the following:

First it builds all projects, as described in [build task](#build-task).

Then it iterates all projects described in [configuration](#configuration), performs the following for each project:

1. Checks whether the project has ["apps" property](#configuring-apps-property) and configures project-level "buildApps" task.

2. Tries to perform "gradle build" in [apps sub-folder](#configuring-apps-property) of the project folder.

###buildExamples task

buildExamples is an optional task, allowing to build "examples" sub-projects of multiple gradle projects (from different git-repositories) in an automated way. 
It does the following:

First it builds all projects, as described in [build task](#build-task).

Then it iterates all projects described in [configuration](#configuration), performs the following for each project:

1. Checks whether the project has [examples](#configuring-examples-property) property. If it does not, the project is skipped (not built).

2. Tries to perform "gradle build" in [examples sub-folder](#configuring-examples-property) of the project folder.

###clean task

clean task allows to clean multiple projects (from different git-repositories) in an automated way.

###gitBranchList task

TODO: document this task!

###gitStatus task

TODO: document this task!

###update task

update task allows to clone/pull multiple projects (not necessarily gradle-projects) from git-sources in an automated way. It does the following:

Iterates all projects described in [configuration](#configuration), checks each project, whether it exists, then:

1. If project does not exist, it is cloned from [git-repository](#configuring-git-repositories).

2. If project exists, it is pulled from [git-repository](#configuring-git-repositories).

###uploadArchives task

TODO: document this task!

##Configuration

"build.gradle" (of multiproject-git-gradle) reads "config.gradle" file in order to "understand" project structure.
"config.gradle" is being interpreted by gradle, therefore it should comply to gradle syntax.

Absolutely minimal version of "config.gradle" (which is useless, but "well-formed" in a sence 
of multiproject-git-gradle) looks like this:

```groovy
multiproject {
}
```

###Specifying projects

The simplest usable configuration looks like this:

```groovy
multiproject {
  project name: 'ProjectA'
  project name: 'ProjectB'
}
```

where ProjectA and ProjectB must designate existing subfolders of the current folder.

**Effect:** when you run "gradle build", multiproject-git-gradle will consequently build each project specified in multiproject configuration.
 
###Configuring inter-project dependencies

You can specify inter-project dependencies the following way:

```groovy
multiproject {
  project name: 'ProjectA'
  project name: 'ProjectB', dependsOn: 'ProjectA'
  project name: 'ProjectC', dependsOn: 'ProjectA'
  project name: 'ProjectD', dependsOn: [ 'ProjectB', 'ProjectC' ]
}
```

Rules:

* dependsOn can be specified as a string or an array of strings.
* dependsOn refers to projects (not tasks).
* dependsOn defines the order in which projects are updated (cloned/pulled from their git-repositories) and built.
* dependsOn is transitive. In the example above, "ProjectD" directly depends on "ProjectB", "ProjectC" and, indirectly,
on "ProjectA".

###Configuring git-repositories

There are two ways to specify git-repositories: via gitBase property and via gitSource property.

####Configuring gitBase property

gitBase specifies "base" URL, from where the project(s) come. It supports all protocols supported by JGit library
(for example, "http", "https", "git", "ssh"). gitBase can be specified "globally", for project group or for individual projects:

```groovy
multiproject {
  gitBase = 'https://github.com/someUser'
  project name: 'ProjectA'
  git gitBase: 'https://github.com/anotherUser', {
    project name: 'ProjectB'
    project name: 'ProjectC'
  }
  project name: 'ProjectD', gitBase: 'https://github.com/thirdUser'
}
```

Rules:

* Whenever a project has gitBase property, the script will use it for calculating git-repository URI the following way:
URI = gitBase + "/" + name + ".git"

* Whenever a project is enclosed by "git" group, defining gitBase property, the script will use it for calculating git-repository.

* Whenever a project does not have gitBase property and is not enclosed by "git" group, the script will use "global" gitBase property for calculating git-repository.

In the concrete example (above) "ProjectA" will be cloned/pulled from "https://github.com/someUser/ProjectA.git",
"ProjectB" will be cloned/pulled from "https://github.com/anotherUser/ProjectB.git", "ProjectC" will be cloned/pulled from 
"https://github.com/anotherUser/ProjectC.git" and "ProjectD" will be cloned/pulled from "https://github.com/thirdUser/ProjectD.git".

####Configuring gitSource property

gitSource property represents complete URI to git-repository and is not combined with anything.
It supports all protocols supported by JGit library (for example, "http", "https", "git", "ssh").

```groovy
multiproject {
  gitBase = 'https://github.com/someUser'
  project name: 'ProjectA'
  project name: "ProjectB", gitSource: "https://anotherdomain.com/someDifferentProjectName.git"
}
```

Rules:

* Whenever a project has gitSource property, the script will use it as complete URI for cloning/pulling.
gitBase property (either per-project, per-group or global) is ignored for such project.

* gitSource can be specified only for individual projects, there is no global gitSource property.

In the concrete example (above) "ProjectA" will be cloned/pulled from "https://github.com/someUser/ProjectA.git",
"ProjectB" will be cloned/pulled from "https://anotherdomain.com/someDifferentProjectName.git".

####Configuring gitNameSeparator and gitNameSuffix

By default, if a project does not have gitSource property, it's effective git repository URI is calculated
by formula: URI = gitBase + "/" + name + ".git"
You can fine-tune this formula by specifying gitNameSeparator and gitNameSuffix: either globally, per-group or per-project:

```groovy
multiproject {
  gitBase = 'https://github.com/someUser'
  project name: 'ProjectA'
  git gitBase: 'git@someGitoliteRepository', gitNameSeparator: ':', gitNameSuffix: '', {
    project name: 'ProjectB'
    project name: 'ProjectC'
  }
}
```

In the concrete example (above) "ProjectA" will be cloned/pulled from "https://github.com/someUser/ProjectA.git",
"ProjectB" will be cloned/pulled from "git@someGitoliteRepository:ProjectB", "ProjectC" will be cloned/pulled from 
"git@someGitoliteRepository:ProjectC".

###Configuring build property

By default, multiproject-git-gradle builds all projects specified in multiproject configuration.
You can fine-tune what and how is built by using "build" property:

```groovy
multiproject {
  project name: 'ProjectA'
  project name: 'ProjectB', build: false
  project name: 'ProjectC', build: 'subFolder'
}
```

Rules:

* Whenever a project does not have "build" property, gradle script will run "build" task against projectDir folder

* Whenever a project has "build" property and it evaluates to false, gradle script will not build the given project

* Whenever a project has "build" property and it evaluates to string, gradle script will run "build" task against the combined folder "$projectDir/$build".

In the concrete example (above) "ProjectA" will be built in "ProjectA" subfolder,
"ProjectB" will not be built, "ProjectC" will be built in "ProjectC/subFolder" subfolder.

###Configuring apps property

If you specify "apps" property for a given project, multiproject-git-gradle defines additional task "buildApps",
which can be called on command line as "gradle buildApps".

```groovy
multiproject {
  project name: 'ProjectA', apps: true
  project name: 'ProjectB', apps: "applications"
}
```

Rules:

* Whenever a project has "apps" property and it evaluates to true, gradle script defines "buildApps" task, 
which will be run against the combined folder "$projectDir/apps".

* Whenever a project has "apps" property and it evaluates to string, gradle script defines "buildApps" task, 
which will be run against the combined folder "$projectDir/$apps".

* Otherwise "buildApps" task is not defined for the given project.

###Configuring examples property

If you specify "examples" property for a given project, multiproject-git-gradle defines additional task "buildExamples",
which can be called on command line as "gradle buildExamples".

```groovy
multiproject {
  project name: 'ProjectA', examples: true
  project name: 'ProjectB', examples: "samples"
}
```

Rules:

* Whenever a project has "examples" property and it evaluates to true, gradle script defines "buildExamples" task, 
which will be run against the combined folder "$projectDir/examples".

* Whenever a project has "examples" property and it evaluates to string, gradle script defines "buildExamples" task, 
which will be run against the combined folder "$projectDir/$examples".

* Otherwise "buildExamples" task is not defined for the given project.

##Copyright and License

Copyright 2013 (c) Andrey Hihlovskiy

All versions, present and past, of "multiproject-git-gradle" script are licensed under MIT license:

* [MIT](http://opensource.org/licenses/MIT)

You are encouraged to use it to whatever purpose and whichever way, all for free, provided that you retain copyright 
notice at the beginning of the script.
