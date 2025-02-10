---
layout: post
title:  "Spring 을 위한 Java 개발 셋업"
date:   2025-01-02 12:20:00 +0900
categories: [기타]
tags: [Java]
---

2024년 말, 고민 끝에 복합적인 이유로 인해 나는 스프링을 공부해보기 시작했다.

1. 일단 가장 걱정은 자바랑 친숙하지 않다는 것이었다.

    -> 그래도 대학교 수업때 사용해던 경험이랑 안드로이드 프로그래밍을 열심히 해보았던 경험이 있어서 완전히 백지상태는 아니였다.
    그리고 요즘은 추세가 Java 에서 Kotlin 으로 많이 넘어가고 있어서 결국에는 Kotlin 을 사용 할 것같다는 생각이 들었다.
    물론 Kotlin 도 잘 모르는 언어이지만 Java 에서 오는 문법적인 단점을 잘 보안하고 생태계도 건강한걸로 알고 있어서 개발자
    입장에서는 배우고 사용하기 좋은 언어이고 앞으로 유망할것 같다는 느낌을 받았다.

2. 이미 Python 이나 Nodejs 쪽 경험도 있고 더 익숙하고 이 언어들도 괜찮은 백엔드 프레임웤이 존재한다.

    -> 그럼에도 생전 모르는 스프링을 공부하기로 한 이유는 백엔드쪽으로 방향성을 잡고 싶고 아무래도 국내에서는 
    취업 기회의 폭이 더 넓어질 것 같다는 생각이 들었다.

그리고 일단 스프링은 많은 기업에서 프로덕션에서 사용할 정도로 탄탄하고 백엔드 개발에서 마주하게 되는 어려움과 고충을 잘 녹여낸 프레임워크여서 배울점이 많이 있겠다고 판단이되었다.

## Java 또는 JVM 언어 개발 환경
공부를 위해 여러 스프링이랑 자바에 관한 여러 책이랑 온라인 자료를 찾아보았지만 모두 인텔리제이나 이클립스를 추천하고 사용하는 자료였다.
그리고 visual studio code 를 사용하는건 단 하나도 보지 못 했다.

그래도 뭔가 친숙한 VSCode 를 사용 할 수는 없을까라는 마음이 한구석에 있어서 간단히 VSCode 자바 개발에 관해 직접 찾아보니 생각보다 괜찮은 지원을 하고 있는것 같다는 느낌을 받았다.

VSCode 를 쓰고 싶은 이유는:

1. 익숙하다. 익숙한 개발툴은 생산성을 높이는데 도움을 준다. 이미 VS Code 의 여러 단축기나 기능들을 잘 알고 있기 때문에 새로운 툴을 사용하는것보다 단기적으로 효율적이다.
2. Language-specific IDE 에서 오는 단점들.
    * 특수하다. VS Code 는 단순한 코드 에디터여서 여러 프로그래밍 언어 파일을 작업 할 수 있다. 반면 인텔리제이나 이클립스는 Java 와 JVM language 에 특화되어있다.
    * 무겁다. IDE 보통 여러 빌트인 기능을 가지고 있기 때문에 일반적으로 단순한 코드에디터보다는 무거운 앱이다.

VSCode 와 인텔리제이 둘다 조금씩 사용해본 결과, 일단 결론은 처음 배우는 초심자의 입장에서 인텔리제이 같은 JVM 언어 전용 IDE 를 사용하는게 편리하기도하고
세팅보다 공부에 더 집중할 수 있는 환경을 제공할 것 같다는 결론을 냈다.
위에서 언급한 두 이유를 반박할 수 있을 정도로 편리함과 생산성에 도움이되는 기능들을 인텔리제이는 쉽게 제공한다.

이 글은 Java 를 위한 기본적인 VScode 설정 및 기능과 그 과정속에서 배운 자바 개발에 대한 조그만한 디테일들을 기록했다. 

## Java 로 개발하기 위해 필요한 의존성
1. JDK (Java Development Kit) - Java 코드를 컴파일하고 구동시키는데 필요
2. Maven 또는 Gradle - Java 빌드 툴.

빌드툴은 무조건 필요하지는 않지만 Java 로 어플리케이션 단위의 프로젝트를 진행할때 빌드를 쉽게 도와줄 수 있어서 편리하다. 
보통 스프링 같이 규모가 있는 앱은 빌드툴을 사용한다.

## Java 설치
Java 는 JDK (Java development kit) 의 형태의 프로그램을 다운받아야한다.
그 안에는 Java 프로그램을 컴파일하고 구동할 수 있는 툴이 번들되어 있다. 예를 들어 JVM, javac (컴파일러), JRE 등등.
예를 들어 자바 소스코드는 일단 바이트코드 형태로 `javac` 를 이용해 컴파일되고 이 바이트코드가 JVM (Java virtual machine) 안에서 실행된다.

OpenJDK 의 종류 중 하나를 설치하면 된다. 되게 다양한 종류가 있어서 각각 어떤 이유로 만들어지고 장단점까지 파악하지 못했지만
보통 [Oracle](https://www.oracle.com/java/technologies/downloads/) 또는 [Eclipse Temurin](https://adoptium.net/temurin/releases/) 에서 
다운받아 설치하는게 일반적인것 같다.

MacOS 를 사용하는 경우 [homebrew](https://formulae.brew.sh/formula/openjdk) 로 설치도 가능하다.

#### JAVA_HOME 셋팅
JDK 설치 후 추가적으로 환경변수를 설정해주어야 한다. 사실 아래 Maven 또는 Gradle 을 사용하기 전까지 `JAVA_HOME` 환경변수를
설정하지 않고도 자바 코드를 컴파일하고 실행하는데 문제는 없었다. 하지만 Java 와 연계해서 사용하는 이런툴은 `JAVA_HOME` 환경변수가 설정이 필요하다.

터미널의 프로파일 파일에 아래 변수를 넣어주어야 한다.
보통 프로파일 파일은 유저 홈 디렉토리에서 `~/.profile`, `~/.zprofile`, `~/.bash_profile` 이름이다. 물론 경로는 각자의 시스템에 맞게 변경해주어야한다. 아래는 명시된 경로는 하나의 예시이지만 보통 `/Library/Java/JavaVirtualMachines/` 시작한다.

```sh
export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home"
```

## 빌드 툴
인텔리제이를 사용하는 경우 보통 JDK 만 설치하면 된다. IDE 에 빌드툴들이 내장되어있다.
하지만 IDE 기능을 통해서 사용 가능하기 때문에 CLI 에서 별도로 Maven 이나 Gradle 을 사용해야하는 경우 추가적으로 설치해야한다.
빌드툴은 하나를 선택해서 설치하면 된다. 물론 둘다 사용해보고 싶다면 둘다 설치도 가능하다.

### Maven 설치
Maven 은 `pom.xml` 이라는 정의파일을 사용해 프로젝트의 메타데이터와 의존성등을 명세한다.
Maven 을 이용해 프로젝트의 소스코드 빌드과정을 손 쉽게 할 수 있다. 빌드과정은 보통 의존성을 가져오기,
소스코드 컴파일, 테스트 코드 실행 등으로 이루어지고 이 작업을 커멘드라인 또는 IDE 의 버튼을 통해 쉽게
실행시킬 수 있게 해준다.

Maven 도 여러 방법으로 설치가 가능하다.

#### 공식 문서
가장 무난하게는 [공식 문서](https://maven.apache.org/install.html)를 토대로 진행하면 된다.


#### homebrew
필자는 macOS 를 사용하고 보통은 한곳에서 다운받은 모든 패키지를 관리하는게 쉬워서 `homebrew` 를 통해서 다운받았다.

아래 커멘드를 통해 다운받을 수 있다. 여기서 주의 할 점은 위에서 OpenJDK 를 `homebrew` 가 아니라 별도로 설치했다면
`homebrew` 가 OpenJDK 가 없다고 판단하여 자동으로 `homebrew` 버전의 OpenJDK 를 설치한다. 
이럴경우 두개의 OpenJDK 가 시스템에 설치되었기 때문에 충돌 할 가능성이 있다.

`--ignore-dependencies` 를 붙여줘서 `homebrew` 가 자동으로 OpenJDK 설치하는 것을 방지 할 수 있다.

```sh
brew install --ignore-depdencies maven
```

### Gradle 설치
Gradle 도 Maven 과 같은 맥락으로 자바 프로젝트 빌드과정을 쉽게 만들어준다.

#### 공식 문서
[공식 문서](https://gradle.org/install/)를 참고하여 설치 또는 

#### homebrew

```sh
brew install --ignore-depdencies gradle
```

## VSCode 세팅
인텔리제이에서 자바나 스프링 프로젝트 시작하는 법은 따로 가이드가 없어도 될 정도로
직관적이고 쉽게 UI/UX 를 제공한다고 생각한다.

하지만 VSCode 는 자바로 개발하기 위해 몇가지 익스텐션을 설치하면 UI 에 자바 프로젝트를 시작 할 수 있는
기능들이 추가된다.

#### Extensions
Extensions 탭에서 `Extension Pack for Java` 를 검색해서 설치하면 자바 개발을 위한 여러 extension 을 번들해서
설치해준다.

![VSCode extension pack for java](/assets/images/extension-pack-for-java.png)

#### 빌드 툴
VSCode 사용시 Maven 또는 Gradle 을 사용할 경우 따로 해당 툴을 시스템에 설치하는건 필수이다.

또한 Maven 사용시 VSCode 설정에서 Maven executable 경로를 설정해주어야 한다.

![Maven path setting](/assets/images/maven-path-setting.png)


#### 간단한 기능
설치 후 `cmd-shift + p` 눌러 command palette 를 열어 `create` 키워드를 검색하면
"Java: Create Java Project..." 하고 "Spring Initializr: Create a Gradle Project..." 같은
quick start 기능을 실행해 프로젝트 디렉토리 구조 및 스타터 코드를 생성 할 수 있다.

![Java extension command palette](/assets/images/java-extension-command-palette.png)

프로젝트 생성 이후 파일 explorer 탭을 열어보면 Java 프로젝트와 관련된 드롭다운 메뉴가 추가 된걸 볼 수 있다. 예를 들어 "Java Projects" 와 만약 Maven 프로젝트를 생성했다면 "Maven" 드롭다운이 추가된다.

![Java project menu](/assets/images/explorer-java-menu.png)


마지막으로 메인 메소드가 작성된 파일을 에디터에서 열면 "Run \| Debug" 문구를 보여주고 개발자가 원클릭으로 메인을 실행 할 수 있게 해준다.

![Java main method](/assets/images/java-entrypoint-run-debug.png)

이런식으로 VSCode 도 익스텐션 설치를 통해 나름 괜찮은 프로젝트 개발 환경을 제공해준다.
