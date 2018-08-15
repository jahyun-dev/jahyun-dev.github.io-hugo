---
title: "Gradle 1 : 멀티 프로젝트"
date: 2018-08-14T18:52:16+09:00
draft: false
tags: ["programming", "gradle", "build-tools"]
description: "Gradle 1 : 멀티 프로젝트"
---

본 문서의 대상자는 Maven에 대한 기본 지식이 있는 자바 개발자다. JDK10, Spring Boot 기반 멀티 모듈 프로젝트를 구성하며 Gradle을 빠르게 훑어볼 것이다.

자료 대부분은 [Gradle User Guide](https://docs.gradle.org/current/userguide/userguide.html)와 [권남님의 위키](http://kwonnam.pe.kr/wiki/root)에서 참고했다. 자세한 사항은 두 링크를 보도록 하자.


# Gradle

Gradle은 기존 Maven의 단점을 다수 개선한 빌드툴이다. Maven의 XML 기반 Pom 파일 대신 Groovy DSL을 사용해 가독성 뿐만 아니라 직접 빌드 스크립트를 짜거나 플러그인을 호출하며, 빌드 과정을 프로그래밍 하듯 할 수 있다. Maven과 더불어 Ant, 스크립트 언어 등의 기능이 합쳐진 것을 Gradle이라고 볼 수 있다.

Gradle을 [설치](https://gradle.org/install/) 하고 프로젝트 폴더에 들어가 아래 명령을 실행하면 [Build Init Plugin](://docs.gradle.org/4.8/userguide/build_init_plugin.html)을 통해 새로운 그레이들 프로젝트를 생성한다. 실행 시 `--type`을 지정 할 수 있다. 생략 시 `basic` 타입이 생성되며, 여기선 기본 타입을 사용하자.

```bash
mkdir hello-world
cd hello-world
gradle init
```

`hello-world` 폴더 내에 아래와 같은 파일/디렉토리가 생성된다.

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

Gradle은 Gradle Wrapper를 사용해 실행할것을 권장한다. Wrapper는 프로젝트에서 사용하기로 설정된 Gradle을 다운받는 일종의 스크립트라고 볼 수 있다. `gradle, gradlew.bat, gradlew`는 Gradle 실행을 위한 파일이다. 개발자가 별도의 설정없이 `gradlew` 혹은 `gradlew.bat`(windows)로 별도 인스톨 없이 Gradle 프로젝트를 빌드 할 수 있다.

settings.gradle은 프로젝트의 구성 정보를, build.gradle은 빌드 방법을 정의한다.
각 파일을 살펴보자.

# settings.gradle

```groovy
rootProject.name = 'hello-world'
```

`rootProject.name`에 최상위 프로젝트 이름을 지정할 수 있다. 프로젝트 디렉토리 이름과 무관하게 설정 가능하다. 여기선 멀티 프로젝트로 구성할 것이므로, 하위 프로젝트를 지정해보자.

```groovy
rootProject.name = 'hello-world'
include 'core', 'api'
```

위와 같이이 하위 프로젝트를 `include` 하면 된다. 여러 프로젝트에서 공유하는 자바 프로젝트인 `core`와 스프링 부트 기반 `api`로 두 개의 프로젝트를 추가했다.

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

실행한 `projects`는 Gradle이 제공하는 `Task`다. Gradle을 통해 기본적으로 실행되는 단위를 `Task`라고 한다. `Task`에 대한 도움말은 `help --task [이름]`으로 확인할 수 있다.

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

프로젝트 별로 디테일한 설정이 필요한 경우 아래와 같이 각 프로젝트 설정을 따로 나눌 수 있다. 여기선 자바 프로젝트(`javaProjects`), 부트 프로젝트(`bootProjects`) 두 가지로 나누기로 한다.

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

프로젝트를 설정해 각각 Task를 지정할 수 있다. 우선 `java project`에만 `java plugin`을 적용해두었다. 빌드를 하면,

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

각 프로젝트 별로 Task가 실행됨을 볼 수 있다.

# 빌드 스크립트 의존성 추가

이제 프로젝트에 의존성을 추가해보자. 우선 `buildscript` 설정을 해보자. `buildscript`는 Gradle 빌드 스크립트 자체를 위한 의존성이나 변수, Task, Plugin 등을 지정할 수 있다. 서드파티 플러그인이나 Task, Class 등을 빌드 스크립트 내에서 추가로 사용하려면 해당 의존성을 추가해줘야 한다. `build.gradle` 자체를 실행하기 위한 설정이라 보면 된다.

본 문서는 스프링 부트 기반 프로젝트를 진행하므로, 이를 위한 [spring-boot-gradle-plugin](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/html/) 의존성을 추가할 것이다. Spring Boot 사용 시 executable jar나 war 패키징, 앱 실행, `spring-boot-dependencies`의 dependency management 기능 등을 사용할 수 있게 하는 플러그인이다.

```groovy
buildscript {
    ext {
       springRepo = 'http://repo.spring.io/libs-release'
    }

    repositories {
       maven { url springRepo }
    }

    dependencies {
       classpath "org.springframework.boot:spring-boot-gradle-plugin:2.0.4.RELEASE"
    }
}
```

위에서 `spring-boot-gradle-plugin` 의존성을 추가해줬으므로 필요한 플러그인을 추가해 사용할 수 있다. 

## 자바 프로젝트 공통 설정

이제 자바 프로젝트(`javaProjects = [core, api]`)에 필요한 설정을 추가해보자

```groovy
configure(javaProjects) {
    repositories {
        mavenLocal()
        maven { url springRepo }
    }

    apply plugin: 'java'
    apply plugin: 'io.spring.dependency-management'

    version = '1.0.0.SNAPSHOT'

    targetCompatibility = javaVersion
    sourceCompatibility = javaVersion

    dependencyManagement {
        imports {
            mavenBom org.springframework.boot.gradle.plugin.SpringBootPlugin.BOM_COORDINATES
        }

        dependencies {
            dependency 'com.google.guava:guava:26.0-jre'
        }
    }

    dependencies {
        compile 'org.slf4j:slf4j-api'

        testCompile 'junit:junit'
        testCompile 'org.hamcrest:hamcrest-library'
        testCompile 'org.mockito:mockito-core'
    }

    println('java projects tasks')
}
```

프로젝트에 필요한 repository와 plugin, java version, dependency management, dependency를 지정했다. `io.spring.dependency-management` 플러그인도 추가했는데, 이는 `spring-boot-plugin`이 정의한 [dependency management](https://docs.gradle.org/current/userguide/introduction_dependency_management.html) 설정을 쓰기 위함이다.

`dependencyManagement`란 프로젝트에서 사용할 의존성을 정의하는 것이다.  `dependencies`는 의존성을 추가되는 것이고, `dependencyManagement`는 어떤 artifact의 버전을 사용할지만 기술하고, 실제 의존성이 추가되는건 `dependency`에서 명시적으로 설정했을 때다. 이를 통해 version이나 scope를 지정하고 하위 프로젝트에선 재지정 없이 의존성을 추가할 수 있다. 중앙에서 의존성을 통합하고, 일관성있게 관리할 수 있으므로 편하다. 여러 서브 프로젝트로 구성된 경우, 중앙에서 버전 관리를 해주지 않는다면 각 프로젝트 별로 설정이 달라지고 충돌이 날 여지가 있기 때문에 멀티 프로젝트에선 꼭 필요한 설정이다.

spring plugin에서 제공해주는 `dependency-management` 플러그인은 이를 위한 BOM(Bill of Materials) 파일을 제공한다. BOM은 프로젝트 의존성 버전 제어를 위한 POM이며, version과 repository 등을 정의해준다. 

```groovy
dependencyManagement {
    imports {
        mavenBom org.springframework.boot.gradle.plugin.SpringBootPlugin.BOM_COORDINATES
    }

    dependencies {
        dependency 'com.google.guava:guava:26.0-jre'
    }
}
```

위와 같이 mavenBom을 지정해주면 하위 프로젝트에선,

```groovy
dependencies {
    compile "org.springframework.boot:spring-boot-starter-web"
    compile 'com.google.guava:guava'
}
```

특별한 버전 지정 없이 group:artifactId만 기술하면 의존성이 추가된다. spring plugin이 제공해주는 BOM은 라이브러리 의존성 등이 고려되어 제공되는 버전이므로 plugin이나 BOM 버전만 높이면 해당 호환성에 맞게 version이 자동 지정되므로 무척 편리하다. 

따로 의존성 버전을 지정하려면 `dependencyManagement > dependencies` 블록에 추가하면 된다.

공통으로 dependency에 포함되는 의존성은,

```groovy
dependencies {
    compile 'org.slf4j:slf4j-api'

    testCompile 'junit:junit'
    testCompile 'org.hamcrest:hamcrest-library'
    testCompile 'org.mockito:mockito-core'
}
```

`dependencies`에 추가해주자. 이 경우 `javaProjects`에 해당하는 프로젝트는 모두 기술한 의존성이 추가된다. 여기선 테스트와 로깅을 위한 의존성을 추가해두었다.

## 부트 프로젝트 공통 설정

다음으로 부트 프로젝트(`bootProjects`)를 설정하자.

```groovy
configure(bootProjects) {
    apply plugin: 'org.springframework.boot'
    println('boot projects tasks')
}
```

여기선 Spring Boot 프로젝트를 실행하기 위한 `org.springframework.boot` Plugin만 추가했다. 후에 보겠지만, Spring Boot 어플리케이션의 실행 등을 간편하게 할 수 있는 기능을 갖고 있다.

공통 설정을 끝냈으므로, 이제 각 프로젝트 별 설정을 해보자.

## Core

```groovy
dependencies {
    compile 'org.springframework:spring-context'
    compile 'com.google.guava:guava'
}
```

Core엔 여러 하위 프로젝트에서 사용하는 공통 스프링 빈이나, 유틸 등 공용 클래스를 넣기 위한 프로젝트다. 연습 프로젝트이므로 spring-context와 guava를 추가했다. 필요에 따라 의존성을 추가하도록 하자.

## API

```groovy

dependencies {
    compile project(':core')
    compile "org.springframework.boot:spring-boot-starter-web"
}
```

`api`는 `core` 프로젝트에 의존성이 있으므로, `compile project(':core')`를 통해 해당 의존성을 추가해줬다. 또한 API 구성을 위해 `spring-boot-starter-web` 의존성을 추가한다. 이 때 버전은 `depenencyManagement`에서 미리 정의된 버전이 사용된다.


## 부트 어플리케이션 실행

이제 `api` 프로젝트를 실행해보자. 이를 위해 간단한 실행 어플리케이션 클래스를 만들 것이다.

```bash
mkdir -p api/src/main/java/hello/world
touch api/src/main/java/hello/world/HelloApplication.java
```

디렉토리와 자바 파일을 추가하고 수정해보자.

```java
// api/src/main/java/hello/world/HelloApplication.java
package hello.world;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HelloApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloApplication.class, args);
        System.out.println("Hello, World!");
    }
}
```

Spring Boot Plugin을 활용해 API 어플리케이션을 실행해보자.

```bash
./gradlew :api:bootRun
```

기본 포트는 8080이므로,

```bash
curl localhost:8080/hello
Hello, World!
```

설정한대로 API가 잘 동작함을 확인할 수 있다.

# 의존성 확인

프로젝트 의존성이 어떻게 구성되어있는지, 아래 명령어를 통해 확인해보자. 여러 설정을 할 수 있으나, 우선 `api` 프로젝트의 `compile` 단계에 추가되는 의존성을 확인해보겠다.

```bash
gradle :api:dependencies --configuration compile

//output

...

------------------------------------------------------------
Project :api
------------------------------------------------------------

compile - Dependencies for source set 'main' (deprecated, use 'implementation ' instead).
+--- org.slf4j:slf4j-api -> 1.7.25
+--- project :core
|    +--- org.slf4j:slf4j-api -> 1.7.25
|    +--- org.springframework:spring-context -> 5.0.8.RELEASE
|    |    +--- org.springframework:spring-aop:5.0.8.RELEASE
|    |    |    +--- org.springframework:spring-beans:5.0.8.RELEASE
|    |    |    |    \--- org.springframework:spring-core:5.0.8.RELEASE
|    |    |    |         \--- org.springframework:spring-jcl:5.0.8.RELEASE
|    |    |    \--- org.springframework:spring-core:5.0.8.RELEASE (*)
|    |    +--- org.springframework:spring-beans:5.0.8.RELEASE (*)
|    |    +--- org.springframework:spring-core:5.0.8.RELEASE (*)
|    |    \--- org.springframework:spring-expression:5.0.8.RELEASE
|    |         \--- org.springframework:spring-core:5.0.8.RELEASE (*)
|    \--- com.google.guava:guava -> 26.0-jre
|         +--- com.google.code.findbugs:jsr305:3.0.2
|         +--- org.checkerframework:checker-qual:2.5.2
|         +--- com.google.errorprone:error_prone_annotations:2.1.3
|         +--- com.google.j2objc:j2objc-annotations:1.1
|         \--- org.codehaus.mojo:animal-sniffer-annotations:1.14
\--- org.springframework.boot:spring-boot-starter-web -> 2.0.4.RELEASE

...
```

위와 같이 각 프로젝트의 의존성을 확인할 수 있다.

좀 더 깔끔한 UI로 확인해보려면, gradle의 [build scan](https://scans.gradle.com/)을 사용하면 된다. WEB 기반 UI로 빌드 성능이나, 의존성 등을 손쉽게 확인할 수 있다.

```bash
./gradlew build --scan
```

위 명령어를 치고 gradle이 제공하는 URL을 따라가보면,

![gradle-build-scan](/gradle-build-scan.png)

위와 같이 깔끔한 UI로 프로젝트 빌드 정보를 확인할 수 있다.

# 마치며

연습 프로젝트를 Gradle로 진행중이다. Maven POM 파일보다 가독성도 좋고, 강력한 기능이 많다. 신규 프로젝트라면 학습 비용을 감안하더라도 Maven 대신 Gradle을 도입하는 것이 좋을 것이다. Gradle의 진가는 `Task` 등 정의를 통해 의존성 뿐만 아니라 빌드 스크립트를 유연하고 강력하게 정의할 수 있는 것으로 보인다. 다음 챕터에선 이러한 부분을 살펴보자.

# 참고자료

- [Gradle User Manual](https://docs.gradle.org/current/userguide/userguide.html)
- [권남 위키 - Maven에서 Gradle로](http://kwonnam.pe.kr/wiki/gradle/from_maven)
