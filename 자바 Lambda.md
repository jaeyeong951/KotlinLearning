# 자바 Lambda

자바는 8버전부터 함수형 프로그래밍을 지원하기 위해 함수를 1급객체로 만들고자 했다.

함수를 인자로 넘겨주기위해 `함수형 인터페이스`라는 개념을 제안한다.

### 함수형 인터페이스

추상 메소드를 단 하나 가지는 interface를 말한다.

interface에 추상 메소드가 단 하나라면, 그 단 하나의 추상 메소드를 구현하는 것 자체가 인터페이스의 구현을 의미한다.

```java
//함수형 인터페이스
@FunctionalInterface
public interface Checker {

    boolean check(Student student);
}
/*
함수형 인터페이스에 @FunctionalInterface 애너테이션을 사용할 수 있습니다. 
굳이 애너테이션이 아니더라도 추상 메서드가 하나인 인터페이스는 함수형 인터페이스로 처리 됩니다. 
다만 애너테이션을 사용할 경우 코드상에 명시되어 확인이 쉽고, 
컴파일러가 단일 추상 메서드만을 가지고 있는지 검사하기 때문에 혹시 모를 구현상의 오류를 방지해 주기도 합니다.
또한 javadoc 페이지에 해당 인터페이스가 함수형 인터페이스임을 알리는 문장을 포함시키는 역할을 하기 때문에 가급적 사용하는 것이 좋습니다.
*/

public class ScoreCheck implements Checker {

    @Override
    public boolean check(Student student) {
        return student.score > 80;
    }
}

//Chcker를 구현한 객체를 인자로 넘겨주는 방법
Checker c = new ScoreCheck();
List<Student> results = test.getPreferStudent(students, c);

//익명 클래스를 이용한 방법
List<Student> results = test.getPreferStudent(students, new Checker() {
     @Override
     public boolean check(Student student) {
         return student.score > 80;
     }
});

//람다를 이용한 방법
List<Student> results = test.getPreferStudent(students, student -> student.score > 80);

```

안드로이드의 SetOnClickListener또한 위와 같은데, SetOnClickListener에 인자로 넘겨주는 onClickListener가 위처럼 함수형 인터페이스이다. onClickListener안에는 onClick 메소드 단 하나 뿐이다.

람다를 쓸때마다 위와같이 함수형 인터페이스를 선언하는 과정은 번거로울 수 있으므로 자바에서는 함수형 API를 제공한다.

- Supplier<T> : 인자(매개변수) 없이 반환값만을 갖는 인터페이스
- Consumer<T> : 인자는 있고 반환값이 없는 인터페이스
- Function<T, R> : 인자도 있고 반환값도 존재.
- Predicate<T> : 인자가 있고 반환값은 Boolean, 매개값을 조사하고 T/F를 리턴

람다의 표현법

```java
( parameters ) -> expression body       // 인자가 여러개 이고 하나의 문장으로 구성
( parameters ) -> { expression body }   // 인자가 여러개 이고 여러 문장으로 구성
() -> { expression body }               // 인자가 없고 여러 문장으로 구성
() -> expression body                   // 인자가 없고 하나의 문장으로 구성
```