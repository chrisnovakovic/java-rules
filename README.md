# Java rules 

This repo provides Java rules for the [please](https://please.build) build system.

# Basic usage
See `./example` in this repo for a working example. 

```python
# BUILD
# This adds the plugin to your project
github_repo(
    name = "java_rules",
    repo = "please-build/java-rules",
    revision = "<Some git tag, commit, or other reference>",
)

# src/main/java/build/please/foo/BUILD
# This defines a java library that can be depended on by other Java rules
java_library(
    name = "foo",
    srcs = ["Foo.java"],
    visibility = ["//src/test/java/build/please/..."]
)

# src/test/java/build/please/foo/BUILD
# A test for the above library. 
java_test(
    name = "foo_test",
    srcs = ["FooTest.java"],
    deps = [
        "//third_party/java:junit", 
        "//src/main/java/build/please/foo",
    ]
)

# src/main/java/build/please/foo/BUILD
# This produces a self-executing .jar 
java_binary(
    name = "app",
    main_class = "build.please.Main",
    srcs = ["Main.java"],
    deps = ["//src/main/java/build/please/foo"]
)

# third_party/java/BUILD
# These rules pull down third party dependencies from maven. 
maven_jar(
    name = "junit",
    hash = "59721f0805e223d84b90677887d9ff567dc534d7c502ca903c0c2b17f05c116a",
    id = "junit:junit:4.12",
    deps = [
        ":hamcrest",
    ],
)

maven_jar(
    name = "hamcrest",
    hash = "4877670629ab96f34f5f90ab283125fcd9acb7e683e66319a68be6eb2cca60de",
    id = "org.hamcrest:hamcrest-all:1.3",
)
```

Then add this to your `.plzconfig`:
```
[Parse]
PreloadSubincludes = @java_rules//build_defs:java
```
Alternatively, if you're not using Java everywhere, you may add `subinclude("@java_rules//build_defs:java")` to each 
BUILD individually. 

## Toolchain
By default, Please will use the JDK on the path. Optionally, you may use `java_toolchain()` to manage your JDK:
```python
# third_party/java/BUILD
java_toolchain(
    name = "toolchain",
    jdk_url = "https://corretto.aws/downloads/resources/11.0.8.10.1/amazon-corretto-11.0.8.10.1-linux-x64.tar.gz",
)
```
This can then be configured to be used by adding the following config:

```
[Plugin "Java"]
Toolchain = //third_party/java:toolchain
```

# Configuration

Plugins are configured through the Plugin section like so:
```
[Plugin "java"]
SomeConfig = some value
```

The available configuration options for this plugin are documented here. 

## JavacTool 
The path to the Java compiler to use. Defaults to `javac`.

```
[Plugin "java"]
JavacTool = /opt/java/bin/javac
```

## JavacFlags
Any additional flags to apply to the javac tool. 

```
[Plugin "java"]
JavacFalgs = --flag1 --flag2
```

## JavacTestFlags
Any additional flags to apply to the javac tool when compiling tests. 

```
[Plugin "java"]
TestJavacFlags = --flag1 --flag2
```

## JunitRunner
The tool used to run Java tests. Defaults to the junit runner for this plugin version 
i.e. `@java_rules//tools:junit_runner`

```
[Plugin "java"]
JunitRunner = /opts/please/bin/junit_runner
```

or

```
[Plugin "java"]
JunitRunner = //tools:my_custom_junit_runner
```

## SourceLevel
The source level of the project. Defaults to 8. 
```
[Plugin "java"]
SourceLevel = 15
```

## TargetLevel
The target level of the project. Defaults to 8.
```
[Plugin "java"]
TargetLevel = 15
```

## ReleaseLevel
The release level of the project. Defaults to 8.
```
[Plugin "java"]
ReleaseLevel = 15
```

## JavacWorker
If set, Please will use this javac worker rather than javac directly. This can speed up compile times as the javac 
process can be reused between build actions.

To start using the javac worker, add this config to your repo, where `java_rules` is the name of the plugin's 
`github_repo()` rule:

```
[Plugin "java"]
JavacWorker = @java_rules//tools:javac_worker
```

## Toolchain
If set, please will use the jdk provided by the `java_toolchain()` rule specified by this build label. See the toolchain 
section above for more info.

```
[Plugin "java"]
Toolchain = //third_party/java:toolchain
```

## DefaultTestPackage
The test package to use when none is specifically set on the `java_test()` rule. If not set, Please will run all tests
present in the `.jar` that are not from third party code. 

```
[Plugin "java"]
DefaultTestPackage = please.build.
```

## MavenRepo
The maven repositories to load jars from. This value can be repeated and defaults to `https://repo1.maven.org/maven2` 
and `https://repo.maven.apache.org/maven2`, the canonical Maven Central repository URLs. 

To add a custom repo, you may do so by setting the following:

```
[Plugin "java"]
MavenRepo = https://repo1.maven.org/maven2
MavenRepo = https://repo.maven.apache.org/maven2
MavenRepo = https://maven.repo.org
```
