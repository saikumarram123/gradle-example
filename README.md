Based on [creating-multi-project-builds](https://github.com/gradle-guides/creating-multi-project-builds)

## Gradle 101
* Everything in Gradle sits on top of two basic concepts: projects and tasks.
* Each project is made up of one or more tasks
* You run a Gradle build using the `gradle` command
* The gradle command looks for a file called `build.gradle` in the current directory. We call this `build.gradle` file a _build script_
* The _build script_ defines a **project** and its **tasks**.
##Setting up Project
1. The first step is to create a folder for the new project and add a Gradle Wrapper to the project

```
$ mkdir creating-multi-project-builds
$ cd creating-multi-project-builds
$ gradle init  
```

2. Open the **settings.gradle**
```groovy
rootProject.name = 'creating-multi-project-builds'
```


3. In a multi-project you can use the top-level build script (also known as the root project) to configure as much commonality as possible, leaving sub-projects to customize only what is necessary for that subproject
The _allprojects_ block is used to a**dd configuration items that will apply to all sub-projects as well as the root project**. In a similar fashion, **the subprojects block can be used to add configurations items for all sub-projects only**. You can use these two blocks as many times as you want in the root project.
Now set the version for each of the modules which you will be adding, via the subproject block in the top-level build script as follows

```groovy
allprojects {
    repositories {
        jcenter() 
    }
}

subprojects {
    version = '1.0'
}
```


#### Configuration for Specific Projects (not all subprojects)
If you need to specify specific configuration for individual subprojects then use following 

```groovy
configure(subprojects.findAll { it.name == 'greeter' || it.name == 'greeting-library' }) { 

// configuration goes here
    apply plugin: 'groovy'

    dependencies {
        testCompile 'org.spockframework:spock-core:1.0-groovy-2.4', {
            exclude module: 'groovy-all'
        }
    }
}
```


#### Using methods

```groovy
configure(appModules()) {
//configuration goes here
}
def modules() {
    subprojects.findAll {
        it.name == '<projects name A>' || it.name == '<projects name B>' 
    }
}
```


#### buildscript

_buildscript_ closure ensures that the dependencies are available for use within the gradle build itself, e.g. buildscript usualy contains list of **gradle plugins as dependencies**

* The global level dependencies and repositories sections list dependencies that required for building your source and running your source etc.
* The _buildscript_ is for the build.gradle file itself. So, this would contain dependencies for say creating RPMs, Dockerfile, and any other dependencies for running the tasks in all the dependent build.gradle


#### Acccessing artifacts(the code itslef) from another subproject
 Creating a collection of sub-projects does not automatically make their respective artifacts automatically available to other sub-projects - that would lead to very brittle projects. Gradle has a specific syntax to link the artifacts of one subproject to the dependencies of another sub-project.
 
 **_greeter/build.gradle_**
 ```groovy
dependencies {
    compile project(':greeting-library') 
}
```

####Build and Task Dependencies
* Task dependencies
```groovy

task hello {
    doLast {
        println 'Hello world!'
    }
}
task intro(dependsOn: hello) {
    doLast {
        println "I'm Gradle"
    }
}
```

* Build dependencies

```groovy
//Adds asciidoctor task into the build lifecycle so that if build is executed for the top-level project, then documentation will be built as well.
build.dependsOn 'asciidoctor'
```

#### Gradle Properties


##### Extra Properties
`ext` is shorthand for project.ext, and is used to define extra properties for the project object. (It's also possible to define extra properties for many other objects.) When reading an extra property, the ext. is omitted (e.g. println project.springVersion or println springVersion). The same works from within methods. It does not make sense to declare a method named ext