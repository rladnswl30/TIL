# Gradle ?
- Gradle은 Groovy를 기반으로 한 빌드 도구
- Ant와 Maven과 같은 이전 세대 빌드 도구의 단점을 보완하고 장점을 취합하여 만든 오픈 소스로 공개된 빌드 도구이다.

# [Gradle GitHub](https://github.com/gradle/gradle)
  
# 장점
1. 의존성 관리를 위한 다양한 방법을 제공한다. 
2. 빌드 스크립트를 XML 언어가 아닌, JVM에서 동작하는 스크립트 언어 ‘그루비’ 기반의 DSL(Domain Specific Language)를 사용한다.
3. 그루비(Groovy)는 자바 문법과 유사하여 자바 개발자가 쉽게 익힐 수 있는 장점이 있으며,  
   Gradle Wrapper를 이용하면 Gradle이 설치되지 않은 시스템에서도 프로젝트를 빌드할 수 있다.
4. 메이븐(Maven)의 pom.xml을 Gradle 용으로 변환할 수 있으며,  
   Maven의 중앙 저장소도 지원하기 때문에 라이브러리를 모두 그대로 가져다 사용할 수 있다.

# 특징
1. High performance  
   Gradle은 실행시켜야 하는 task만 실행시키고 다른 불필요한 동작은 하지 않는다.  
   또, build cache를 사용함으로써 이전 실행의 task output을 재사용할 수 있다.  
   심지어 서로 다른 기계에서도 build cache를 공유하여 성능을 높일 수 있다.
   
2. JVM foundation  
   Gradle은 JVM에서 실행되고, JVM을 사용하려면 JDK를 설치해야 한다.    
   따라서 Java Standard API를 빌드 로직에 사용할 수 있다. 또한 Gradle을 다양한 플랫폼에서 실핼할 수 있다.

3. Conventions  
   Gradle은 Maven으로부터 의존 라이브러리 관리 기능을 차용했다.    
   따라서 컨벤션을 따라 Java 프로젝트와 같은 일반적인 유형의 프로젝트를 쉽게 빌드할 수 있다.  
   그리고 필요하다면 컨벤션을 오버라이딩하거나 task를 추가하면서 컨벤션 기반의 빌드를 커스터마이징 할 수 있다.

4. Extensibility  
   Gradle을 확장하면 고유의 task 타입을 제공하거나 모델을 빌드할 수 있다.

5. IDE, Build Scan support  
   Android Studio, IntelliJ IDEA, Eclipse 등의 IDE에서 Gradle을 임포트하여 사용할 수 있다. 그리고 빌드를 모니터링할 수 있는 Build Scan을 지원한다.
   
# 구조
![gradle 구조](https://user-images.githubusercontent.com/9473513/118345773-b0f96b80-b571-11eb-922f-8071ec6955ef.png)
- gradlew  
리눅스 또는 맥OS용 실행 쉘 스크립트 파일

- gradlew.bat  
윈도우용 실행 배치 스크립트 파일

- gradle-wrapper.jar  
JAR 형식으로 압축된 Wrapper 파일.  
gradlew나 gradlew.bat 파일이 프로젝트 안에 설치되는 이 파일을 사용하여 Gradle task를 실행한다.

- gradle-wrapper.properties  
Gradle Wrapper 설정 정보 파일.   
Wrapper의 버전 등을 설정할 수 있다.

- build.gradle  
프로젝트의 라이브러리 의존성, 플러그인, 라이브러리 저장소 등을 설정할 수 있는 빌드 스크립트 파일이다.

- settings.gradle  
프로젝트의 구성 정보 파일이다. 멀티 프로젝트를 구성하여 프로젝트를 모듈화할 경우, 하위 프로젝트의 구성을 설정할 수 있다.
  
# Dependency Configuration
Gradle 프로젝트에서 선언된 모든 의존성은 사용되는 특정 범위를 가진다.  
예를 들어 어떤 의존성은 컴파일 할 때에만 사용될 수 있고, 다른 의존성은 런타임할 때에 사용될 수 있다.  
이렇게 의존성의 범위를 표현한 것을 dependency configuration이라고 한다.

- Implementation: 구현할 때에만 사용된다.
- compileOnly: 컴파일할 때에만 사용되고 런타임 때에는 사용되지 않는다.
- runtimeOnly: 런타임 때에만 사용된다.
- testImplementation: 테스트할 때에만 사용된다.

# 라이브러리 의존성 관리
![gradle 라이브러리 의존성 관리](https://user-images.githubusercontent.com/9473513/118345821-0df52180-b572-11eb-9fb7-f9274d5090cc.png)  
의존성은 종종 모듈로 제공되는데, 이 모듈들을 저장하고 있는 곳을 repository라고 한다.  
repository는 로컬 저장소가 될 수도 있고 원격 저장소가 될 수도 있다.  
Gradle에게 어디서 의존성 모듈을 가져올건지 알려줘야 하는데, repository 선언을 통해 할 수 있다.  
Gradle은 특정 task를 실행시키기 위해 필요한 의존성들을 런타임시에 원격 저장소에서 다운로드받거나 로컬 저장소에서 가져온다.  
멀티 프로젝트를 구성했을 경우에는 다른 프로젝트를 가져온다. 이 과정을 dependency resolution이라고 한다.

Gradle은 향후에 불필요한 네트워크 호출을 하지 않기 위해 의존성 파일들을 dependency cache라고 하는 로컬 캐시에 저장한다.


### reference  
https://docs.gradle.org/current/userguide/what_is_gradle.html#what_is_gradle  
https://medium.com/@goinhacker/%EC%9A%B4%EC%98%81-%EC%9E%90%EB%8F%99%ED%99%94-1-%EB%B9%8C%EB%93%9C-%EC%9E%90%EB%8F%99%ED%99%94-by-gradle-7630c0993d09  
https://limdevbasic.tistory.com/12  
