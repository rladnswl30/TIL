# Spring WebFlux (Reactive)
- Spring 5.0 버전에 추가된 Reactive Stack Web Framework  
- 완전한 Non-Blocking으로 동작

# Overview
WebFlux가 탄생한 이유는   
첫 번째로, 적은 쓰레드로 동시 처리를 제어하고, 적은 하드웨어 리소스를 확장하기 위해 만들어졌다.  
두 번째로, 자바 8부터 함수형 API를 작성할 수 있게 됨으로써 Non-Blocking Application에서 비동기 로직을 작성이 편리해지면서 만들어졌다.


# Reactive
- "Reactive"라는 용어는 변화에 반응하는 것을 중심에 두고 만든 Programming model을 의미  
  (I/O 이벤트에 반응하는 네트워크 컴포넌트, 마우스 이벤트에 반응하는 UI 컨트롤러 등...)  
- Non-Blocking은 작업을 기다리기보다는 완료되거나 데이터를 사용할 수 있게 되면 반응하므로, 말그대로 Reactive
- Non-Blocking back pressure는 중요한 메커니즘이다.  
  동기식에서는 Blocking 호출은 호출자를 강제로 기다리게 하는 일종의 back pressure다.  
  Non-Blocking 코드에선, Producer 속도가 Consumer 속도를 압도하지 않도록 이벤트 속도를 제어한다.
- Reactive Stream은 back pressure를 통한 비동기 컴포넌트간 상호작용을 정의한 간단한 스펙  
  ex) 데이터 Repository(Publisher)가 데이터를 만들고, Http 서버(Subscriber)로 이 데이터로 요청을 처리할 수 있다.  
  Reactive Stream을 쓰는 주 목적은 Subscriber가 Publisher의 데이터 생산 속도를 제어하는 것!

# Reactive API
- Reactor는 Spring WebFlux가 선택한 Reactive library  
  (Mono와 Flux API 타입 제공)
- 서버 사이드 자바에 초점을 두고 Spring과 긴밀하게 협력하여 개발됨
- 순수한 Publisher를 입력으로 받아 내부적으로 Reactor 타입으로 맞추고, 이걸 사용해서 Flux나 Mono를 반환

# Programming Models
### Annotated Controllers
- Spring MVC와 동일하며 spring-web 모듈에 있는 같은 어노테이션 사용
- @RequestBody로 Reactive 인자를 받을 수 있다.
### Functional Endpoints
- 경량화된 Lambda 기반 함수형 Programming Model
- 요청을 라우팅해주는 Library, Util 모음
- Annotation으로 의도를 선언해서 콜백받는 Annotated Controller와 달리 요청을 Application이 처음부터 끝까지 다 제어 

# Applicability
<img src="https://user-images.githubusercontent.com/9473513/119232799-74c19e80-bb61-11eb-8e40-19099cd130df.png" width="70%" />  

- 이미 잘 동작하고 있는 Application이라면 굳이 바꿀 필요 없다.
- Spring MVC Application에서 외부 서비스를 호출한다면 Reactive WebClient 사용을 고려해보자.  
  서비스 호출에 지연이 있거나 여러 서비스가 엮여 있는 API라면 효과가 좋다.
- 팀 규모가 크다면 Non-Blocking, 함수형, 선언적 프로그래밍은 Running Curve가 높다는 점을 고려해야 한다.  
  한 번에 전환하지 않고 Reactive WebClient부터 적용해보는 것도 좋은 방법이다.  
  Non-Blocking IO 동작 방식과 효과부터 학습하는 것이 좋다. 
  
# Performance
WebClient를 사용해서 외부 서비스 호출을 병렬로 처리하면 빨라질 수 있지만, 전반적으로 보면 Non-Blocking 방식이 처리할 일이 더 많다 보니 처리 시간이 더 길어질 수 있다.  
이점이라고 하면 고정된 적은 thread와 적은 memory로도 확장할 수 있다.  
예측할 수 있는 방법으로 확장하기 때문에 부하 속에서도 Application의 복원 능력은 더 좋아진다.  
이 점이 Reactive stack의 강점이고 그 차이는 엄청나다.