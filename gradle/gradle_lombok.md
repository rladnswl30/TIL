# Gradle lombok 주입
Junit에서도 lombok을 사용하고자 하는데 보통 아래와 같이 가이드를 하고 있다.
```gradle
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
```

junit에서도 사용하기 위해서는 testCompileOnly, testAnnotationProcessor를 추가해서 사용 가능하며,  
implementation만 추가해서 사용 가능하다.

근데 gradle 5 버전 이상에서는 생성자에 필요한 인자를 주입받지 못하므로 (컴파일 에러 발생)  
testCompileOnly, testAnnotationProcessor를 각각 추가해야 한다고 함.

```gradle
annotationProcessor 'org.projectlombok:lombok'
implementation 'org.projectlombok:lombok'
testAnnotationProcessor 'org.projectlombok:lombok'
testImplementation 'org.projectlombok:lombok'
```

### Reference
https://kkambi.tistory.com/155  
https://eblo.tistory.com/70