# DepClean <img src="https://github.com/castor-software/depclean/blob/master/.img/logo.svg" align="left" height="135px" alt="DepClean logo"/>

[![Build Status](https://travis-ci.org/castor-software/depclean.svg?branch=master)](https://travis-ci.org/castor-software/depclean)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=castor-software_depclean&metric=alert_status)](https://sonarcloud.io/dashboard?id=castor-software_depclean)
[![Maintainability Rating](https://sonarcloud.io/api/project_badges/measure?project=castor-software_depclean&metric=sqale_rating)](https://sonarcloud.io/dashboard?id=castor-software_depclean)
[![Reliability Rating](https://sonarcloud.io/api/project_badges/measure?project=castor-software_depclean&metric=reliability_rating)](https://sonarcloud.io/dashboard?id=castor-software_depclean)
[![Security Rating](https://sonarcloud.io/api/project_badges/measure?project=castor-software_depclean&metric=security_rating)](https://sonarcloud.io/dashboard?id=castor-software_depclean)
[![Maven Central](https://img.shields.io/maven-central/v/se.kth.castor/depclean-core.svg)](https://search.maven.org/search?q=g:se.kth.castor%20AND%20a:depclean*)
[![Vulnerabilities](https://sonarcloud.io/api/project_badges/measure?project=castor-software_depclean&metric=vulnerabilities)](https://sonarcloud.io/dashboard?id=castor-software_depclean)
[![Bugs](https://sonarcloud.io/api/project_badges/measure?project=castor-software_depclean&metric=bugs)](https://sonarcloud.io/dashboard?id=castor-software_depclean)
[![Code Smells](https://sonarcloud.io/api/project_badges/measure?project=castor-software_depclean&metric=code_smells)](https://sonarcloud.io/dashboard?id=castor-software_depclean)
[![Lines of Code](https://sonarcloud.io/api/project_badges/measure?project=castor-software_depclean&metric=ncloc)](https://sonarcloud.io/dashboard?id=castor-software_depclean)
[![Duplicated Lines (%)](https://sonarcloud.io/api/project_badges/measure?project=castor-software_depclean&metric=duplicated_lines_density)](https://sonarcloud.io/dashboard?id=castor-software_depclean)
[![Technical Debt](https://sonarcloud.io/api/project_badges/measure?project=castor-software_depclean&metric=sqale_index)](https://sonarcloud.io/dashboard?id=castor-software_depclean)

<!--
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=castor-software_depclean&metric=coverage)](https://sonarcloud.io/dashboard?id=castor-software_depclean)
-->

## What is DepClean?

DepClean is a tool to automatically remove dependencies that are included in your Java dependency tree but are not actually used in the project's code. DepClean detects and removes all the unused dependencies declared in the `pom.xml` file of a project or imported from its parent. For that, it relies on bytecode static analysis and extends the `maven-dependency-analyze` plugin (more details on this [plugin](https://maven.apache.org/plugins/maven-dependency-plugin/analyze-mojo.html)). DepClean does not modify the original source code of the application nor its original `pom.xml`. It can be executed as a Maven goal through the command line or integrated directly into the Maven build lifecycle.

## How does it work?

DepClean runs before executing the `package` phase of the Maven build lifecycle. It statically collects all the types referenced in the project under analysis as well as in its declared dependencies. Then, it compares the types that the project actually use in the bytecode with respect to the class members belonging to its dependencies.

With this usage information, DepClean constructs a new `pom.xml` based on the following steps:

1. add all used transitive dependencies as direct dependencies
2. remove all unused direct dependencies
3. exclude all unused transitive dependencies

If all the tests pass, and the project builds correctly after these changes, then it means that the dependencies identified as bloated can be removed. DepClean produces a file named `pom-debloated.xml`, located in the root of the project, which is a clean version of the original `pom.xml` without bloated dependencies.


## Usage

You can configure the `pom.xml` file of your Maven project to use DepClean as part of the build:

```xml
<plugin>
    <groupId>se.kth.castor</groupId>
    <artifactId>depclean-maven-plugin</artifactId>
    <version>1.1.0</version>
    <executions>
        <execution>
            <goals>
                <goal>depclean</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### Optional Parameters

The Maven plugin can be configured with the following additional parameters.

| Name   |  Type |   Description      | 
|:----------|:-------------:| :-------------| 
| `<ignoreDependencies>` | `Set<String>` | Add a list of dependencies, identified by their coordinates, to be ignored by DepClean during the analysis and considered as used dependencies. Useful to override incomplete result caused by bytecode-level analysis. **Dependency format is:** `groupId:artifactId:version`.|
| `<ignoreScopes>` | `Set<String>` | Add a list of scopes, to be ignored by DepClean during the analysis. Useful to not analyze dependencies with scopes that are not needed at runtime. **Valid scopes are:** `compile`, `provided`, `test`, `runtime`, `system`, `import`. An Empty string indicates no scopes (default).|
| `<createPomDebloated>` | `boolean` | If this is true, DepClean creates a debloated version of the pom without unused dependencies called `debloated-pom.xml`, in the root of the project. **Default value is:** `false`.|
| `<createResultJson>` | `boolean` | If this is true, DepClean creates a JSON file of the dependency tree along with metadata of each dependency. The file is called `depclean-results.json`, and is located in the root of the project. **Default value is:** `false`.|
| `<failIfUnusedDirect>` | `boolean` | If this is true, and DepClean reported any unused direct dependency in the dependency tree, the build fails immediately after running DepClean. **Default value is:** `false`.|
| `<failIfUnusedTransitive>` | `boolean` | If this is true, and DepClean reported any unused transitive dependency in the dependency tree, the build fails immediately after running DepClean. **Default value is:** `false`.|
| `<failIfUnusedInherited>` | `boolean` | If this is true, and DepClean reported any unused inherited dependency in the dependency tree, the build fails immediately after running DepClean. **Default value is:** `false`.|
| `<skipDepClean>` | `boolean` | Skip plugin execution completely. **Default value is:** `false`.|


## Installing and building from source

Prerequisites:

- [Java OpenJDK 8](https://openjdk.java.net) or above
- [Apache Maven](https://maven.apache.org/)

In a terminal clone the repository and switch to the cloned folder:

```bash
git clone https://github.com/castor-software/depclean.git
cd depclean
```
Then run the following Maven command to build the application and install the plugin locally:

```bash
mvn clean install
```
Once the plugin is installed, you can execute the `depclean` plugin goal directly in the command line:

```shell script
# First, compile the project sources and tests
mvn compile   
mvn compiler:testCompile
# Then, executed DepClean
mvn se.kth.castor:depclean-maven-plugin:1.1.0:depclean -Dcreate.pom.debloated=true -Dcreate.result.json=true
```

This is an example of the output (note the dependencies are ordered according to the JAR size):

```
-------------------------------------------------------
 D E P C L E A N   A N A L Y S I S   R E S U L T S
-------------------------------------------------------
USED DIRECT DEPENDENCIES [11]:
	org.apache.flink:flink-shaded-guava:18.0-12.0:compile (2 MB)
	org.apache.flink:flink-shaded-jackson:2.10.1-12.0:test (2 MB)
	org.apache.commons:commons-compress:1.20:compile (617 KB)
	commons-collections:commons-collections:3.2.2:compile (574 KB)
	joda-time:joda-time:2.5:test (574 KB)
	org.apache.commons:commons-lang3:3.3.2:compile (403 KB)
	com.esotericsoftware.kryo:kryo:2.24.0:compile (331 KB)
	org.apache.flink:flink-shaded-asm-7:7.1-12.0:compile (275 KB)
	org.apache.flink:flink-test-utils-junit:1.12-SNAPSHOT:test (54 KB)
	org.apache.flink:flink-metrics-core:1.12-SNAPSHOT:compile (18 KB)
	org.apache.flink:flink-annotations:1.12-SNAPSHOT:compile (16 KB)
USED INHERITED DEPENDENCIES [7]:
	org.mockito:mockito-core:2.21.0:test (550 KB)
	junit:junit:4.12:test (307 KB)
	org.hamcrest:hamcrest-all:1.3:test (299 KB)
	org.powermock:powermock-api-mockito2:2.0.4:test (86 KB)
	org.powermock:powermock-module-junit4:2.0.4:test (46 KB)
	org.slf4j:slf4j-api:1.7.15:compile (39 KB)
	com.google.code.findbugs:jsr305:1.3.9:compile (32 KB)
USED TRANSITIVE DEPENDENCIES [5]:
	org.powermock:powermock-core:2.0.4:test (196 KB)
	org.powermock:powermock-reflect:2.0.4:test (64 KB)
	org.hamcrest:hamcrest-core:1.3:test (43 KB)
	org.objenesis:objenesis:2.1:compile (40 KB)
	com.esotericsoftware.minlog:minlog:1.2:compile (4 KB)
POTENTIALLY UNUSED DIRECT DEPENDENCIES [2]:
	org.projectlombok:lombok:1.16.22:test (1 MB)
	org.joda:joda-convert:1.7:test (100 KB)
POTENTIALLY UNUSED INHERITED DEPENDENCIES [5]:
	org.apache.logging.log4j:log4j-core:2.12.1:test (1 MB)
	org.apache.logging.log4j:log4j-api:2.12.1:test (270 KB)
	org.apache.logging.log4j:log4j-1.2-api:2.12.1:test (65 KB)
	org.apache.logging.log4j:log4j-slf4j-impl:2.12.1:test (22 KB)
	org.apache.flink:force-shading:1.12-SNAPSHOT:compile (7 KB)
POTENTIALLY UNUSED TRANSITIVE DEPENDENCIES [6]:
	net.bytebuddy:byte-buddy:1.8.15:test (2 MB)
	org.javassist:javassist:3.24.0-GA:test (759 KB)
	net.bytebuddy:byte-buddy-agent:1.8.15:test (40 KB)
	org.powermock:powermock-api-support:2.0.4:test (21 KB)
	org.powermock:powermock-module-junit4-common:2.0.4:test (17 KB)
	org.apache.flink:force-shading:1.12-SNAPSHOT:compile (7 KB)
[INFO] Starting debloating POM
[INFO] Adding 5 used transitive dependencies as direct dependencies.
[INFO] Removing 2 unused direct dependencies.
[INFO] Excluding 5 unused transitive dependencies one-by-one.
[INFO] POM debloated successfully
[INFO] pom-debloated.xml file created in: /Users/cesarsv/IdeaProjects/flink-master/flink-core/pom-debloated.xml
[INFO] depclean-results.json file created in: /Users/cesarsv/IdeaProjects/flink-master/flink-core/depclean-results.json
```

## License

Distributed under the MIT License. See [LICENSE](https://github.com/castor-software/depclean/blob/master/LICENSE.md) for more information.

## Funding

DepClean is partially funded by the [Wallenberg Autonomous Systems and Software Program (WASP)](https://wasp-sweden.org).

<img src="https://github.com/castor-software/depclean/blob/master/.img/wasp.svg" height="50px" alt="Wallenberg Autonomous Systems and Software Program (WASP)"/>
