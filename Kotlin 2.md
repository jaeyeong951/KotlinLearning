# Kotlin 2.

# 코틀린 기초

## 문(statement)과 식(expression)

코틀린에서 if는 식이다. 문이 아니다. 식은 값을 만들어 내며 다른 식의 하위 요소로 계산에 참여할 수 있다. 반면 문은 그렇지 않다. 자바와 달리 코틀린은 반복문을 제외한 대부분의 제어 구조가 식이다. 이 특징으로 인해 자바에서는 표현 불가능한 다양한 표현이 가능하다. 하지만 대입문은 자바에서는 식이었으나 코틀린에서는 문이 됐다.

```kotlin
// 블록이 본문인 함수
fun max(a: Int, b: Int): Int { 
	return if(a > b) a else b
} 

// 식이 본문인 함수
fun max(a: Int, b: Int) = if (a > b) a else b
```

## 변수

### 변경 가능한 변수와 변경 불가능한 변수

변수 선언 시 사용하는 키워드는 다음과 같이 2가지가 있다.

- val(값을 뜻하는 value) : 변경 불가능한(immutable) 참조를 저장하는 변수 → 자바의 final과 비슷
- var(변수를 뜻하는 variable) : 변경 가능한(mutable) 참조를 저장하는 변수 → 자바의 일반 변수

코틀린 저자는 모든 변수를 val 키워드를 사용해 불변 변수로 선언하고, 나중에 꼭 필요한 경우에만 var로 변경하라고 권장.

변경 불가능한 변수와 변경 불가능한 객체를 부수 효과가 없는 순수 함수와 함께 조합해 사용하면 코드가 함수형에 가까워진다.

val 참조 자체는 불변일지라도 그 참조가 가리키는 객체의 내부 값은 변경될 수 있다.

```kotlin
val languages = arrayListOf("Java")

languages.add("Kotlin")
```

### 문자열 템플릿

```kotlin
fun main(args: Array<String>) {
	val name = if (args.size > 0) args[0] else "Kotlin"
	println("Hello, $name!")
}
```

코틀린의 문자열 템플릿은 자바의 문자열 접합 연산(`"Hello, " + name + "!"`)과 동일한 역할을 하지만 좀 더 간결하며, 자바의 접합 연산처럼 효율적이다.

문자열 템플릿에는 변수 뿐만 아니라 복잡한 식도 중괄호({})로 둘러싸서 문자열 템플릿에 넣을 수 있다.

```kotlin
fun main(args: Array<String>) {
	if(args.size > 0) {
		println("Hello, ${args[0]}!")
	}
}
```

하지만 문자열 템플릿 안에서 변수 이름만 사용하는 경우라도 "Hello, ${name}!"처럼 변수 이름을 {}로 감싸는 습관을 들이면 더 좋다. 문자열 템플릿 안에서 변수가 쓰인 부분을 더 쉽게 식별할 수 있어 가독성이 좋다.

## 클래스와 프로퍼티

```java
//간단한 JavaBean 클래스
public class Person {
	private final String name;

	public Person(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}
}
```

필드가 둘 이상으로 늘어나면 생성자의 본문에서 파라미터를 이름이 같은 필드에 대입하는 대입문의 수도 늘어난다. 코틀린에선 그런 반복적인 필드 대입 코드를 필요로 하지 않는다.

```kotlin
//코틀린으로 변환한 Person 클래스
class Person(val name: String)
```

코틀린의 기본 가시성은 public이므로 이런 경우 생략 가능하다.

코틀린은 getter, setter, 생성자 파라미터를 필드에 대입하기 위한 로직 등 자바에 존재하는 여러 가지 번거로운 준비 코트를 묵시적으로 제공한다.

### 프로퍼티

자바에서는 클래스의 데이터를 필드에 저장하며, 멤버 필드는 보통 private으로 선언한다. 그리고 접근자 메소드를 사용하여 필드를 읽기위한 getter를 제공하고 필드의 변경을 허용해야 하는 경우 setter를 추가한다. 

자바는 필드와 접근자를 묶어 프로퍼티(Property)라고 부르며, 코틀틴은 이를 언어 기본 기능으로 제공한다. 코틀린 프로퍼티는 자바의 필드와 접근자 메소드를 대신한다.

```kotlin
class Person(
	val name: String, //읽기 전용 프로퍼티, getter가 자동으로 생성된다.
	var isMarried: Boolean //읽기와 수정이 가능한 프로퍼티, getter와 setter가 자동으로 생성된다.
)
```

```kotlin
/* Java */
Person person = new Person("Bob", true);
System.out.println(person.getName());
System.out.println(person.isMarried());

/* Kotlin */
val person = Person("Bob", true) // new 키워드를 사용하지 않고 생성자를 호출
println(person.name) // 프로퍼티 이름을 직접 사용해도
printlb(person.isMarried) // 코틀린이 자동으로 게터를 호출해준다.
```

### 커스텀 접근자

프로퍼티의 접근자를 직접 작성할 수 있다.

직사각형 클래스인 Rectangle을 정의하고 자신이 정사각형인지 알려주는 기능을 만들어본다. 정사각형인지를 별도의 필드에 저장할 필요가 없다. 사각형의 너비와 높이가 같은지 검사하면 정사각형 여부를 그때그때 알 수 있다.

```kotlin
class Rectangle(val height: Int, val width: Int) {
	val isSquare: Boolean
		get() {
			return height == width
		}
}
```

isSqaure 프로퍼티에는 자체 값을 저장하는 필드가 필요 없다. 이 프로퍼티에는 게터만 존재하고 클라이언트가 이 프로퍼티에 접근할 때마다 게터가 프로퍼티의 값을 매번 계산해서 리턴한다.

## enum과 when

### enum 클래스 정의

```kotlin
enum class Color {
	RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
}
```

코틀린에서 enum은 소프트 키워드라 부르는 존재다. enum은 class 앞에 있을 때만 특별한 의미를 가지고 다른 곳에서는 이름에도 사용할 수 있다. 반면 class는 키워드이고 이름에 사용할 수 없다.

enum 클래스는 단순히 값만 열거하는 존재가 아니며 enum 클래스 안에도 프로퍼티나 메소드를 정의할 수 있다.

```kotlin
/* 프로퍼티와 메소드가 있는 enum 클래스 */
enum class Color(val r: Int, val g: Int, val b: Int) { // 프로퍼티 정의
	RED(255, 0, 0),
	ORANGE(255, 165, 0),
	YELLOW(255, 255, 0),
	GREEN(0, 255, 0), // 각 상수를 생성할 때 그에 대한 프로퍼티 값을 지정
	BLUE(0, 0, 255),
	INDIGO(75, 0, 130),
	VIOLET(238, 130, 238); // 상수 정의 끝에 반드시 세미콜론을 사용
	
	fun rgb() = (r * 256 + g) * 256 + b // enum 클래스 안에서 메소드를 정의
}

println(Color.BLUE.rgb()) // 255
```

### when으로 enum 클래스 다루기

무지개의 각 색에 대해 그와 상응하는 연상 단어를 짝지어주는 함수가 필요하다고 하자. 자바라면 switch문을 사용할 것이고 이에 해당하는 코틀린 구성 요소는 when이다.

if와 마찬가지로 when도 값을 만들어내는 '식'이다.

```kotlin
fun getMnemonic(color : Color) =
	when(color) {
		Color.RED -> "Richard"
		Color.ORANGE -> "Of"
		Color.YELLOW -> "York"
		Color.GREEN -> "Gave"
		Color.BLUE -> "Battle"
		Color.INDIGO -> "In"
		Color.VIOLET -> "Vain"
}

println(getMnemonic(Color.BLUE)) // Battle
```

```kotlin
/* enum 상수 값을 임포트해서 수식자 없이 사용하기 */
import ch02.cikirs.Color.*

fun getWarmth(color: Color) = when {
	RED, ORANGE, YELLOW -> "warm"
	GREEN -> "neutral" // 임포트한 enum 상수를 이름만으로 사용 가능
	BLUE, INDIGO, VIOLET-> "cold"
}
```

### when과 임의의 객체를 함께 사용

분기 조건에 상수만을 사용할 수 있는 자바 switch와 달리 코틀린 when의 분기 조건은 임의의 객체를 허용

```kotlin
fun mix(c1: Color, c2: Color) = 
	when(setOf(c1, c2)) {
		setof(RED, YELLOW) -> ORANGE
		setof(YELLOW, BLUE) -> GREEN
		setof(BLUE, VIOLET) -> INDIGO
		else -> throw Exception("Dirty color")
	}

```

setOf는 코틀린 표준 라이브러리로 인자로 전달받은 여러 객체를 그 객체를 포함하는 집합인 Set으로 만들어준다. Set은 원소가 모여 있는 컬렉션으로, 각 원소의 순서는 무관하다.

### 인자 없는 when 사용

위의 mix함수 같은 경우 함수 인자로 주어진 두 색과 분기 조건의 두 색을 비교하기 위해 여러 Set 인스턴스를 생성한다. 불필요한 가비지 객체가 늘어나는 것을 방지하기 위해 함수를 고쳐보자.

코드의 가독성은 약간 떨어질 수 있으나 성능 향상을 위해 어느정도 비용을 감수해야 하는 경우도 있다.

```kotlin
fun mixOptimized(c1: Color, c2: Color) = 
	when { // when에 아무 인자도 없다.
		(c1 == RED && c2 = YELLOW) || (c1 == YELLOW && c2 == RED) -> ORANGE
		(c1 == YELLOW && c2 = BLUE) || (c1 == BLUE && c2 == YELLOW) -> GREEN
		(c1 == BLUE && c2 = VIOLET) || (c1 == VIOLET && c2 == BLUE) -> INDIGO
		else -> throw Exception("Dirty color")
	}

```

when에 아무 인자도 없으려면 각 분기의 조건이 Boolean 결과를 리턴하는 식이어야 한다.

### 스마트 캐스트

코틀린에서는 is를 사용해 변수 타입을 검사한다.

is는 자바의 instanceof와 비슷한데, 자바는 instanceof로 타입을 확인한 다음 그 타입에 속한 멤버에 접근하기 위해서는 명시적으로 변수의 타입 캐스팅이 필요하다. 

하지만 코틀린은 컴파일러가 이 작업을 대신 해준다. 어떤 변수가 어떤 타입인지 is로 검사하고 나면 굳이 변수를 해당 타입으로 캐스팅하지 않아도 되는 것이다. 이를 스마트 캐스트(smart cast)라 부른다.

```kotlin
interface Expr
//Num은 그 값을 반환, Sum은 left와 right의 합을 반환
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

/* 자바 스타일로 작성 */
fun eval(e: Expr) : Int {
	if(e is Num) {
		val n = e as Num // -> 코틀린에서는 스마트 캐스트로 인해 불필요한 구문(중복)
		return n.value
	}
	if(e is Sum) {
		return eval(e.right) + eval(e.left) // 변수 e에 대해 스마트 캐스트 적용
	}
	throw IllegalArgumentException("Unknown expression")
}

/* when을 활용하여 코틀린 식으로 리팩토링 */
fun evalWithLogging(e: Expr): Int = 
	when(e) {
		is Num -> {
			println("num: ${e.value}")
			e.value // 이 식이 블록의 마지막 식이므로 e.value가 반환
		}
		is Sum -> {
			val left = evalWithLogging(e.left)
			val right = evalWithLogging(e.right)
			println("sum: ${left} + ${right}")
			left + right
		}
		else -> throw IllegalArgumentException("Unknown expression")
	}

```

블록의 마지막 식이 블록의 결과라는 규칙은 블록이 값을 만들어내야 하는 경우 항상 성립한다.

하지만 이 규칙은 함수에 대해서는 성립하지 않는데, 블록이 본문인 함수는 내부에 return 문이 반드시 있어야 한다.

## 이터레이션(Iteration) : while 과 for 루프

### while 루프

코틀린도 자바처럼 while과 do-while 루프가 있고 문법이 자바와 다르지 않다.

```kotlin
while(조건) {
	/* ... */
}

do {
	/* ... */
} while(조건) // 맨 처음에 무조건 본문을 한 번 실행 후, 조건이 참인 동안 본문을 반복 실행
```

### range(범위)

코틀린에는 자바의 for 루프(한 변수를 초기화 하고 그 변수를 루프를 한 번 실행할 때마다 갱신하고 루프 조건이 거짓이 될 떄 반복을 마치는 형태의 루프)와 동일한 요소가 없다.

이와 같이 초깃값, 증가값, 최종값을 사용한 루프를 위해 코틀린에서는 range를 사용한다.

```kotlin
val oneToTen = 1..10
```

코틀린의 range는 폐구간(닫힌 구간 = 범위의 양 끝값을 포함하는 구간)이다.

어떤 범위에 속한 값을 일정한 순서로 이터레이션하는 경우를 수열이라고 부른다.