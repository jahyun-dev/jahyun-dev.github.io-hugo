---
title: "빌드툴 Gradle"
date: 2018-08-14T18:52:16+09:00
draft: false
tags: ["programming", "gradle", "build-tools"]
description: "빌드툴 Gradle"
---

본 문서의 대상자는 Maven에 대한 기본 지식이 있는 자바 개발자다. JDK10, Spring Boot 기반 멀티 모듈 프로젝트를 구성하며 Gradle을 빠르게 훑어볼 것이다.

문서의 자료는 대부분 [Gradle User Guide](https://docs.gradle.org/current/userguide/userguide.html)와 [권남님의 위키](http://kwonnam.pe.kr/wiki/root)에서 참고했다. [참고자료](#참고자료)에 첨부한다. 자세한 사항이 알고 싶다면 두 링크를 참고하도록 하자.


# Gradle

Gradle은 기존 Maven의 단점을 다수 개선한 빌드툴이다. Maven의 XML 기반 Pom 파일 대신 Groovy DSL을 사용해 가독성 뿐만 아니라 직접 빌드 스크립트를 짜거나 플러그인을 호출하며, 빌드 과정을 프로그래밍 하듯 할 수 있다. Maven과 더불어 Ant, 스크립트 언어 등의 기능이 합쳐진 것을 Gradle이라고 보면 된다.

Gradle을 [설치](https://gradle.org/install/) 하고 프로젝트 폴더에 들어가 아래 명령을 실행하면 [Build Init Plugin](://docs.gradle.org/4.8/userguide/build_init_plugin.html)을 통해 새로운 그레이들 프로젝트를 생성한다. 실행 시 `--type`을 지정 할 수 있다. 생략 시 `basic` 타입이 생성되며, 여기선 이를 사용하기로 한다.

```bash
mkdir hello-world
cd hello-world
gradle init
```

이를 통해 `hello-world` 폴더 내에 아래와 같은 파일/디렉토리가 생성된다.

```bash
> hello-world
    settings.gradle
    build.gradle
    gradlew
    gradlew.bat
    > gradle
        > wrapper
            gradle-wrapper.jar
            gradle-wrapper.properties
```

`gradle, gradlew.bat, gradlew`는 Gradle 실행을 위한 파일이다. Gradle 실행 시 권장 방법은 Gradle Wrapper를 사용하는 것이다. Wrapper는 프로젝트에서 사용하기로 설정된 Gradle을 다운받는 일종의 스크립트라고 볼 수 있다. 개발자가 별도의 설정없이 `gradlew` 혹은 `gradlew.bat`(windows)로 별도의 인스톨 과정없이 Gradle 프로젝트를 빌드 할 수 있다.

settings.gradle은 프로젝트의 구성 정보를, build.gradle은 빌드 방법을 정의한다.
각각의 파일을 살펴보도록 하자.

# settings.gradle

```groovy
rootProject.name = 'hello-world'
```

`rootProject.name`에 최상위 프로젝트 이름을 지정할 수 있다. 프로젝트 디렉토리 이름과 무관하게 설정 가능하다. 여기선 멀티 프로젝트로 구성할 것이므로 하위 프로젝트 또한 지정해주어야 한다.

```groovy
rootProject.name = 'hello-world'
include 'core', 'api'
```

위 스크립트에서 보듯이 하위 프로젝트를 `include` 하면 된다. 여러 프로젝트에서 공유하는 자바 프로젝트인 `core`와 스프링 부트 기반 `api`로 두 개의 하위 프로젝트를 include 한다.

`./gradlew projects` 명령어를 통해 settings.gradle 에 구성된 프로젝트 목록을 Gradle이 잘 인식함을 확인할 수 있다.

```bash
> Task :projects

------------------------------------------------------------
Root project
------------------------------------------------------------

Root project 'hello-world'
+--- Project ':api'
\--- Project ':core'
```

위에서 실행한 `projects`는 Gradle이 제공하는 `Task`다. Gradle을 통해 기본적으로 실행되는 단위를 `Task`라고 한다. `Task`에 대한 도움말은 `help --task [이름]`으로 확인할 수 있다.

```bash
➜ ./gradlew help --task projects

> Task :help
Detailed task information for projects

Path
     :projects

Type
     ProjectReportTask (org.gradle.api.tasks.diagnostics.ProjectReportTask)

Description
     Displays the sub-projects of root project 'hello-world'.

Group
     help

```

# 각 프로젝트 디렉토리 생성

루트 프로젝트의 `settings.gradle`에 설정한 것과 같이, 프로젝트의 디렉토리를 생성하자. 각 프로젝트 내부에 build.gradle 파일을 만들자.

```bash
mkdir core api
touch core/build.gradle api/build.gradle
```

# build.gradle

`build.gradle`은 빌드 구성 스크립트다. 해당 프로젝트의 빌드와 작업을 정의한다. 지금까지 한 구성으로 루트 프로젝트와 Core, API의 세 빌드 파일이 있다.

```bash
> hello-world
    build.gradle
    > core
        build.gradle
    > api
        build.gradle
```

최상위 `build.gradle`은 모든 프로젝트에서 사용할만한 공통 설정이나 Task 등이 들어가며, 각 프로젝트 내부의 `build.gradle`엔 각 프로젝트의 설정이 들어간다.

우선 각 프로젝트 간의 의존성을 설정해보자. `hello-world` 프로젝트는 `api` 컴포넌트가 `core` 컴포넌트에 의존성이 있다고 가정한다. 의존성은 아래와 같이 추가할 수 있다.

우선 core 프로젝트를 보자. Java 기반 프로젝트이므로 [java plugin](https://docs.gradle.org/current/userguide/java_plugin.html)을 추가했다. `java plugin`은 프로젝트에 테스트나 번들링, 컴파일 등의 기능을 프로젝트에 추가해준다. 

```bash
// core/build.gradle
apply plugin: 'java'
println('build core project')
```

아래는 api 프로젝트다. `dependencies`에 `project(':core')`를 추가한다.

```bash
// api/build.gradle
apply plugin: 'java'

dependencies {
    compile project(':core')
}

println('build api project')
```

프로젝트 간 의존성이 추가되었다. 이제 `build`를 실행해보자.

```bash
./gradlew build

> Configure project :api
build api project

> Configure project :core
build core project

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 up-to-date
```

두 프로젝트가 무사히 빌드되는 것을 확인할 수 있다.

# 공통 설정 분리

여러 프로젝트 간 공통 설정이 필요할 수 있다. 이러한 부분은 루트 프로젝트의 build.gradle에 정의할 수 있다. 

우선 `apply plugin: 'java'` 설정을 분리해보자.

```groovy
// build.gradle
allprojects {
    apply plugin: 'java'
}
```

`apply plugin: 'java'`는 core와 api의 모든 프로젝트에 적용되는 Task이므로 `allprojects`에 선언하면 된다. `allprojects` 속성은 현재와 모든 서브 프로젝트를 리턴하는 프로젝트다. `allprojects`는 서브 프로젝트 뿐만 아니라 상위 프로젝트도 리턴한다. 이 때 서브 프로젝트에 대해서만 적용하려면 `subprojects`와 같이 선언할 수도 있다.

프로젝트 별로 디테일한 설정이 필요한 경우 아래와 같이 각 프로젝트 설정을 따로 나눌 수 있다. 여기선 일반 자바 프로젝트, 부트 프로젝트 두 가지로 나누기로 한다.

```groovy
def javaProjects = [project(':core'), project(':api')]
def bootProjects = [project(':api')]

allprojects {
    println('all projects tasks')
}

configure(javaProjects) {
    apply plugin: 'java'
    println('java projects tasks')
}

configure(bootProjects) {
    println('boot projects tasks')
}
```

위와 같이 특정한 프로젝트를 설정해 각각 Task를 지정할 수 있다. 우선 `java project`에만 `java plugin`을 적용해두었다. 빌드를 하면,

```bash
./gradlew build

> Configure project :
all projects tasks
all projects tasks
all projects tasks
java projects tasks
java projects tasks
boot projects tasks

> Configure project :api
build api project

> Configure project :core
build core project

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 up-to-date
```

와 같이 각 프로젝트 별로 Task가 실행되는 것을 확인할 수 있다.

# 의존성 추가

# 참고자료

- [권남 위키 - Maven에서 Gradle로](http://kwonnam.pe.kr/wiki/gradle/from_maven)
