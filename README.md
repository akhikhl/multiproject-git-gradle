# multiproject-git-gradle

## Overview

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
  * [release task](#release-task)
* [Configuration](#configuration)  
  * [Specifying projects](#specifying-projects)
  * [Configuring inter-project dependencies](#configuring-inter-project-dependencies)
  * [Configuring git-repositories](#configuring-git-repositories) 
  * [Configuring build property](#configuring-build-property)
  * [Configuring apps property](#configuring-apps-property)
  * [Configuring examples property](#configuring-examples-property)
  * [Configuring skipTests property](#configuring-skiptests-property)
* [Automated Release Feature (ARF)](#automated-release-feature)
  * [ARF Requirements](#arf-requirements)
  * [How to start using ARF](#how-to-start-using-arf)
  * [ARF side effects](#arf-side-effects)
  * [ARF release versions](#arf-release-versions)
  * [ARF new versions](#arf-new-versions)
  * [DSL for custom version increments](#dsl-for-custom-version-increments)
  * [releaseNoPush property](#releasenopush-property)
  * [releaseNoCommit property](#releasenocommit-property)
  * [Publishing to maven repositories](#publishing-to-maven-repositories)
    * [Prerequisites](#prerequisites)
    * [Publishing to local maven repo](#publishing-to-local-maven-repo)
    * [Publishing to remote maven repo (artifactory)](#publishing-to-remote-maven-repo-artifactory)
* [Copyright and License](#copyright-and-license)

## Required files

To start using multiproject-git-gradle, you need two files: 

"build.gradle"  - taken from this repository, this is gradle script. 

"config.gradle" - you write it yourself, according to your needs. You can use "config.gradle" from this repository 
as an example/starting-point for editing. See more information on this in chapter [Configuration](#configuration).

## Command line syntax

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

## Supported tasks

### build task

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

### buildApps task

buildApps is an optional task, allowing to build "apps" sub-projects of multiple gradle projects (from different git-repositories) in an automated way. 
It does the following:

First it builds all projects, as described in [build task](#build-task).

Then it iterates all projects described in [configuration](#configuration), performs the following for each project:

1. Checks whether the project has ["apps" property](#configuring-apps-property) and configures project-level "buildApps" task.

2. Tries to perform "gradle build" in [apps sub-folder](#configuring-apps-property) of the project folder.

### buildExamples task

buildExamples is an optional task, allowing to build "examples" sub-projects of multiple gradle projects (from different git-repositories) in an automated way. 
It does the following:

First it builds all projects, as described in [build task](#build-task).

Then it iterates all projects described in [configuration](#configuration), performs the following for each project:

1. Checks whether the project has [examples](#configuring-examples-property) property. If it does not, the project is skipped (not built).

2. Tries to perform "gradle build" in [examples sub-folder](#configuring-examples-property) of the project folder.

### clean task

clean task allows to clean multiple projects (from different git-repositories) in an automated way.

###gitBranchList task

TODO: document this task!

### gitStatus task

TODO: document this task!

### update task

update task allows to clone/pull multiple projects (not necessarily gradle-projects) from git-sources in an automated way. It does the following:

Iterates all projects described in [configuration](#configuration), checks each project, whether it exists, then:

1. If project does not exist, it is cloned from [git-repository](#configuring-git-repositories).

2. If project exists, it is pulled from [git-repository](#configuring-git-repositories).

### uploadArchives task

TODO: document this task!

### release task

This task performs full release process on multiple git projects.

See more information at [Automated Release Feature (ARF)](#automated-release-feature).

## Configuration

"build.gradle" (of multiproject-git-gradle) reads "config.gradle" file in order to "understand" project structure.
"config.gradle" is being interpreted by gradle, therefore it should comply to gradle syntax.

Absolutely minimal version of "config.gradle" (which is useless, but "well-formed" in a sence 
of multiproject-git-gradle) looks like this:

```groovy
multiproject {
}
```

### Specifying projects

The simplest usable configuration looks like this:

```groovy
multiproject {
  project name: 'ProjectA'
  project name: 'ProjectB'
}
```

where ProjectA and ProjectB must designate existing subfolders of the current folder.

**Effect:** when you run "gradle build", multiproject-git-gradle will consequently build each project specified in multiproject configuration.
 
### Configuring inter-project dependencies

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

### Configuring git-repositories

There are two ways to specify git-repositories: via gitBase property and via gitSource property.

#### Configuring gitBase property

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

#### Configuring gitSource property

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

#### Configuring gitNameSeparator and gitNameSuffix

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

### Configuring build property

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

### Configuring apps property

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

### Configuring examples property

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

### Configuring skipTests property

Boolean property skipTests allows to enable and disable unit-tests on all projects or individual projects:

```groovy
multiproject {
  skipTests = true
  project name: 'ProjectA'
  project name: 'ProjectB', skipTests: false
}
```

By default skipTests is globally set to false, i.e. unit-tests are enabled.

## Automated Release Feature

Since version 1.0.12 multiproject-git-gradle implements Automated Release Feature (AFR), which allows to release the software from multiple git repositories.

AFR is implemented as part of multiproject-git-gradle and accessible as a single task: "release". As with other tasks of multiproject-git-gradle, the file "config.gradle" defines upstream git repositories and inter-repository dependencies.

### ARF Requirements

1. All involved git repositories are required to have root "build.gradle", which includes [townsfolk/gradle-release](https://github.com/townsfolk/gradle-release) plugin.

2. All involved git repositories are required to have "gradle.properties" file, containing at least "version" property. 

3. AFR uses and updates versions exlusively via "gradle.properties" file. It is expected that none of "build.gradle" files (either at the roots of git-repositories or in subprojects) contain any "hardcoded" dependency versions to the artifacts of another git repository. Any inter-repository dependencies should be managed via dependencies of kind: compile "group:artifact:${repoName}_version", where "${repoName}_version" is defined as property in "gradle.properties" and repoName is the name of existing git repository listed in "config.gradle".

### How to start using ARF

1. Add townsfolk/gradle-release plugin to root "build.gradle" of every git repository.

2. Add "gradle.properties" with at least "version" property to every git repository.

3. Convert all inter-repository dependencies to managed dependencies with versions in "gradle.properties" in the format "${repoName}_version=versionValue".

### ARF side effects

1. ARF performs all standard checks (implemented by townsfolk/gradle-release plugin) against all involved git repositories. If any check fails, the whole release process is cancelled and error message is shown.

2. ARF modifies "gradle.properties" files of all involved git repositories, replacing current versions with release versions. See section [ARF release versions](#arf-release-versions) for details.

3. ARF checks for the presence of snapshot dependencies in all involved git repositories. If some git repositories contain snapshot dependencies, the whole release process is cancelled and error message is shown.

4. ARF builds code in all involved git repositories. The inter-repository order of build is defined by "config.gradle".

5. ARF performs the following in each git repository: creates a commit, creates a release tag and pushes to the origin. Push can be disabled by [releaseNoPush property](#releasenopush-property). Commit can be disabled by [releaseNoCommit property](#releasenocommit-property).

6. ARF modifies "gradle.properties" files of all involved git repositories, replacing current versions with new versions. See section [ARF new versions](#arf-new-versions) for details.

7. ARF performs the following in each git repository: creates a commit and pushes to the origin. Push can be disabled by [releaseNoPush property](#releasenopush-property). Commit can be disabled by [releaseNoCommit property](#releasenocommit-property).

8. ARF restores the changed "gradle.properties" files to their original state in case of errors.

9. ARF can clone some or all repositories (mentioned in "config.gradle") from upstream locations, if they are not already present in the root directory.

10. If some repositories are already present in the root directory, automated release feature will perform git-pull on them.

### ARF release versions

ARF uses the following algorithm for defining release version:

It takes "version" property from the given project and matches it against regex:

```regex
/(\d+)([^\d]*$)/
```

if project version matches this pattern, the release version is calculated as:

```groovy
m.replaceAll(m[0][1])
```

where m is an object of class java.util.regex.Matcher (the result of regex match).

Example: if original version is "1.0-SNAPSHOT", the calculated release version will be "1.0".

### ARF new versions

ARF uses the following algorithm for defining new version:

It takes "version" property from the given project and matches it against regex:

```regex
/(\d+)([^\d]*$)/
```

if project version matches this pattern, the new version is calculated as:

```groovy
m.replaceAll("${ (m[0][1] as int) + 1 }${ m[0][2] }") }
```

where m is an object of class java.util.regex.Matcher (the result of regex match).

Example: if original version is "1.0-SNAPSHOT", the calculated new version will be "1.1-SNAPSHOT".

### DSL for custom version increments

You can define your own rules for release version and new version with the help of multiproject-git-gradle DSL:

```groovy
multiproject {
  git baseDir: 'upstream_repos', {
    project name: 'project1'
    project name: 'project3'
    project name: 'project2', dependsOn: [ 'project1', 'project3' ], {
      releaseVersion(/(\d+)([^\d]*$)/, { m -> m.replaceAll("${ m[0][1] }-RELEASE") })
      newVersion(/(\d+)([^\d]*$)/, { m -> m.replaceAll("${ m[0][1] }-UNSTABLE") })
    }
  }
}
```

in this example we define that project2 gets release versions that look like "X.Y-RELEASE" and new versions that look like "X.Y-UNSTABLE".

releaseVersion and newVersion elements are actually functions, accepting two arguments: first one is regex, second one is closure.

Closure is only invoked when the given regex is successfully matched against "version" property of "gradle.properties" file. Closure receives an object of class java.util.regex.Matcher (the result of regex match) as a parameter. Closure must return a string (or an object meaningfully convertible to string) containing version.

It is possible to specify "global" releaseVersion and newVersion elements:

```groovy
multiproject {
  releaseVersion(/(\d+)([^\d]*$)/, { m -> m.replaceAll("${ m[0][1] }-release") })
  newVersion(/(\d+)([^\d]*$)/, { m -> m.replaceAll("${ m[0][1] }-unstable") })
  git baseDir: 'upstream_repos', {
    project name: 'project1'
    project name: 'project3'
    project name: 'project2', dependsOn: [ 'project1', 'project3' ], {
      releaseVersion(/(\d+)([^\d]*$)/, { m -> m.replaceAll("${ m[0][1] }-RELEASE") })
      newVersion(/(\d+)([^\d]*$)/, { m -> m.replaceAll("${ m[0][1] }-UNSTABLE") })
    }
  }
}
```

in this example we define that project1 and project3 get gets release versions that look like "X.Y-release" and new versions that look like "X.Y-unstable", while project2 gets release versions that look like "X.Y-RELEASE" and new versions that look like "X.Y-UNSTABLE".

It is possible to specify multiple releaseVersion and newVersion elements in the same scope, each one with it's own regex and closure.

If there are multiple releaseVersion elements with the same regex in the same scope, only the last one is used, the previous ones are ignored.

If there are multiple newVersion elements with the same regex in the same scope, only the last one is used, the previous ones are ignored.

### releaseNoPush property

releaseNoPush boolean property could be defined either in context of multiproject element or in context of project element:

```groovy
multiproject {
  releaseNoPush = true
  git baseDir: 'upstream_repos', {
    project name: 'project1', releaseNoPush: true
    project name: 'project3'
    project name: 'project2', dependsOn: [ 'project1', 'project3' ]
  }
}
```

when releaseNoPush=true, ARF commits release and new versions, but does not push anything upstream.

### releaseNoCommit property

releaseNoCommit boolean property could be defined either in context of multiproject element or in context of project element:

```groovy
multiproject {
  releaseNoCommit = true
  git baseDir: 'upstream_repos', {
    project name: 'project1', releaseNoCommit: true
    project name: 'project3'
    project name: 'project2', dependsOn: [ 'project1', 'project3' ]
  }
}
```

when releaseNoCommit=true, ARF does not commit nor push release and new versions.

### Publishing to maven repositories

Since version 1.0.25 multiproject-git-gradle supports publishing both to local maven repo and any remote maven repo (like artifactory). Below are the detailed instructions.

#### Prerequisites

- set of git repositories, each git repository contains one or more JVM-based projects

- those projects, that are intended for publishing to local or remote maven repo, must have an instruction: 

  ```groovy
  apply plugin: 'maven'
  ```
  
  Note that applying maven plugin only at root project will not work in case of nested projects. You'll need to add `apply plugin: 'maven'` to every subproject that needs to be published. This can be automated via:
  
  ```groovy
  subprojects {
    apply plugin: 'maven'
  }
  ```

#### Publishing to local maven repo

1. In "config.gradle": add releaseDeployTasks property to either multiproject element or to project element:

```groovy
multiproject {
  releaseDeployTasks = ['install']
  git baseDir: 'upstream_repos', {
    project name: 'project1'
    project name: 'project3'
    project name: 'project2', dependsOn: [ 'project1', 'project3' ]
  }
}
```

2. Invoke on command line: `gradle release`.

**Effect**: the release versions and new versions will be installed to local maven repo (~/.m2) after the commit to git. The exact sequence is:

- prepare release version
- commit release version to git
- publish release version to local maven repo
- prepare new version
- commit new version to git
- publish new version to local maven repo

#### Publishing to remote maven repo (artifactory)

1. In "config.gradle": add releaseDeployTasks property to either multiproject element or to project element:

```groovy
multiproject {
  releaseDeployTasks = ['uploadArchives']
  git baseDir: 'upstream_repos', {
    project name: 'project1'
    project name: 'project3'
    project name: 'project2', dependsOn: [ 'project1', 'project3' ]
  }
}
```

2. Create new file "~/.gradle/init.d/deploy.gradle", insert code:

```groovy
rootProject {
  ext {
    artifactoryReleases = [ url: 'protocol-host-and-port/artifactory/libs-release-local', user: 'UploadUser', password : 'UploadPassword' ]
    artifactorySnapshots = [ url: 'protocol-host-and-port/artifactory/libs-snapshot-local', user: 'UploadUser', password : 'UploadPassword' ]
  }
}

afterProject { proj ->

  if(proj.plugins.findPlugin('maven') && proj.uploadArchives instanceof org.gradle.api.tasks.Upload) {
  
    proj.configurations {
      deployerJars
    }

    proj.dependencies {
      deployerJars 'org.apache.maven.wagon:wagon-http:2.4'
    }
    
    proj.uploadArchives {
      repositories.mavenDeployer {
        configuration = proj.configurations.deployerJars
        if(proj.version.contains('-SNAPSHOT'))
          repository(url: rootProject.ext.artifactorySnapshots.url) {
            authentication(userName: rootProject.ext.artifactorySnapshots.user, password: rootProject.ext.artifactorySnapshots.password)
          }
        else
          repository(url: rootProject.ext.artifactoryReleases.url) {
            authentication(userName: rootProject.ext.artifactoryReleases.user, password: rootProject.ext.artifactoryReleases.password)
          }
      }
    }
  }
}
```

where "protocol-host-and-port" should be replaced with concrete protocol, host and port of target maven repository, "UploadUser" should be replaced with an existing user and "UploadPassword" should be replaced with a valid password.

3. Invoke on command line: `gradle release`.

**Effect**: the release versions and new versions will be deployed to remote maven repo after the commit to git. The exact sequence is:

- prepare release version
- commit release version to git
- publish release version to remote maven repo
- prepare new version
- commit new version to git
- publish new version to remote maven repo

## Copyright and License

Copyright 2013 (c) Andrey Hihlovskiy

All versions, present and past, of "multiproject-git-gradle" script are licensed under MIT license:

* [MIT](http://opensource.org/licenses/MIT)

You are encouraged to use it to whatever purpose and whichever way, all for free, provided that you retain copyright 
notice at the beginning of the script.

