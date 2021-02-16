# Kotlin 3. 함수 정의와 호출

## 코틀린에서 컬렉션 만들기

```kotlin
val set = hashSetOf(1, 7, 53)

val list = arrayListOf(1, 7, 53)

val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
```

여기서 hashMapOf의 to는 특별한 키워드가 아니라 일반 함수라는 점에 유의하자.

```kotlin
println(set.javaClass) // class java.util.HashSet

println(list.javaClass) // class java.util.ArrayList

println(map.javaClass) // class java.util.HashMap
```

위 코드에서 알 수 있는 점은 코틀린이 자신만의 고유한 컬렉션을 제공하지 않고 자바의 것을 그대로 가져다 쓴다는 점이다. 코틀린이 자체 컬렉션을 제공하지 않는 이유는 자바와의 상호작용 떄문이다. 자바에서 코틀린을 호출하거나 그 반대의 상황이라도 서로 변환하는 과정이 필요 없다.

코틀린 컬렉션은 자바 컬렉션과 똑같은 클래스이다. 하지만 코틀린에서는 자바보다 더 많은 기능을 쓸 수 있다. 리스트의 마지막 원소를 가져오거나 수로 이루어진 컬렉션에서 최대값을 찾는 기능은 코틀린에서는 기본 제공된다.

## 함수를 호출하기 쉽게 만들기

다음과 같은 함수가 있다고 가정하자.

컬렉션의 원소를 원하는 문자로 구분하고 리턴하는 함수이다.

```kotlin
fun <T> joinToString(
			collection: Collection<T>,
			separator: String,
			prefix: String,
			postfix: String
) : String {
	
	val result = StringBuilder(prefix)
	for((index, element) in collection.withIndex()) {
		if (index > 0) result.append(separator)
		result.append(element)
	}
	result.append(postfix)
	return result.toString()
}
```

```kotlin
>>> val list = listOf(1, 2, 3)
>>> println(joinToString(list, "; ", "(", ")"))
(1; 2; 3)
```

이 함수는 제네릭(generic)하다. 즉, 이 함수는 어떤 타입의 값을 원소로 하는 컬렉션이든 상관없이 처리할 수 있다.

하지만 선언 부분이 마음에 들지 않는다. 함수를 호출 할 때 마다 매번 4개의 인자를 일일이 전달해야 하고, 각 인자가 어떤 역할을 하는지 함수 내부를 보지않는 이상 와닿지 않는다

### 이름 붙인 인자

먼저 함수 호출 부분의 가독성을 개선시켜보자.

코틀린은 다음처럼 함수를 호출하는 것이 가능하다.

```kotlin
joinToString(collection, separator = "; ", prefix = "(", postfix = ")")
```

코틀린으로 작성한 함수를 호출할 떄는 함수에 전달하는 인자 중 일부의 이름을 명시할 수 있다.

### 디폴트 파라미터

코틀린에서는 함수를 선언할 때 파라미터의 기본 값을 지정할 수 있다.

```kotlin
fun <T> joinToString(
			collection: Collection<T>,
			// 디폴트 값이 지정된 파라미터들
			separator: String = ", ",
			prefix: String = "(",
			postfix: String = ")"
) : String {
	
	val result = StringBuilder(prefix)
	for((index, element) in collection.withIndex()) {
		if (index > 0) result.append(separator)
		result.append(element)
	}
	result.append(postfix)
	return result.toString()
}
```

위와 같이 디폴트 값을 선언하면 이 함수를 호출할 때 인자 중 일부를 생략할 수 있다.

```kotlin
>>> joinToString(list)
(1; 2; 3)
```

### 정적 유틸리티 클래스 없애기

객체지향 언어인 자바에서는 모든 코드가 클래스의 메소드로 작성되어야 한다. 하지만 어떤 한 클래스에만 포함시키기 어려운 코드가 많이 생기고 그 결과 정적 메소드를 모아두고 자신의 특별한 상태나 인스턴스 메소드는 없는 정적 유틸리티 클래스가 생겨난다. 그 예로는 자바의 Collections 클래스나 Math 클래스가 전형적인 예시다. 

하지만 코틀린에서는 이런 무의미한 클래스가 필요없다. 대신 함수를 소스 파일의 최상위, 즉 모든 다른 클래스의 밖에 위치시킬 수 있다. 이런 함수들은 여전히 그 파일의 맨 앞에 정의된 패키지의 멤버 함수이므로 다른 패키지에서 해당 패키지를 임포트하면 사용할 수 있다.

JVM은 클래스 안에 들어있는 코드만을 실행할 수 있다. 그럼에도 이게 가능한 이유는 컴파일러가 해당 파일을 컴파일할 때 새로운 클래스를 정의해주기 떄문이다.

### 최상위 프로퍼티

함수와 마찬가지로 프로퍼티도 파일의 최상위 수준에 놓을 수 있다. 어떤 데이터를 클래스 밖에 위치시켜야 하는 경우는 흔하지는 않지만 그래도 유용할 때가 있다.

```kotlin
var opCount = 0 // 최상위 프로퍼티 선언

fun performOperation() {
		opCount++
		// ...
}

fun reportOperationCount() {
	println("Operation performed ${opCount} times") // 최상위 프로퍼티 접근
}
```

이런 프로퍼티의 값은 정적 필드에 저장된다.

기본적으로 최상위 프로퍼티도 다른 모든 프로퍼티처럼 접근자 메소드를 통해 자바코드에 노출된다. 겉으론 상수처럼 보이지만 실제로는 게터를 사용해야 한다면 자연스럽지 못하다. 더 자연스럽게 사용하려면 이 상수를 자바처럼 public static final 필드로 선언해야한다. 이와 동일한 역할을 코틀린에서는 const가 대신한다. const 변경자를 추가하면 프로퍼티를 public static final 필드로 컴파일할 수 있고 상수처럼 사용할 수 있다. 단, 원시 타입과 String 타입에 한정된다.

```kotlin
/* 자바 */
public static final String SOMEPROPERTY = "P";

/* 코틀린 */
const val SOMEPROPERTY = "P"
```

## 메소드를 다른 클래스에 추가 : 확장 함수와 확장 프로퍼티

### 확장함수

개념적으로 확장 함수는 단순하다.

확장 함수는 어떤 클래스의 멤버 메소드인 것처럼 호출할 수 있지만 그 클래스 밖의 다른곳에 선언된 함수이다.

예시로 어떤 문자열의 마지막 문자를 돌려주는 메소드를 만들어보자.

```kotlin
fun String.lastChar() : Char = this.get(this.length - 1)
```

확장 함수를 만들기위해선 추가하려는 함수의 이름 앞에 그 함수가 확장할 클래스의 이름을 덧붙이기만 하면 된다.

클래스 이름을 receiver type 이라 부르며, 확장 함수가 호출되는 대상이 되는 값(객체)을 receiver object라 부른다. 

호출은 다음과 같다. String이 receiver type이고 "Kotlin"이 receiver object이다.

```kotlin
>>> println("Kotlin".lastChar())
n
```

이는 String 클래스에 새로운 메소드를 추가하는 것과 같다. String 클래스가 우리가 직접 작성한 코드가 아니고 String 클래스의 소스 코드를 소유한 것도 아니지만, 우리는 원하는 메소드를 String 클래스에 추가할 수 있다.

하지만 확장 함수라고 하더라도 캡슐화를 깨지는 않는다는 사실을 기억하자.

확장 함수 내부에서 수신 객체의 메소드나 프로퍼티를 바로 사용할 수 있지만 private나 protect 멤버는 사용할 수 없다.

지금까지 내용을 토대로 joinToString() 메소드를 개선하면 다음과 같다.

```kotlin
fun <T> Collection<T>.joinToString( // Collection<T>에 대한 확장함수 선언
    separator: String,
    prefix: String,
    postfix: String
) : String {

    val result = StringBuilder(prefix)
    for((index, element) in this.withIndex()) { // this는 수신 객체로, 여기서는 Collection<T>를 가리킴
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```

### 확장 함수는 오버라이드 불가능하다.

멤버 함수 오버라이드

```kotlin
open class View {
	open fun click() = println("View Clicked")
}

class Button : View() {
	override fun click() = println("Button Clicked")
}

>>> val view: View = Button()
>>> view.click()
Button Clicked
```

Button이 View의 하위 타입이기 때문에 View 타입 변수를 선언해도 Button 타입 변수를 그 변수에 대입할 수 있다. 그리고 Button이 View의 메소드를 오버라이드 했다면 Button의 메소드가 호출된다.

하지만 확장 함수는 위처럼 작동하지 않는다.

```kotlin
fun View.showOff() = println("I am a view!")

fun Button.showOff() = println("I am a button!")

>>> val view: View = Button()
>>> view.showOff()
I am a view!
```

확장 함수는 클래스의 일부가 아니다.

이름과 파라미터가 완전히 같은 확장 함수를 기반 클래스와 하위 클래스에 대해 각각 정의하더라도, 어떤 확장 함수가 호출될 지는 수신 객체로 지정한 변수의 "정적 타입"에 의해 결정되지 그 변수에 저장된 객체의 "동적 타입"에 의해 결정되지 않는다.

- 동적 디스패치(dynamic dispatch) : 실행 시점에 객체 타입에 따라 동적으로 호출될 대상 메소드를 결정하는 방식
- 정적 디스패치(static dispatch) : 컴파일 시점에 알려진 변수 타입에 따라 정해진 메소드를 호출하는 방식

그리고 어떤 클래스를 확장한 함수와 그 클래스의 멤버 함수의 이름과 시그니처가 같다면 확장 함수가 아니라 멤버 함수가 호출된다. 멤버 함수의 우선순위가 더 높기 때문이다.

### 확장 프로퍼티

확장 프로퍼티를 사용하면 기존 클래스 객체에 대한 프로퍼티 형식의 구문으로 사용할 수 있는 API를 추가할 수 있다. 프로퍼티라 불리지만 상태를 저장할 방법은 없다. 하지만 프로퍼티 문법으로 더 짧게 코드를 작성할 수 있어서 편한 경우가 많다.

앞에서 lastChar라는 메소드를 정의했는데 이를 확장 프로퍼티 형식으로 바꿔본다.

```kotlin
val String.lastChar: Char
	get() = get(length - 1)
```

하지만 확장 프로퍼티는 일반 프로퍼티와 달리 backing field가 없어서 기본 게터 구현을 제공할 수 없다. 그래서 최소한 게터는 꼭 정의해야한다. 마찬가지로 초기화 코드에서 계산한 값을 담을 장소가 없으므로 초기화 코드도 쓸 수 없다.

## 컬렉션 처리 : 라이브러리 확장, 가변 길이 인자, 중위 함수

### 자바 컬렉션 API 확장

앞에서 코틀린 컬렉션은 자바와 같은 클래스를 사용하지만 더 확장된 API를 제공한다고 했다.

```kotlin
>>> val strings: List<String> = listOf("first", "second", "fourteenth")
>>> strings.last()
fourteenth

>>> val numbers: Collection<Int> = setOf(1, 14, 2)
>>> numbers.max()
14
```

위 코드에서 strings와 numbers는 모두 자바 라이브러리 클래스의 인스턴스 컬렉션이다. 하지만 자바 라이브러리에서 last()와 max()는 제공하지 않는다. 어떻게 코틀린이 자바 클래스에 새로운 기능을 추가할 수 있었는가?

이제는 답을 말할 수 있다. last()와 max()는 확장 함수이다.

코틀린 표준 라이브러리는 수많은 확장 함수를 포함한다. IDE의 코드 자동 완성 기능을 통해 이를 활용해보자.

### 가변 인자 함수 : 인자의 갯수가 달라질 수 있는 함수 정의

코틀린의 리스트를 생성하는 함수 listOf를 살펴보자.

```kotlin
val list = listOf(2, 3, 5, 7, 9)
```

라이브러리에서 이 함수의 정의를 살펴보면 다음과 같다.

```kotlin
public fun <T> listOf(vararg elements: T): List<T> 
	= if (elements.size > 0) elements.asList() else emptyList()
```

코틀린의 가변 길이 인자는 자바와 비슷하지만 문법이 조금 다르다. 타입 뒤에 ...를 붙이는 자바와 달리 코틀린에서는 파라미터 앞에 vararg 변경자를 붙인다.

### 값의 쌍 다루기  : 중위 호출과 구조 분해 선언

맵을 만들려면 mapOf 함수를 사용한다.

```kotlin
val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
```

여기서 to라는 단어는 코틀린의 키워드가 아니다. 이 코드는 중위 호출(infix call)이라는 특별한 방식으로 to라는 일반 메소드를 호출한 것이다.

중위 호출 시에는 수신 객체와 유일한 메소드 인자 사이에 매소드 이름을 넣는다.(사이에 공백 필수)

다음 두 호출은 동일하다.

```kotlin
1.to("one") //to 메소드의 일반적인 호출

1 to "one"  //to 메소드의 중위방식으로 호출
```

인자가 하나뿐인 일반 메소드나 인자가 하나뿐인 확장 함수에 중위 호출을 사용할 수 있다. 함수 앞에 infix 변경자를 추가하면 된다. to 함수의 정의는 다음과 같다.

```kotlin
infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```

이 to 함수는 Pair 인스턴스를 반환하는데, 이를 이용해 두 변수를 즉시 초기화하는 방법도 있다.

```kotlin
val (number, name) = 1 to "one"
```

이런 기능을 구조 분해 선언(destructing declaration)이라고 부른다.

앞의 joinToString 함수에서도 이를 활용한 코드가 있었다.

```kotlin
for((index, element) in collection.withIndex()) {
	println("${index}: ${element}")
}
```

to는 확장 함수이며 수신 객체가 제네릭하다. 그렇기 때문에 to를 사용하면 타입과 관계없이 임의의 순서쌍을 만들 수 있다.

## 정리

- 코틀린은 자체 컬렉션 클래스는 없지만 자바 컬렉션 클래스를 확장하여 더 풍부한 API를 제공한다.
- 함수 파라미터의 디폴트 값을 정의하면 오버로딩한 함수를 정의할 필요성이 줄어든다. 이름붙인 인자를 사용하면 함수의 인자가 많을 때 함수 호출의 가독성을 향상시킬 수 있다.
- 코틀린 파일에선 클래스 멤버가 아닌 바깥에 최상위 함수와 프로퍼티를 선언할 수 있다. 이를 활용하여 코드 구조를 더 유연하게 만들 수 있다.
- 확장 함수와 프로퍼티를 사용하면 외부 라이브러리에 정의된 클래스를 포함해 모든 클래스의 API를 해당 클래스의 내부 코드를 바꿀 필요 없이 확장할 수 있다.
- 중위 호출을 통해 인자가 하나뿐인 메소드나 확장 함수를 더 깔끔한 구문으로 호출할 수 있다.